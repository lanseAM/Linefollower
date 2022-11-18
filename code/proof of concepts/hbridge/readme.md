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
