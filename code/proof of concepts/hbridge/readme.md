# H-Bridge proof of concept

minimale hard- & software die aantoont dat 2 motoren onafhankelijk van elkaar kunnen draaien, en (traploos) regelbaar zijn in snelheid en draairichting.

nu werkt met bibliotheek in arduino stepper.H maar kan ook gwn met pwm zonder bibliotheek wat minder problemen kan geven

code: 

#include <Stepper.h>
// change this to the number of steps on your motor
#define STEPS 5
// create an instance of the stepper class, specifying
// the number of steps of the motor and the pins it's
// attached to
Stepper stepper(STEPS, 3, 5, 9, 10);
void setup()
{
 Serial.begin(9600);
 Serial.println("Stepper test!");
 // set the speed of the motor to 30 RPMs
 stepper.setSpeed(5);
}
void loop()
{
 Serial.println("Forward");
 stepper.step(STEPS);
 Serial.println("Backward");
 stepper.step(-STEPS);
}


![image](https://user-images.githubusercontent.com/114751410/202721173-654bdcd4-0b6d-456a-b4e6-82be4f5a7fc4.png)

met pwm
code:
#include "SerialCommand.h"

#include <EEPROM.h>
#include "EEPROMAnything.h"

#define SerialPort Serial

#define Baudrate 9600

SerialCommand sCmd(SerialPort);
bool debug;
unsigned long previous, calculationTime;
struct param_t
{
  bool start;
  unsigned long cycleTime;
  int power;
  bool links;
  bool reverse;
  bool rechts;
  
} params;

void setup()
{
  SerialPort.begin(Baudrate);

  pinMode(3, OUTPUT);  //A1
  pinMode(5, OUTPUT);  //A2
  pinMode(9, OUTPUT);  //B2
  pinMode(10, OUTPUT); //B1

  sCmd.addCommand("set", onSet);
  sCmd.addCommand("start", onStart);
  sCmd.addCommand("stop", onStop);
  sCmd.addCommand("links", onLinks);
  sCmd.addCommand("rechts", onRechts);
  sCmd.addCommand("reverse", onReverse);
  sCmd.addCommand("debug", onDebug);
  sCmd.setDefaultHandler(onUnknownCommand);

  EEPROM_readAnything(0, params);
  params.start = false;
  SerialPort.println("ready");
}

void loop()
{
  sCmd.readSerial();
 
  unsigned long current = micros();
  if (current - previous >= params.cycleTime)
  {
    previous = current;
  }
 int snelheid = params.power;
    if(params.start){
      if(params.links){
          analogWrite(3, abs(snelheid));
          digitalWrite(5, LOW);
          digitalWrite(10, LOW);
          digitalWrite(9, LOW);
      }
      else if(params.rechts){
          digitalWrite(3, LOW);
          digitalWrite(5, LOW);
          analogWrite(9, abs(snelheid));
          digitalWrite(10, LOW);
      }
      else if(params.reverse){
          analogWrite(5, abs(snelheid));
          digitalWrite(3, LOW);
          analogWrite(9, abs(snelheid));
          digitalWrite(10, LOW);
      }else{
          analogWrite(3, abs(snelheid));
          digitalWrite(5, LOW);
          analogWrite(10, abs(snelheid));
          digitalWrite(9, LOW);
      }
    }else{
      digitalWrite(3, LOW);
      digitalWrite(5, LOW);
      digitalWrite(9, LOW);
      digitalWrite(10, LOW);
    }
    
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
  char* param = sCmd.next();
  char* value = sCmd.next();  
  
  if (strcmp(param, "cycle") == 0) params.cycleTime = atol(value);
  if (strcmp(param, "power") == 0) params.power = atoi(value);

  EEPROM_writeAnything(0, params);
}

void onDebug()
{
  SerialPort.print("cycle time: ");
  SerialPort.println(params.cycleTime);
  SerialPort.print("Aan : ");
  SerialPort.println(params.start);
  SerialPort.print("Power : ");
  SerialPort.println(params.power); 
  SerialPort.print("calculation time: ");
  SerialPort.print(calculationTime);
  calculationTime = 0;
}

void onStart()

{
    params.start = true;
    
}

void onStop()

{
    params.start = false;
    params.links = false;
    params.rechts = false;
    params.reverse =false;
}
void onLinks()
{
  params.links = true;
  params.reverse = false;
  params.rechts = false;
  
}
void onRechts()
{
  params.rechts = true;
  params.links = false;
  params.reverse = false;
}
void onReverse()
{
  params.reverse = true;
  params.links = false;
  params.rechts = false;
}


   
