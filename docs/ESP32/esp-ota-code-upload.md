---
id: 208
title: ESP OTA code upload
type: post
date: 2020-05-17 19:11:42
lastmod: 2023-04-01 23:59:29
categories:
- Arduino/ESP
---


When ESP is installed on a project, the USB port is not always accessible. Sometimes, even though the USB is accessible, the device is installed on site and bringing along your laptop to reprogram it is troublesome. It is more convinient to just reprogram it over the air (OTA).







However, in the case of over the air updates, there cannot be any delays over 5s (5000ms). So as long as we check for updates every 1s or so, it should be ok.




Do note that the device can no longer communicate over serial monitor (however it can still be reprogrammed over serial).




Here is a compiled header file that can be inlcuded in the codes to make it simple to reprogram the ESP with OTA support.





```c++
#include <WiFi.h>
#include <ESPmDNS.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
#include <TelnetStream.h>
#include <credentials.h>

void setupOTA() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.waitForConnectResult() != WL_CONNECTED) {
    Serial.println("Connection Failed! Rebooting...");
    delay(5000);
    ESP.restart();
  }

  // Port defaults to 3232
  // ArduinoOTA.setPort(3232);

  // No authentication by default
  // ArduinoOTA.setPassword("admin");

  // Password can be set with it's md5 value as well
  // MD5(admin) = 21232f297a57a5a743894a0e4a801fc3
  // ArduinoOTA.setPasswordHash("21232f297a57a5a743894a0e4a801fc3");

  ArduinoOTA
  .onStart([]() {
    String type;
    if (ArduinoOTA.getCommand() == U_FLASH)
      type = "sketch";
    else // U_SPIFFS
      type = "filesystem";

    // NOTE: if updating SPIFFS this would be the place to unmount SPIFFS using SPIFFS.end()
    Serial.println("Start updating " + type);
  })
  .onEnd([]() {
    Serial.println("\nEnd");
  })
  .onProgress([](unsigned int progress, unsigned int total) {
    Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
  })
  .onError([](ota_error_t error) {
    Serial.printf("Error[%u]: ", error);
    if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
    else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
    else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
    else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
    else if (error == OTA_END_ERROR) Serial.println("End Failed");
  });

  ArduinoOTA.begin();
  TelnetStream.begin();

  Serial.println("OTA Initialized");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}
```



Once the header file is placed in library folder, add this to the template (example) for your IDE.





```c++
#include "OTA.h"

const char* ssid = mySSID;
const char* password = myPASSWORD;

unsigned long entry;

void setup() {
  Serial.begin(115200);
  Serial.println("Booting");
  ArduinoOTA.setHostname("TemplateSketch");
  setupOTA();
}

void loop() {
  entry=micros();
  ArduinoOTA.handle();
  TelnetStream.println(micros()-entry);
  TelnetStream.println("Loop");
  delay(1000);
}
```



After, use a telnet program to monitor the serial and reprogram the ESP over the air by simply choosing it from "Ports".






