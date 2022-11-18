# Sensoren proof of concept

minimale hard- en software die aantoont dat minimaal 6 sensoren onafhankelijk van elkaar kunnen uitgelezen worden en dat daarbij een zo groot mogelijk bereik van de AD converter benut wordt (indien van toepassing)


code: 

//Code for the QRE1113 Analog board
//Outputs via the serial terminal - Lower numbers mean more reflected
int QRE1113_Pin = 0; //connected to analog 0

void setup(){
  Serial.begin(9600);
}


void loop(){

  int QRE_Value = analogRead(QRE1113_Pin);
  Serial.println(QRE_Value); 

}
