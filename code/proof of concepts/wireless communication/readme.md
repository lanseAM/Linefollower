# draadloze communicatie proof of concept
minimale hard- en software waarmee aangetoond wordt dat duplex kan gecommuniceerd worden tussen de microcontroller en een [smartphone], gebruik makend van [MIT AI2 Companion] (in te vullen)
<br />

### configuratie
Voor de communicatie plan B werd een HC-05 (HC-06 plan A)gebruikt met de app serial bluetooth monitor van de appstore
![image](https://github.com/lanseAM/Linefollower/assets/114751410/c8709df8-33d1-4279-b0cb-9af0c225a805)


### opmerkingen
elektronisch schema
![image](https://github.com/lanseAM/Linefollower/assets/114751410/c951b366-90cf-4dca-bfc6-40fd646eb019)


### Arduino code:
-----------
###HC-05
#include <SoftwareSerial.h>
SoftwareSerial EEBlue(18, 19); // RX | TX
 
void setup()
{
  Serial.begin(9600);
  EEBlue.begin(38400);  //Baud Rate for command Mode. 
  Serial.println("Enter AT commands!");
}
 
void loop()
{
 
  // Feed any data from bluetooth to Terminal.
  if (EEBlue.available())
    Serial.write(EEBlue.read());
 
  // Feed all data from termial to bluetooth
  if (Serial.available())
    EEBlue.write(Serial.read());
}


-----------
###HC-06
#include <SoftwareSerial.h>
SoftwareSerial hc06(2,3);
String cmd="";
float sensor_val=0;
void setup(){
  //Initialize Serial Monitor
  Serial.begin(9600);
  //Initialize Bluetooth Serial Port
  hc06.begin(9600);
}
void loop(){
  //Read data from HC06
  while(hc06.available()>0){
    cmd+=(char)hc06.read();
  }
  //Select function with cmd
  if(cmd!=""){
    Serial.print("Command recieved : ");
    Serial.println(cmd);
    // We expect ON or OFF from bluetooth
    if(cmd=="ON"){
        Serial.println("Function is on");
    }else if(cmd=="OFF"){
        Serial.println("Function is off");
    }else{
        Serial.println("Function is off by default");
    }
    cmd=""; //reset cmd
  }
  // Simulate sensor measurement
  sensor_val=(float)random(256); // random number between 0 and 255
  //Write sensor data to HC06
  hc06.print(sensor_val);
  delay(100);
}




