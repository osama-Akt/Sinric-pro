# Sinric-pro
Working exemple of Garage Door + Switches

/*
 * 
 * If you encounter any issues:
 * - check the readme.md at https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md
 * - ensure all dependent libraries are installed
 *   - see https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md#arduinoide
 *   - see https://github.com/sinricpro/esp8266-esp32-sdk/blob/master/README.md#dependencies
 * - open serial monitor and check whats happening
 * - check full user documentation at https://sinricpro.github.io/esp8266-esp32-sdk
 * - visit https://github.com/sinricpro/esp8266-esp32-sdk/issues and check for existing issues or open a new one
 */

// Uncomment the following line to enable serial debug output
//#define ENABLE_DEBUG

#ifdef ENABLE_DEBUG
       #define DEBUG_ESP_PORT Serial
       #define NODEBUG_WEBSOCKETS
       #define NDEBUG
#endif 

#include <Arduino.h>
#ifdef ESP8266 
       //#include <ESP8266WiFi.h>
#endif 
//#ifdef ESP32   
       //#include <WiFi.h>
//#endif

#include "SinricPro.h"
#include "SinricProSwitch.h"
#include "SinricProGarageDoor.h"

#define WIFI_SSID         ""    
#define WIFI_PASS         ""
#define APP_KEY           ""      // Should look like "de0bxxxx-1x3x-4x3x-ax2x-5dabxxxxxxxx"
#define APP_SECRET        ""   // Should look like "5f36xxxx-x3x7-4x3x-xexe-e86724a9xxxx-4c4axxxx-3x3x-x5xe-x9x3-333d65xxxxxx"
#define GARAGEDOOR_ID     ""    // Should look like "5dc1564130xxxxxxxxxxxxxx"
#define BAUD_RATE         9600                     // Change baudrate to your need

#define SWITCH_ID_1       ""    // Should look like "5dc1564130xxxxxxxxxxxxxx"
#define SWITCH_ID_2       ""    // Should look like "5dc1564130xxxxxxxxxxxxxx"
#define SWITCH_ID_3       ""    // Should look like "5dc1564130xxxxxxxxxxxxxx"

const int SWITCH1 = D1;  //GPIO5
const int SWITCH2 = D2;  //GPIO4
const int SWITCH3 = D3;  //GPIO0

const int Up = D6;       //GPIO12
const int Down= D5;      //GPIO14

bool onDoorState(const String& deviceId, bool &doorState) {
  Serial.printf("Garagedoor is %s now.\r\n", doorState?"closed":"open");
  switch(doorState) {
    case true: digitalWrite(Up, HIGH);digitalWrite(Down, LOW);
               delay(19000);
               digitalWrite(Down, HIGH);
             
               break;
    case false: digitalWrite(Up, LOW);digitalWrite(Down, HIGH);
                delay(19000);
                digitalWrite(Up, HIGH);
               break;     
  }
  return true;
}

bool onPowerState1(const String &deviceId, bool &state) {
  Serial.printf("Device 1 turned %s\r\n", state?"on":"off");
  switch(state) {
    case true: digitalWrite(SWITCH1, LOW);
                break;
    case false: digitalWrite(SWITCH1, HIGH);
                break;     
  }
  return true; // request handled properly
}

bool onPowerState2(const String &deviceId, bool &state) {
  Serial.printf("Device 2 turned %s\r\n", state?"on":"off");
  switch(state) {
    case true: digitalWrite(SWITCH2, LOW);
                break;
    case false: digitalWrite(SWITCH2, HIGH);
                break;     
  }
  return true; // request handled properly
}

bool onPowerState3(const String &deviceId, bool &state) {
  Serial.printf("Device 3 turned %s\r\n", state?"on":"off");
  switch(state) {
    case true: digitalWrite(SWITCH3, LOW);
                break;
    case false: digitalWrite(SWITCH3, HIGH);
                break;     
  }
  return true; // request handled properly
}



void setupWiFi() {
  Serial.printf("\r\n[Wifi]: Connecting");
  WiFi.begin(WIFI_SSID, WIFI_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    Serial.printf(".");
    delay(250);
    
    // check conection state
   if (WiFi.status() == WL_CONNECTED) {
    digitalWrite(LED_BUILTIN, LOW); //led is ON
  } else {
    // turn LED on and off:
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    
  }

 
  }
  
  IPAddress localIP = WiFi.localIP();
  Serial.printf("connected!\r\n[WiFi]: IP-Address is %d.%d.%d.%d\r\n", localIP[0], localIP[1], localIP[2], localIP[3]);

 
}

void setupSinricPro() {
  SinricProGarageDoor &myGarageDoor = SinricPro[GARAGEDOOR_ID];
  myGarageDoor.onDoorState(onDoorState);

    // add devices and callbacks to SinricPro
  SinricProSwitch& mySwitch1 = SinricPro[SWITCH_ID_1];
  mySwitch1.onPowerState(onPowerState1);

  SinricProSwitch& mySwitch2 = SinricPro[SWITCH_ID_2];
  mySwitch2.onPowerState(onPowerState2);

  SinricProSwitch& mySwitch3 = SinricPro[SWITCH_ID_3];
  mySwitch3.onPowerState(onPowerState3);

    // setup SinricPro
  SinricPro.onConnected([](){ Serial.printf("Connected to SinricPro\r\n"); }); 
  SinricPro.onDisconnected([](){ Serial.printf("Disconnected from SinricPro\r\n"); });
  SinricPro.begin(APP_KEY, APP_SECRET);
}

void setup() {
  pinMode(SWITCH1, OUTPUT );digitalWrite(SWITCH1, HIGH);
  pinMode(SWITCH2, OUTPUT );digitalWrite(SWITCH2, HIGH);
  pinMode(SWITCH3, OUTPUT );digitalWrite(SWITCH3, HIGH);
  
  pinMode(Up, OUTPUT ); digitalWrite(Up, HIGH);
  pinMode(Down, OUTPUT ); digitalWrite(Down, HIGH);
  
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(BAUD_RATE); Serial.printf("\r\n\r\n");
 
  setupWiFi();
  setupSinricPro();
}

void loop() {
  SinricPro.handle();
}
