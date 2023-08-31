# basis programma

volgende zaken werken samen in dit template project  

* instelbare parameters mbv. SerialCommands 
  * cyclus tijd
  * debug on/off
  * start/stop
* parameters worden opgeslagen in eeprom geheugen
* start/stop kan ook geactiveerd worden mbv. een externe interrupt
* draadloze communicatie

Met behulp van de cyclus kan je bvb. een knipperende led programmeren.

## arduino code

#include "SerialCommand.h"
#include "EEPROMAnything.h"
#define MotorLeftForward 9
#define MotorLeftBackward 8
#define MotorRightForward 11
#define MotorRightBackward 10


#define Baudrate 115200
SerialCommand sCmd(SerialPort);


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
  SerialPort.begin(Baudrate);
  sCmd.addCommand("calibrate", onCalibrate);
  sCmd.addCommand("set", onSet);
  sCmd.addCommand("debug", onDebug);
  sCmd.setDefaultHandler(onUnknownCommand);
  sCmd.addCommand("start", onStart);
  sCmd.addCommand("stop", onStop);
  EEPROM_readAnything(0, params);
  SerialPort.println("ready");
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
    normalised[6];
    for (int i = 0; i < 6; i++)
    {
      long value = analogRead(sensor[i]);
      normalised[i] = map(value, params.black[i], params.white[i], 0, 1000);
    }
    float position = 0;
    int index = 0;
    for (int i = 1; i < 6; i++) if (normalised[i] < normalised[index]) index = i;

    if (index == 0) position = -30;
    else if (index == 5) position = 30;
    else{
    long sZero = normalised[index];
    long sMinusOne = normalised[index-1];
    long sPlusOne = normalised[index+1];

    float b = ((float) (sPlusOne - sMinusOne)) / 2;
    float a = sPlusOne - b - sZero;

    position = -b / (2 * a);  
    position += index - 2.5;   
    position *= 15;  //sensor distance in mm


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
  SerialPort.print("unknown command: \"");
  SerialPort.print(command);
  SerialPort.println("\"");
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
  SerialPort.print("cycle time: ");
  SerialPort.println(params.cycleTime);
  /* parameters weergeven met behulp van het debug commando doe je hier ... */
  SerialPort.print("black: ");
  for (int i = 0; i<6; i++)
  {
  SerialPort.print(params.black[i]);
  SerialPort.print(" ");
  }
  SerialPort.println(" ");
  
  SerialPort.print("white: ");
  for (int i = 0; i<6; i++)
  {
  SerialPort.print(params.white[i]);
  SerialPort.print(" ");
  }
  SerialPort.println(" ");
  
  SerialPort.print("normalised: ");
  for (int i = 0; i < 6; i++)
  {
    SerialPort.print(normalised[i]);
    SerialPort.print(" ");
  }
  SerialPort.println(" ");
  
  SerialPort.print("calculation time: ");
  SerialPort.println(calculationTime);
  calculationTime = 0;
  
  SerialPort.print("power: ");
  SerialPort.println(params.power);
  
  SerialPort.print("kp: ");
  SerialPort.println(params.kp);

  SerialPort.print("ki: ");
  SerialPort.println(params.ki);

  SerialPort.print("kd: ");
  SerialPort.println(params.kd);
    
  SerialPort.print("Positie: ");
  SerialPort.println(debugPosition);

  SerialPort.print("diff: ");
  SerialPort.println(params.diff);  
}

void onCalibrate()
{
   char* param = sCmd.next();

  if (strcmp(param, "black") == 0)
  {
    SerialPort.print("start calibrating black... ");
    for (int i = 0; i < 6; i++) params.black[i]=analogRead(sensor[i]);
    SerialPort.println("done");
  }
 else if (strcmp(param, "white") == 0)
  {
    SerialPort.print("start calibrating white... ");    
    for (int i = 0; i < 6; i++) params.white[i]=analogRead(sensor[i]);  
    SerialPort.println("done");      
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
  
