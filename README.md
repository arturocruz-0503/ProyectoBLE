Link del repositorio de GitHub: https://github.com/arturocruz-0503/ProyectoBLE/edit/main/README.md
Componentes utilizados.
•	Placa ESP32 con conexión Bluetooth.
•	Celular Android.
•	3 LEDs (verde, amarillo y azul)
•	Protoboard
•	Jumpers
Objetivo del proyecto.
Diseñar e implementar un sistema de control inalámbrico de LEDs haciendo uso de la comunicación BLE entre un ESP32 y una aplicación móvil, a través de dicha aplicación móvil se controlarán LEDs de manera inalámbrica que estarán conectados a la placa ESP32 
Descripción del proyecto.
El proyecto consiste en desarrollar un sistema de comunicación inalámbrica utilizando BLE entre una placa de desarrollo ESP32 (o alguna parecida) y una aplicación móvil creada en MIT App Inventor.
Mediante la aplicación creada con App Inventor se podrá realizar lo siguiente:
•	Buscar dispositivos Bluetooth cercanos.
•	Mostar una lista de los dispositivos detectados.
•	Conectar la placa ESP32 a un dispositivo móvil usando la aplicación móvil creada en App Inventor.
•	Enviar comandos (cadenas de texto) del dispositivo móvil al ESP32 con los cuales encenderá LEDs según el botón apretado.

Conexiones
LED	GPIO ESP32	Descripción
Verde	16	Listo para conectar / Control LED 1
Azul	17	Bluetooth conectado / Control LED 2
Amarillo	18	Bluetooth desconectado / Control LED 3

Comandos
Comando	Acción	GPIO
A	Enciende LED Verde	16
a	Apaga LED Verde	16
B	Enciende LED Azul	17
b	Apaga LED Azul	17
C	Enciende LED Rojo	18

Código
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>

// ID para identificar el dispsitivo y mandar las acciones desde mit app inventor //
#define SERVICE_UUID        "12345678-1234-1234-1234-1234567890ab"
#define CHARACTERISTIC_UUID "abcd1234-5678-1234-5678-abcdef123456"

// numero de pines de los leds //
#define LED1_PIN 16
#define LED2_PIN 17
#define LED3_PIN 18

bool dispositivoConectado = false;

void setAll(bool on) {
  digitalWrite(LED1_PIN, on ? HIGH : LOW);
  digitalWrite(LED2_PIN, on ? HIGH : LOW);
  digitalWrite(LED3_PIN, on ? HIGH : LOW);
}

class MyServerCallbacks : public BLEServerCallbacks {
  void onConnect(BLEServer* pServer) override {
    dispositivoConectado = true;
    Serial.println("Conectado");
  }
  void onDisconnect(BLEServer* pServer) override {
    dispositivoConectado = false;
    Serial.println("Desconectado");
    pServer->getAdvertising()->start(); // desconcetarse de el esp32
  }
};

class MyCallbacks : public BLECharacteristicCallbacks {
  
void onWrite(BLECharacteristic* pCharacteristic) override {
  String rx = pCharacteristic->getValue();   // ahora lo recibe como texto para que cuando en la aplicacion se ejecute un boton, el esp32 lo tome y pueda hacer la accion segun correponda con el boton que se presiono
  if (rx.length() == 0) return;

  char c = rx.charAt(0);                     // tomamos el primer caracter o letra que da la aplicacion o el boton presionado
  Serial.print("Recibido: ");
  Serial.println(c);

  switch (c) {
    case 'A': digitalWrite(LED1_PIN, HIGH); break;
    case 'a': digitalWrite(LED1_PIN, LOW);  break;
    case 'B': digitalWrite(LED2_PIN, HIGH); break;
    case 'b': digitalWrite(LED2_PIN, LOW);  break;
    case 'C': digitalWrite(LED3_PIN, HIGH); break;
    case 'c': digitalWrite(LED3_PIN, LOW);  break;
    default:
      Serial.println("Comando no reconocido");
      break;
  }
}

};

void setup() {
  Serial.begin(115200);

  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(LED3_PIN, OUTPUT);
  setAll(false);

  BLEDevice::init("ESP32 sin seven"); 
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  BLEService *pService = pServer->createService(SERVICE_UUID);

  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
    CHARACTERISTIC_UUID,
    BLECharacteristic::PROPERTY_WRITE     |
    BLECharacteristic::PROPERTY_WRITE_NR  | // por si la app escribe sin respuesta
    BLECharacteristic::PROPERTY_READ
  );

  pCharacteristic->setCallbacks(new MyCallbacks());

  pService->start();
  pServer->getAdvertising()->start();

  Serial.println("ESP32 listo (BLE)");
}

void loop() {
  // no es necesario hacer nada aqui ya que el funcionamiento se hace afuera del loop porque desde ahi se puede hacer

Lógica del programa
1. Inicialización

Se configura BLE con:
Service UUID: 12345678-1234-1234-1234-1234567890ab
Characteristic UUID: abcd1234-5678-1234-5678-abcdef123456

2. Conexión desde App

La app escanea y muestra el dispositivo ESP32_EquipoSicsSeven.
Al conectarse:
Serial: “Conectado”
Se habilitan botones de control en la app

3. Recepción de comandos
La app envía un carácter a la característica BLE:
Comando Acción que le envia la app al esp32.
‘A’Enciende LED Verde
‘a’Apaga LED Verde
‘B’Enciende LED Azul
‘b’Apaga LED Azul
‘C’Enciende LED Rojo
‘c’Apaga LED Rojo

4. Desconexión
Serial: “Desconectado” y vuelve a buscar dispositivos nuevos
el esp32 se desconecta de la aplicación o del teléfono para que otro teléfono con la aplicación pueda conectarse

Cómo usar la App en MIT App Inventor
Instalar extensión BluetoothLE.

Agregar componentes:
BluetoothLE1
ListView para dispositivos
Botones de control (A/a, B/b, C/c, X)


Escanear luego seleccionar ESP32 y conectarlo conectar
La interfaz cambia a botones de control
Cada botón envía un carácter a la ESP32 para que se pueda prender o apagar un led


Configuración BLE para el Funcionamiento de la practica:
Callbacks para conexión/desconexión
Interpretación de comandos BLE
Control de 3 LEDs físicos
Mensajes por monitor serial


Cómo ejecutar
Conectar ESP32 a la PC
Cargar el programa proporcionado
Abrir monitor serial (115200) para ver si manda los mensajes de conectado, desconectado y el botón que se presiono en la aplicación 

En MIT App Inventor:
Abrir App, luego Escanear y Conectamos el esp32
Usar los botones para encender/apagar LEDs en tiempo real


Características especiales que se usaron:
Comunicación BLE compatible con Android y iOS
App cambia automáticamente de interfaz al conectarse haciendo que ahora pase a conectado
Comandos simples (1 carácter), baja latencia
Estados visuales con LEDs físicos
