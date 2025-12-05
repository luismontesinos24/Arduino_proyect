# Arduino_proyect
https://www.instructables.com/Arduino-Radar-1/ (Punto de partida)


imagen del poroyecto montado en tinkercad
<img width="1920" height="856" alt="Amazing Kieran-Fulffy" src="https://github.com/user-attachments/assets/cd40dc8e-7850-4443-918f-87407224ee93" />

Debido a no contar con un led blanco hemos tenido que usar 2 rgb.
codiogo  acabado

#include <Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// --- PANTALLA OLED ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// --- PINOUT ---
const int pinTrig = 10;
const int pinEcho = 9;
const int pinServo = 11;

// RGB 1: Semáforo
const int pinSem_Rojo = 4;
const int pinSem_Verde = 5;

// RGB 2: Flash (Blanco)
const int pinFlash_R = 6;
const int pinFlash_G = 7;
const int pinFlash_B = 8;

Servo miServo;

// --- CONFIGURACIÓN VELOCIDAD ---
float limiteVelocidad = 50.0; // Velocidad para multa

// --- VARIABLES DEL SERVO (TEMPORIZADOR) ---
unsigned long ultimoCambioServo = 0;
// 5 Minutos en milisegundos = 5 * 60 * 1000 = 300000
const unsigned long intervaloServo = 300000; 
bool servoEnPosicionInicial = true; // Para saber dónde está

// --- VARIABLES DEL RADAR ---
float distancia1, distancia2;
float velocidad; 
unsigned long tiempoEsperaRadar = 100; // Medir cada 0.1s

void setup() {
  Serial.begin(9600);
  
  // Pantalla
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 
    for(;;);
  }
  display.clearDisplay();
  display.setTextColor(WHITE);
  
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  
  pinMode(pinSem_Rojo, OUTPUT);
  pinMode(pinSem_Verde, OUTPUT);
  pinMode(pinFlash_R, OUTPUT);
  pinMode(pinFlash_G, OUTPUT);
  pinMode(pinFlash_B, OUTPUT);
  
  miServo.attach(pinServo);
  miServo.write(0); // Posición inicial
  
  // Mensaje inicio
  display.setTextSize(1);
  display.setCursor(0, 10);
  display.println("SISTEMA AUTO");
  display.display();
  delay(1000);
}

void loop() {
  // Obtenemos la hora actual del Arduino
  unsigned long tiempoActual = millis();

  // ---------------------------------------------
  // TAREA 1: CONTROL DEL SERVO (CADA 5 MINUTOS)
  // ---------------------------------------------
  if (tiempoActual - ultimoCambioServo >= intervaloServo) {
    // Han pasado 5 minutos, toca cambiar
    ultimoCambioServo = tiempoActual; // Reseteamos cronómetro
    
    if (servoEnPosicionInicial) {
      // Si estaba en 0, va a 180
      miServo.write(180);
      servoEnPosicionInicial = false;
    } else {
      // Si estaba en 180, vuelve a 0
      miServo.write(0);
      servoEnPosicionInicial = true;
    }
  }

  // ---------------------------------------------
  // TAREA 2: RADAR DE VELOCIDAD (SIEMPRE ACTIVO)
  // ---------------------------------------------
  
  // Medir distancia 1
  distancia1 = medir();
  delay(tiempoEsperaRadar); // Pequeña pausa para el cálculo
  // Medir distancia 2
  distancia2 = medir();
  
  // Calcular
  velocidad = (distancia1 - distancia2) / (tiempoEsperaRadar / 1000.0);
  
  // Actualizar Pantalla y Luces
  actualizarPantallaYLuces(velocidad);
}

void actualizarPantallaYLuces(float vel) {
  display.clearDisplay();
  display.drawRect(0, 0, 128, 64, WHITE);
  
  // Mostrar estado del servo (informativo)
  display.setTextSize(1);
  display.setCursor(5, 5);
  if(servoEnPosicionInicial) {
    display.print("MODO: A (0 deg)");
  } else {
    display.print("MODO: B (180 deg)");
  }

  // Mostrar Velocidad
  display.setTextSize(3);
  display.setCursor(20, 25);
  if (vel > 2) display.print((int)vel);
  else display.print("0");
  display.setTextSize(1);
  display.print(" cm/s");

  // Lógica de Multa
  if (vel > limiteVelocidad) {
    // ! MULTA !
    display.setCursor(10, 55);
    display.print("! EXCESO !");
    
    controlarSemaforo(1, 0); // Rojo
    dispararFlash();         // Flashazo
    
  } else {
    // CIRCULACIÓN NORMAL
    display.setCursor(10, 55);
    display.print("VIGILANDO...");
    
    controlarSemaforo(0, 1); // Verde
    apagarFlash();
  }
  
  display.display();
}

// --- FUNCIONES EXTRA ---
void controlarSemaforo(int rojo, int verde) {
  digitalWrite(pinSem_Rojo, rojo);
  digitalWrite(pinSem_Verde, verde);
}

void dispararFlash() {
  // Flash rápido (sin bloquear mucho tiempo)
  for(int i=0; i<3; i++){
    digitalWrite(pinFlash_R, HIGH);
    digitalWrite(pinFlash_G, HIGH);
    digitalWrite(pinFlash_B, HIGH);
    delay(30);
    digitalWrite(pinFlash_R, LOW);
    digitalWrite(pinFlash_G, LOW);
    digitalWrite(pinFlash_B, LOW);
    delay(30);
  }
}

void apagarFlash() {
  digitalWrite(pinFlash_R, LOW);
  digitalWrite(pinFlash_G, LOW);
  digitalWrite(pinFlash_B, LOW);
}

float medir() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);
  long duracion = pulseIn(pinEcho, HIGH, 25000); 
  if (duracion == 0) return 0;
  return duracion / 58.2;
}

