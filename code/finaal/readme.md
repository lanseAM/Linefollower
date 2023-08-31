# arduino code

#include "SerialCommand.h"
#include "EEPROMAnything.h"
#include <QTRSensors.h>
#define MotorLeftForward 9
#define MotorLeftBackward 8
#define MotorRightForward 11
#define MotorRightBackward 10
#define NUM_SENSORS 6
#define SENSOR_PINS {A6, A5, A4, A3, A2, A1}

#define Baudrate 9600
SerialCommand sCmd(Serial1);

QTRSensors qtr;
// Array to store sensor readings
unsigned int sensorValues[NUM_SENSORS];

bool debug;
unsigned long previous, calculationTime;
const int sensor[] = {A6, A5, A4, A3, A2, A1};
long normalised[6];
volatile bool state = LOW;
float debugPosition;
float iTerm;
float lastErr;
struct param_t
{
  unsigned long cycleTime;
  int black[6];
  int white[6];
  float kp;
  float ki;
  float kd;
  int power;
  bool StartStop;
  float diff;
  
  /* andere parameters die in het eeprom geheugen moeten opgeslagen worden voeg je hier toe ... */
} params;

void setup()
{
  Serial1.begin(Baudrate);

 // Configure sensor pins
  unsigned char sensorPins[NUM_SENSORS] = SENSOR_PINS;

  // Configure sensor settings (optional)
  qtr.setTypeRC();
  qtr.setSensorPins(sensorPins, NUM_SENSORS);

  // Calibrate the sensors
  qtr.calibrate();

  // Display calibration minimum and maximum values
  for (int i = 0; i < NUM_SENSORS; i++) {
    Serial.print("Sensor ");
    Serial.print(i);
    Serial.print(": ");
  }

  //commands
  sCmd.addCommand("calibrate", onCalibrate);
  sCmd.addCommand("set", onSet);
  sCmd.addCommand("debug", onDebug);
  sCmd.setDefaultHandler(onUnknownCommand);
  sCmd.addCommand("start", onStart);
  sCmd.addCommand("stop", onStop);
  sCmd.addCommand("waarde", onWaarde);
  
  EEPROM_readAnything(0, params);

   qtr.calibrate();
   qtr.setTypeRC();
  
  Serial1.println("ready");
  pinMode(MotorLeftForward, OUTPUT);
  pinMode(MotorLeftBackward, OUTPUT);
  pinMode(MotorRightForward, OUTPUT);
  pinMode(MotorRightBackward, OUTPUT);
  pinMode(47,OUTPUT);
}

void loop()
{
  sCmd.readSerial();
 
  unsigned long current = micros();
  if (current - previous >= params.cycleTime)
  {
    previous = current;

    /* code die cyclisch moet uitgevoerd worden programmeer je hier ... */
 
  
    /* normaliseren en interpoleren sensor */
    int normalised[6];
    
    for (int i = 0; i < 6; i++)
    {
      
      
      long value = analogRead(sensor[i]);
      normalised[i] = map(analogRead(sensor[i]), params.black[i], params.white[i], 0, 1000);
    }
    float position = 0;
    int index = 0;
    for (int i = 1; i < 6; i++) if (normalised[i] < normalised[index]) index = i;

    if (index == 0) position = -30; //kruispunten, uitschieten
    else if (index == 5) position = 30;
    else{
    long sZero = normalised[index];
    long sMinusOne = normalised[index-1];
    long sPlusOne = normalised[index+1];

    float b = ((float) (sPlusOne - sMinusOne)) / 2;
    float a = sPlusOne - b - sZero;

    position = -b / (2 * a);  
    position += index - 2.5;   // sensordistance tov elkaar
    position *= 11;  //sensor distance in mm


    }
    debugPosition = position;

 
    float error = -position;
    float output = error * params.kp;

    output += params.kd * (error - lastErr);
    lastErr = error;
    
    output = constrain(output, -510, 510);
    int powerLeft = 0;
    int powerRight = 0;

    if (state == 1) if (output >= 0)
    {
      powerLeft = constrain(params.power + params.diff * output, -255, 255);
      powerRight = constrain(powerLeft - output, -255, 255);
      powerLeft = powerRight + output;
    }
    else 
    {
      powerRight = constrain(params.power - params.diff * output, -255, 255);
      powerLeft = constrain(powerRight + output, -255, 255);
      powerRight = powerLeft - output; 
    } 
    
    analogWrite(MotorLeftForward, powerLeft > 0 ? powerLeft : 0);
    analogWrite(MotorLeftBackward, powerLeft < 0 ? -powerLeft : 0);
    analogWrite(MotorRightForward, powerRight > 0 ? powerRight : 0);
    analogWrite(MotorRightBackward, powerRight < 0 ? - powerRight : 0);

    
}
    /* aansturen motoren */


    /* pid regeling */

    
  unsigned long difference = micros() - current;
  if (difference > calculationTime) calculationTime = difference;
}


