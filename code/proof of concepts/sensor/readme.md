# Sensoren proof of concept

minimale hard- en software die aantoont dat minimaal 6 sensoren onafhankelijk van elkaar kunnen uitgelezen worden en dat daarbij een zo groot mogelijk bereik van de AD converter benut wordt (indien van toepassing)

### Opstelling 
![image](https://github.com/lanseAM/Linefollower/assets/114751410/2c8191bf-f1e6-4c46-91ac-64f4e4cd9ab0)



### Arduino code
Verbindt de arduino Mega met je pc en upload deze code: 

#include <QTRSensors.h>

// Define the number of sensors and their pins
#define NUM_SENSORS 6
#define SENSOR_PINS {A6, A5, A4, A3, A2, A1}

// Create an instance of the QTRSensors class
QTRSensors qtr;

// Array to store sensor readings
unsigned int sensorValues[NUM_SENSORS];

void setup() {
  // Initialize serial communication
  Serial.begin(9600);

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

  delay(2000);  // Allow time to observe calibration values
}

void loop() {
  // Read sensor values
  qtr.read(sensorValues);

  // Print sensor values
  for (int i = 0; i < NUM_SENSORS; i++) {
    Serial.print(sensorValues[i]);
    Serial.print('\t');
  }
  Serial.println();

  delay(100);  // Adjust delay as needed
}

3) In de seriele monitor krijg je nu de values van de sensoren te zien.
   
