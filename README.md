# Arduino_proyect
https://www.instructables.com/Arduino-Radar-1/ (Punto de partida)


imagen del poroyecto montado en tinkercad
<img width="1920" height="856" alt="Amazing Kieran-Fulffy" src="https://github.com/user-attachments/assets/cd40dc8e-7850-4443-918f-87407224ee93" />


codiogo practicamente acabado

#include <Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// --- CONFIGURACIÓN PANTALLA OLED ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1 
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// --- PINOUT ---
const int pinTrig = 10;
const int pinEcho = 9;
const int pinServo = 11;

// Pines del RGB (Si usas leds separados, ajusta aquí)
const int pinRGB_Rojo = 4;
const int pinRGB_Verde = 5;
const int pinLedFlash = 6; 

Servo miServo;

// --- VARIABLES ---
float distancia1, distancia2;
float velocidad; 
unsigned long tiempoEspera = 100; // 0.1 segundos
float limiteVelocidad = 40.0; 

void setup() {
  Serial.begin(9600);
  
  // Iniciar Pantalla
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // La dirección suele ser 0x3C
    Serial.println(F("Error: No encuentro la pantalla OLED"));
    for(;;); // Se queda pillado aquí si falla
  }
  display.clearDisplay();
  display.setTextColor(WHITE);
  
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  
  pinMode(pinRGB_Rojo, OUTPUT);
  pinMode(pinRGB_Verde, OUTPUT);
  pinMode(pinLedFlash, OUTPUT);
  
  miServo.attach(pinServo);
  miServo.write(0); 
  
  setColor(0, 1); // Verde inicial
  
  // Mensaje de bienvenida en pantalla
  display.setTextSize(1);
  display.setCursor(0, 10);
  display.println("RADAR ACTIVO");
  display.display();
  delay(2000);
}

void loop() {
  distancia1 = medir();
  delay(tiempoEspera); 
  distancia2 = medir();
  
  velocidad = (distancia1 - distancia2) / (tiempoEspera / 1000.0);

  // --- ACTUALIZAR PANTALLA ---
  display.clearDisplay();
  
  // Dibujar caja alrededor
  display.drawRect(0, 0, 128, 64, WHITE);
  
  // Título
  display.setTextSize(1);
  display.setCursor(10, 5);
  display.print("VELOCIDAD ACTUAL:");
  
  // Número GRANDE de la velocidad
  display.setTextSize(3);
  display.setCursor(20, 25);
  
  if (velocidad > 2) {
    display.print((int)velocidad); // Mostramos sin decimales pa que se vea grande
  } else {
    display.print("0");
  }
  
  display.setTextSize(1);
  display.print(" cm/s");
  
  // --- LÓGICA DE ALERTAS ---
  if (velocidad > 2) { 
    // Servo
    int angulo = map(velocidad, 0, 100, 0, 180);
    miServo.write(constrain(angulo, 0, 180));

    if (velocidad > limiteVelocidad) {
      // --- MODO MULTA ---
      display.setCursor(10, 55);
      display.print("! EXCESO !"); // Aviso en pantalla
      display.display(); // Actualizamos pantalla antes del flash
      
      setColor(1, 0); // Rojo
      
      // Flash
      for(int i=0; i<3; i++){
        digitalWrite(pinLedFlash, HIGH);
        delay(40);
        digitalWrite(pinLedFlash, LOW);
        delay(40);
      }
      delay(1000);
      
    } else {
      // --- MODO SEGURO ---
      display.setCursor(10, 55);
      display.print("SEGURO");
      display.display();
      setColor(0, 1); // Verde
    }
  } else {
    // Reposo
    display.setCursor(10, 55);
    display.print("ESPERANDO...");
    display.display();
    
    miServo.write(0);
    setColor(0, 1); 
  }
}

void setColor(int rojo, int verde) {
  digitalWrite(pinRGB_Rojo, rojo);
  digitalWrite(pinRGB_Verde, verde);
}

float medir() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);
  long duracion = pulseIn(pinEcho, HIGH);
  float d = duracion / 58.2;
  if (d > 300 || d == 0) return 0;
  return d;
}

hemos tenido la idea de que en vez de un sonar el sensor sirviese como radar para los vehiculos, este seria un radar movil que esta en un costado de la carretera girando en 180 grados y captando las velocidades de los coches. esta informacion se pasa al semaforo, que son 3 leds que s exceden la velocidad limite de la zona el semaforo se pondra en rojo automaticamente obligando al coche a detenerse y reducir su velocidad
