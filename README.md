# TP3(EFC) – Final Project - Accordeur automatique de guitare - (Thomas Chu)

# 1. Le projet

## Présentation 

Le but du projet est de réaliser un accordeur automatique de guitare avec des capteurs et (dans mon cas) un arduino UNO. L'utilisateur peut sélectionner la corde qu'il veut accorder à l'aide du potentiomètre prévu à cet effet et des leds qui jouent le rôle d'indicateurs visuels. Une fois la corde sélectionnée, le stepper motor tournera la clé de la guitare pour accorder la guitare. J'ai moi nême fait une démo du circuit dans la vidéo ci-dessous.

## Difficultés 

Le project n'est pas 100% opérationnel, j'ai rencontré des difficultés pour mesurer la fréquence d'une source sonore (voir le code [ici](https://github.com/cegepmatane/2022-a-projet-final-GitGudShu/blob/main/frequency/frequency.ino)). Pour tout de même rendre quelque chose de présentable, j'ai simulé la clé d'une guitare avec un potentiomètre. C'est cette simulation qui est décrite ci-dessous.

Si j'avais à ma disposition une guitare, l'idée serait donc aussi d'ajouter un "bras" attaché au stepper pour saisir la clé de la guitare.

# 2. Vidéo de présentation

[![Watch the video](https://github.com/cegepmatane/2022-a-projet-final-GitGudShu/blob/main/demo.jpg)](https://www.youtube.com/watch?v=oNwg5O6B6pg&ab_channel=Sh%C5%AB)

# 3. Composants

- Arduino UNO
- 6 LED 
- 6 Résistances 220 ohm
- Fils de liaisons mâles et femelles
- Pile de 5V
- Stepper Motor 28BYJ-R8
- ULN2003 Driver Board
- 2 Potentiomètres

# 4. Code

```cpp

// Include the AccelStepper library:
#include <AccelStepper.h>

// Motor pin definitions:
#define motorPin1  8      // IN1 on the ULN2003 driver
#define motorPin2  9      // IN2 on the ULN2003 driver
#define motorPin3  10     // IN3 on the ULN2003 driver
#define motorPin4  11     // IN4 on the ULN2003 driver

// Define the AccelStepper interface type; 4 wire motor in half step mode:
#define MotorInterfaceType 8

// Initialize with pin sequence IN1-IN3-IN2-IN4 for using the AccelStepper library with 28BYJ-48 stepper motor:
AccelStepper stepper = AccelStepper(MotorInterfaceType, motorPin1, motorPin3, motorPin2, motorPin4);

int motorSpeed = 500;

// led pins
int LED_E2 =  6;
int LED_A2 = 5;
int LED_D2 = 4;
int LED_G3 =  3;
int LED_B3 = 2;
int LED_E3 = 7;

// string frequency
float E2 = 82.407;
float A = 110.000; // can't use A2 because of pin name
float D2 = 146.832;
float G3 = 195.998;
float B3 = 246.942;
float E3 = 329.628;

int threshold = 5; // I will allow 5 Hz off the right frequency

// I use this variable to select the string I want to tune
int activeLight = 0;

int pMin = 14;  //the lowest value that comes out of the potentiometer
int pMax = 948; //the highest value that comes out of the potentiometer.

int x = 0;  //I will use this value to store the readings of the first potentiometer
int y = 0;  //I will use this value to store the readings of the second potentiometer

void setup(){
  pinMode(LED_E2, OUTPUT);
  pinMode(LED_A2, OUTPUT);
  pinMode(LED_D2, OUTPUT);
  pinMode(LED_G3, OUTPUT);
  pinMode(LED_B3, OUTPUT);
  pinMode(LED_E3, OUTPUT);
  stepper.setMaxSpeed(1000);
  Serial.begin(9600);
}

void light(int LED)
{
  digitalWrite(LED_E2, LOW);
  digitalWrite(LED_A2, LOW);
  digitalWrite(LED_D2, LOW);
  digitalWrite(LED_G3, LOW);
  digitalWrite(LED_B3, LOW);
  digitalWrite(LED_E3, LOW);

  digitalWrite(LED, HIGH);
}

void tune(float note){
  if(x <= note - threshold){
    stepper.setSpeed(motorSpeed);
  }else if(x >= note + threshold){
    stepper.setSpeed(-motorSpeed);
  }else{
    stepper.setSpeed(0);
  }
}

void loop()
{
  x = analogRead(A0); //connect the potentiometer to the A0 pin of the Arduino
  x = map(x, pMin, pMax, 0, 350); //take the value of x, compared it to the scale of the potentiometer pMin to pMax, and translate that value to the scale of 0 to 100

  y = analogRead(A1); //connect the potentiometer to the A0 pin of the Arduino
  y = map(y, pMin, pMax, 0, 120); //take the value of x, compared it to the scale of the potentiometer pMin to pMax, and translate that value to the scale of 0 to 100

  // Light the lights (I'm funny)
  if(y < 20){
    light(LED_E2);
    activeLight = 1;
  }else if(y>=20 && y<40){
    light(LED_A2);
    activeLight = 2;
  }else if(y>=40 && y<60){
    light(LED_D2);
    activeLight = 3;
  }else if(y>=60 && y<80){
    light(LED_G3);
    activeLight = 4;
  }else if(y>=80 && y<100){
    light(LED_B3);
    activeLight = 5;
  }else if(y>100){
    light(LED_E3);
    activeLight = 6;
  }

  // turn the step motor
  switch(activeLight){
    case 1:
      tune(E2);
      break;
    case 2:
      tune(A);
      break;
    case 3:
      tune(D2);
      break;
    case 4:
      tune(G3);
      break;
    case 5:
      tune(B3);
      break;
    case 6:
      tune(E3);
      break;
  }

  // Step the motor with constant speed as set by setSpeed():
  stepper.runSpeed();

  // Display frequency value
  Serial.print(x);
  Serial.println(" Hz");
  //Serial.println(y);  //post mapped value, println is a linebreak as well
}
```

# Sources

- [stepper tutoriel](https://www.makerguides.com/28byj-48-stepper-motor-arduino-tutorial/)
- [Fréquence audio tutoriel](https://create.arduino.cc/projecthub/lbf20012001/audio-frequency-detector-617856)
- [Accel stepper documentation](https://www.airspayce.com/mikem/arduino/AccelStepper/classAccelStepper.html)
