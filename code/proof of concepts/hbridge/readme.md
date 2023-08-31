# H-Bridge proof of concept

minimale hard- & software die aantoont dat 2 motoren onafhankelijk van elkaar kunnen draaien, en (traploos) regelbaar zijn in snelheid en draairichting.

### Opstelling
![image](https://github.com/lanseAM/Linefollower/assets/114751410/ea44f4df-89a3-496a-9ce5-baa266fdbd53)



### Arduino code:
   /*
  DRV8833-Dual-Motor-Driver-Module
  made
  by Lanse Peleman

*/

#define AIN1 8
#define AIN2 9
#define BIN1 10
#define BIN2 11

void setup() {
  Serial.begin(9600);
  
  pinMode(AIN1,OUTPUT);
  pinMode(AIN2,OUTPUT);
  pinMode(BIN1,OUTPUT);
  pinMode(BIN2,OUTPUT);
  
}
 
void loop() {
  
  digitalWrite(AIN1,HIGH); 
  digitalWrite(AIN2,LOW);
  digitalWrite(BIN1,HIGH); 
  digitalWrite(BIN2,LOW);

  delay(1000);
  
  digitalWrite(AIN1,LOW); 
  digitalWrite(AIN2,LOW);
  digitalWrite(BIN1,LOW); 
  digitalWrite(BIN2,LOW);
  
  delay(1000);
  
  digitalWrite(AIN1,LOW); 
  digitalWrite(AIN2,HIGH);
  digitalWrite(BIN1,LOW); 
  digitalWrite(BIN2,HIGH);

  delay(1000);
  
  digitalWrite(AIN1,LOW); 
  digitalWrite(AIN2,LOW);
  digitalWrite(BIN1,LOW); 
  digitalWrite(BIN2,LOW);
  
  delay(1000);
   
}

   
