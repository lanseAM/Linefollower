# start/stop interrupt proof of concept
minimale hard- en software die de correcte werking van een start/stop drukknop aantoont, gebruik makend van een hardware interrupt

### Opstelling
![image](https://github.com/lanseAM/Linefollower/assets/114751410/5105ffdc-08dd-4577-ae42-286d6bb01b9b)


### Arduino code:

#define LED_PIN 9
#define BUTTON_PIN 3

volatile byte ledState = LOW;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT);

  attachInterrupt(digitalPinToInterrupt(BUTTON_PIN), blinkLed, RISING);
}

void loop() {
  // nothing here!
}

void blinkLed() {
  ledState = !ledState;
  digitalWrite(LED_PIN, ledState);
}