void onUnknownCommand(char *command)
{
  Serial1.print("unknown command: \"");
  Serial1.print(command);
  Serial1.println("\"");
}

void onSet()
{
  char* parameter = sCmd.next();
  char* value = sCmd.next();
  
  if (strcmp(parameter, "power") == 0) params.power = atol(value);
  else if (strcmp(parameter, "kp") == 0) params.kp = atof(value);
  else if (strcmp(parameter, "ki") == 0) params.ki = atof(value);
  else if (strcmp(parameter, "kd") == 0) params.kd = atof(value);
  else if (strcmp(parameter, "cycle") == 0) params.cycleTime = atof(value);
  else if (strcmp(parameter, "diff") == 0) params.diff = atof(value);
  
  EEPROM_writeAnything(0, params);
}

void onDebug()
{
  Serial1.print("cycle time: ");
  Serial1.println(params.cycleTime);
  /* parameters weergeven met behulp van het debug commando doe je hier ... */



// Adjust delay as needed

  
Serial1.print("black: ");
  
  Serial1.print("black: ");
  for (int i = 0; i<6; i++)
  {
  Serial1.print(params.black[i]);
  Serial1.print(" ");
  }
  Serial1.println(" ");
  
  Serial1.print("white: ");
  for (int i = 0; i<6; i++)
  {
  Serial1.print(params.white[i]);
  Serial1.print(" ");
  }
  Serial1.println(" ");
  
  Serial1.print("normalised: ");
  for (int i = 0; i < 6; i++)
  {
    Serial1.print(normalised[i]);
    Serial1.print(" ");
  }
  Serial1.println(" ");
  
  Serial1.print("calculation time: ");
  Serial1.println(calculationTime);
  calculationTime = 0;
  
  Serial1.print("power: ");
  Serial1.println(params.power);
  
  Serial1.print("kp: ");
  Serial1.println(params.kp);

  Serial1.print("ki: ");
  Serial1.println(params.ki);

  Serial1.print("kd: ");
  Serial1.println(params.kd);
    
  Serial1.print("Positie: ");
  Serial1.println(debugPosition);

  Serial1.print("diff: ");
  Serial1.println(params.diff);  
}

void onCalibrate()
{
   char* param = sCmd.next();

  if (strcmp(param, "black") == 0)
  {
    Serial1.print("start calibrating black... ");
    for (int i = 0; i < 6; i++) params.black[i]=analogRead(sensor[i]);
    Serial1.println("done");
  }
 else if (strcmp(param, "white") == 0)
  {
    Serial1.print("start calibrating white... ");    
    for (int i = 0; i < 6; i++) params.white[i]=analogRead(sensor[i]);  
    Serial1.println("done");      
  }

  EEPROM_writeAnything(0, params);
  
}

void onStart(){
  state = HIGH;
  digitalWrite(47,HIGH);
}

void onStop(){
  state = LOW;
  digitalWrite(47,LOW);
  digitalWrite(MotorRightForward, LOW);
  digitalWrite(MotorRightBackward, LOW);
  digitalWrite(MotorLeftForward, LOW);
  digitalWrite(MotorLeftBackward, LOW);
}

void onWaarde(){
  // Read sensor values
  qtr.read(sensorValues);

  // Print sensor values
  for (int i = 0; i < NUM_SENSORS; i++) {
    Serial.print(sensorValues[i]);
    Serial.print('\t');
  }
  Serial.println();
}


 
  


  
