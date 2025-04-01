# leviathan-soccer-bot
An ESP32 based soccer bot

## Components
1. ESP32 dev kit
2. BTS7960 motor drivers (x2)
3. 37GB555 motors (x4)
4. LiPo battery (2200 mAh, 11.1 V, 3S) + XT-60 pin

## Pin Connections
|Component|ESP32 Pin|BTS7960 Pin|Purpose|
|---|---|---|---|
|Left Motor RPWM|GPIO 25|RPWM-L|Control Speed and direction|
|Left Motor LPWM|GPIO 26|LPWM-L|Control Speed and direction|
|Right Motor RPWM|GPIO 27|RPWM-R|Control Speed and direction|
|Right Motor LPWM|GPIO 14|LPWM-R|Control Speed and direction|
|Left Motor Enable|GPIO 12,13|REN-L, LEN-L|Enable motor driver|
|Right Motor Enable|GPIO 33,32|REN-R, LEN-R|Enable motor driver|
|VCC|5V|VCC|Power supply for logic|

## Code for controls

```c
#include <BluetoothSerial.h>

BluetoothSerial SerialBT;

// Motor driver pins
const int RPWM_L = 25; // Left motor RPWM white
const int LPWM_L = 26; // Left motor LPWM purple
const int REN_L = 12;  // Left motor enable red
const int LEN_L = 13;  // Left motor enable orange

const int RPWM_R = 27; // Right motor RPWM orange
const int LPWM_R = 14; // Right motor LPWM yellow
const int REN_R = 33;  // Right motor enable red
const int LEN_R = 32;  // Right motor enable brown

int speedLevel = 0; // Speed (0-255)
int speedLevel_mod; // for compound movement
int num = 2, denom = 5;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("Leviathan"); // Bluetooth device name
  pinMode(RPWM_L, OUTPUT);
  pinMode(LPWM_L, OUTPUT);
  pinMode(RPWM_R, OUTPUT);
  pinMode(LPWM_R, OUTPUT);
  pinMode(REN_L, OUTPUT);
  pinMode(LEN_L, OUTPUT);
  pinMode(REN_R, OUTPUT);
  pinMode(LEN_R, OUTPUT);

  digitalWrite(REN_L, HIGH);
  digitalWrite(LEN_L, HIGH);
  digitalWrite(REN_R, HIGH);
  digitalWrite(LEN_R, HIGH);
}


void loop() {
  if (SerialBT.available()) {
    char command = SerialBT.read();

    switch (command) {
      case 'F': controlMotors(speedLevel, 0, 0, speedLevel);       // forward
        Serial.println("Forward"); break;
      case 'B': controlMotors(0, speedLevel, speedLevel, 0);       // backward
        Serial.println("Backward"); break;
      case 'L': controlMotors(0, speedLevel, 0, speedLevel);       // left
        Serial.println("Left"); break;
      case 'R': controlMotors(speedLevel, 0, speedLevel, 0);       // right
        Serial.println("Right"); break;
      case 'G': controlMotors(speedLevel_mod, 0, 0, speedLevel);   // forward left
        Serial.println("Forward Left"); break;
      case 'I': controlMotors(speedLevel, 0, 0, speedLevel_mod);   // forward right
        Serial.println("Forward Right"); break;
      case 'H': controlMotors(0, speedLevel_mod, speedLevel, 0);   // backward left
        Serial.println("Backward Left"); break;
      case 'J': controlMotors(0, speedLevel, speedLevel_mod, 0);   // backward right
        Serial.println("Backward Right"); break;
      case '0' ... '9': speedLevel = map(command - '0', 0, 9, 0, 255), 
                        speedLevel_mod = (speedLevel * num)/denom; break; // Speed control
      case 'S': stopMotors(); break; // Stop
    }
  }
}


// Common function for motor control
void controlMotors(int leftFwd, int leftBwd, int rightFwd, int rightBwd) {
  analogWrite(RPWM_L, leftFwd);
  analogWrite(LPWM_L, leftBwd);
  analogWrite(RPWM_R, rightFwd);
  analogWrite(LPWM_R, rightBwd);
}

// Stop all motors
void stopMotors() {
  controlMotors(0, 0, 0, 0);
}
```
## Code for OTA(WiFi)

> Make sure the ESP32 and the device uploading the code is connected to the same WiFi

```c
#include <WiFi.h>
#include <ArduinoOTA.h>

const char* ssid = "WiFi_SSID";
const char* password = "WiFi_Password";

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    switch(WiFi.status()) {
      case 1: Serial.println("network unavailable"); break;
      case 3: Serial.println("connected"); break;
      case 4: Serial.println("wrong password"); break;
      case 6: Serial.println("not connected (check SSID)"); break;
    }

    // for HTTP OTA : visit https://xx.xx.xx.xx/
    // Serial.println(WiFi.localIP());

    // OTA Setup
    ArduinoOTA.begin();
}

void loop() {
    ArduinoOTA.handle();  // Keep OTA running
}
```