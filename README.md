# Project Monitoring & Controlling Greenhouse
Project Monitoring & Controlling Greenhouse merupakan pengembangan yang bertujuan memajukan sistem pertanian, terutama dalam penggunaan Rumah Kaca (Greenhouse). Sistem ini dapat membantu petani dalam memonitoring dan mengontrol greenhouse dengan mudah melalui aplikasi pada smartphone (android).
***By Universitas Gunadarma & Balai Besar Pengujian Standar Instrumen Mekanisasi Pertanian (BBPSI Mektan).***

## Komponen Yang Digunakan
**Mikrokontroler :**
* ESP32

**Sensor :**
* DHT22
* LDR
* Soil Moisture

**Output :**
* Relay 5V
* Lampu Pijar
* Pompa Air
* LCD Nextion

## Skematik Rangkaian Komponen
![Logo](https://github.com/raffx29/Project-Smart-Greenhouse/blob/b63ba998d72723995becf825e132150ddcd37892/Lib/10.jpg)

Pin yang digunakan komponen sensor dan output pada ESP32 yaitu :
| Komponen | Pin     |
| :-------- | :------- |
| `DHT22` | `32` |
| `LDR` | `34` |
| `Soil Moisture` | `35` |
| `Relay 1` | `5` |
| `Relay 2` | `17` |

## Program ESP32
```c++
#include <FirebaseESP32.h>
#include <WiFi.h>
#include "DHT.h"
#include "EasyNextionLibrary.h"

#define FIREBASE_HOST "https://greenhouseapp-e59ae-default-rtdb.firebaseio.com/"
#define WIFI_SSID "BBP MEKTAN"
#define WIFI_PASSWORD "1ndoJarwo"
#define FIREBASE_Authorization_key "HSCT6Im72mX2O0IyLdTaCM8FdMOfVRmheRQVQ0oO"

#define LDRPIN 34
#define SOILPIN 35
#define DHTPIN 32

#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

// Relay
#define Relay1 5   // WATER PUMP
#define Relay2 17  // LAMPU

float humUdara, tempUdara, humTanah;
int ldr;

EasyNex myNex(Serial);

FirebaseData firebaseData;
FirebaseData fbdo;

void setup() {
  Serial.begin(9600);
  myNex.begin(9600);
  dht.begin();
  pinMode(Relay1, OUTPUT);
  pinMode(Relay2, OUTPUT);
  digitalWrite(Relay1, 1);
  digitalWrite(Relay2, 1);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting...");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  Firebase.begin(FIREBASE_HOST, FIREBASE_Authorization_key);
}

void loop() {
  readSensor();
  button();

  Serial.print("Temperature: " + (String)tempUdara + " °C");
  Serial.print("Humidity: " + (String)humUdara + " %");
  Serial.print("Intensitas Cahaya : " + (String)ldr + " Cd");
  Serial.print("Kelembaban Tanah : " + (String)humTanah + " %");
  Serial.println();

  Firebase.setFloat(firebaseData, "/ESP32_APP/TEMPUDARA", tempUdara);
  Firebase.setFloat(firebaseData, "/ESP32_APP/HUMUDARA", humUdara);
  Firebase.setFloat(firebaseData, "/ESP32_APP/INTENSITY", ldr);
  Firebase.setFloat(firebaseData, "/ESP32_APP/HUMTANAH", humTanah);
  delay(200);

  myNex.NextionListen();
  nextSensor();
}

void button() {
  if (Firebase.getString(firebaseData, "/Relay1")) {
    if (firebaseData.dataType() == "string") {
      String FBStatus = firebaseData.stringData();
      if (FBStatus == "ON") {
        Serial.println("Relay ON");
        digitalWrite(Relay1, HIGH);
      } else if (FBStatus == "OFF") {
        Serial.println("Relay OFF");
        digitalWrite(Relay1, LOW);
      } else {
        Serial.println("Salah kode! isi dengan data ON/OFF");
      }
    }

    void readSensor() {
      humUdara = dht.readHumidity();
      tempUdara = dht.readTemperature();
      humTanah = analogRead(SOILPIN);
      humTanah = map(humTanah, 0, 4095, 100, 0);
      ldr = analogRead(LDRPIN);
    }

    void nextSensor() {
      String tempUdaraString = String(tempUdara, 1);
      myNex.writeStr("t1.txt", tempUdaraString);
      int tempUdaraInt = tempUdara * 10;
      myNex.writeNum("x1.val", tempUdaraInt);

      String humUdaraString = String(humUdara, 1);
      myNex.writeStr("t2.txt", humUdaraString);
      int humUdaraInt = humUdara * 10;
      myNex.writeNum("x2.val", humUdaraInt);

      String ldrString = String(ldr, 1);
      myNex.writeStr("t0.txt", ldrString);
      int ldrInt = ldr * 10;
      myNex.writeNum("x0.val", ldrInt);

      String humTanahString = String(humTanah, 1);
      myNex.writeStr("t3.txt", humTanahString);
      int humTanahInt = humTanah * 10;
      myNex.writeNum("x3.val", humTanahInt);
    }
```
Tipe data sensor yang digunakan yaitu :
| Komponen | Tipe     |
| :-------- | :------- |
| `DHT22` | `float` |
| `LDR` | `int` |
| `Soil Moisture` | `float` |

## Interface Aplikasi
![Logo](https://github.com/raffx29/Project-Smart-Greenhouse/blob/7023ba70bec9769d1c439761c3b0e9c368770138/Lib/9.jpg)

## Program Aplikasi
![Logo](https://github.com/raffx29/Project-Smart-Greenhouse/blob/8dc61d599ebccf31e32524cc46457a2cb7a10615/Lib/6.jpg)
![Logo](https://github.com/raffx29/Project-Smart-Greenhouse/blob/8dc61d599ebccf31e32524cc46457a2cb7a10615/Lib/7.jpg)

## Database Google Firebase
![Logo](https://github.com/raffx29/Project-Smart-Greenhouse/blob/b63ba998d72723995becf825e132150ddcd37892/Lib/8.jpg)
