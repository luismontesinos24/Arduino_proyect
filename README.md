# Arduino_proyect
https://www.instructables.com/Arduino-Radar-1/ (Punto de partida)


imagen del poroyecto montado en tinkercad
<img width="1920" height="856" alt="Amazing Kieran-Fulffy" src="https://github.com/user-attachments/assets/cd40dc8e-7850-4443-918f-87407224ee93" />


codiogo de el radar el cual ha tocado modificar brevemente 

#includeconst int TriggerPin = 8;

#include <Servo.h>

 int TriggerPin = 8;

const int EchoPin = 9;

const int motorSignalPin = 10;
const int startingAngle = 90;

const int minimumAngle = 6;

const int maximumAngle = 175;

const int rotationSpeed = 1;

Servo motor;

void setup(void)

{ pinMode(TriggerPin, OUTPUT);

pinMode(EchoPin, INPUT);

motor.attach(motorSignalPin);

Serial.begin(9600);

}

void loop(void)

{ static int motorAngle = startingAngle;

static int motorRotateAmount = rotationSpeed;

motor.write(motorAngle);

delay(10);

SerialOutput(motorAngle, CalculateDistance());

motorAngle += motorRotateAmount;

if(motorAngle <= minimumAngle || motorAngle >= maximumAngle) { motorRotateAmount = -motorRotateAmount;

}}

int CalculateDistance(void)


{ digitalWrite(TriggerPin, HIGH);

delayMicroseconds(10);

digitalWrite(TriggerPin, LOW);

long duration = pulseIn(EchoPin, HIGH);

float distance = duration * 0.017F;

return int(distance);

}

void SerialOutput(const int angle, const int distance)

{

String angleString = String(angle);

String distanceString = String(distance);

Serial.println(angleString + "," + distanceString);

}

