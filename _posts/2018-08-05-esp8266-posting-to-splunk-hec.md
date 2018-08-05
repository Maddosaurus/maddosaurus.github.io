---
layout: post
title:  "ESP8266 POSTing sensor data to Splunk HEC"
date:   2018-08-05 13:37
tags: splunk
---
One Day Builds: Use an ESP8266 to collect sensor data and transmit these to Splunk!  
I recently acquired a DHT22 temperature and humidity sensor and though to myself "gee, it would be awfully nice if I could collect time series data of this sensor".  

<!--more-->

## Final Product
A bit of a tradition right from the start: A glance at the finished PoC.  
There are two gauges that display the current temperature and humidity, as well as a time series display for historical data that can be scaled through the attached timepicker.  
The whole dashboard didn't took me longer than 30 minutes to put together - Splunk really makes this easy.  
![Splunk dashboard](https://i.imgur.com/LcbQapY.png){:class="img-responsive"}

## Logical Layout
So basically, a DHT22 is attached to an ESP8266, feeding of its power line and providing its sensor data to the ESP if needed.  
The ESP in turn is connected to my WiFi at home and sends the current values for temperature, humidity and heat index to my Splunk instance once a minute.  
This is done through the magic of the [Splunk HTTP Event Collector](http://docs.splunk.com/Documentation/Splunk/latest/Data/UsetheHTTPEventCollector), which I have set up and running using HTTPS only.

<div class="mermaid">
graph LR
    A["fa:fa-thermometer-half DHT22"] --- B["fa:fa-microchip ESP8266"]
    B -. WiFi/Internet .- C["fa:fa-cloud Splunk"]
</div>

## Hardware
As ESP I'm using an [*ESP-12E* development board](https://aliexpress.com/item/Free-shipping-NodeMCU-development-board-for-ESP-12E-from-ESP8266-esp-12E-Lua-IoT-programable-wifi/32357301146.html), which is arguably one of the better ESP8266 versions.  
The temperature sensor is an [*AM2302 DHT22* sensor package](https://eu.banggood.com/Wholesale-Warehouse-AM2302-DHT22-Temperature-And-Humidity-Sensor-Module-For-Arduino-SCM-wp-Eu-937403.html) that is already seated on a PCB.  
The sensor can be operated at either 3V or 5V, so I connected it directly to the 3v3 rail on my ESP.  
The data line has been hooked up to ~D4 (silkscreened), which corresponds to the digital pin 2 on my board.  
Basically, the wiring layout is inspired by [this post](https://www.losant.com/blog/getting-started-with-the-esp8266-and-dht22-sensor).
![wiring layout](https://i.imgur.com/dLHy5zl.jpg){:class="img-responsive"}

## Software
I'm using the [Arduino IDE](https://www.arduino.cc/en/Main/Software) for programming - while it doesn't provide nice features like code completion or a language server, it makes managing board and functionality libraries somewhat manageable.  

### Patching the board lib
First things first: Using HTTPS with the ESP8266HTTPClient is, at the time of writing, a hot mess. Unfortunately, one has to provide certificate fingerprints with every call to verify the host - and even if provided, these don't always work. The troubleshooting kept me occupied for some hours until I found [this nice post](https://www.esp8266.com/viewtopic.php?f=29&t=16473) which linked to an [open pull request](https://github.com/esp8266/Arduino/pull/2821) where someone has changed this behaviour.  
As of today, this PR still isn't merged, so let's get started.  
I provide instructions for Arch Linux - please double check your paths!  

1. Have a look at the [official ESP8266 install instructions](https://github.com/esp8266/Arduino#using-git-version) and follow along.  
    1.1: The arduino application directory for me is `/usr/share/arduino/hardware`
2. Enther the cloned directory and pull in the PR: `git pull origin pull/2821/head`
3. Open the Arudino IDE again. You should be able to choose an ESP8266 flavour in the board selection!

As this is done, you are now able to switch off the HTTPS certificate verify by calling `http.setIgnoreTLSVerifyFailure(true);` in your code.

### Source Code
The source that is currently running on the ESP is a bit wonky but does what its supposed to do.  
It needs some infos from you before it can run:
- WiFi SSID and password to connect to
- Splunk server address and token

Especially the Splunk info needs to adhere to a special format.  
The code expects the collector URL to contain the full path to your HEC instance (most likely `https://splunk.local:8088/services/collector`).  
The token can be generated via *Splunk Web -> Settings -> Data Inputs -> HTTP Event Collector -> New Token* and should be provided as "Splunk $token".  
**Note the Splunk with the whitespace**. This is important as Splunk expects the auth header to be formed like this.  

The code should be somewhat readable, but let me walk you through the main bits:  
**setup()**
- Connects to the WiFi and flashes internal LED while doing this. If connected, the flashing stops

**loop()**
- Only executes anything if WiFi is connected!
- read in temperature and humidity and do some error checking
- calculate the heat index
- initialize and fill the JSON 
- use HTTP POST to send data over to Splunk

Splunk expects the JSON in the specific format of `{"event": $some_stuff}`.  
This is the JSON I am sending with this code:
```JSON
{
    "event":
    {
        "sensorLocation": "location_in_room",
        "temperature": 29.1,
        "humidity": 41.8,
        "heatindex": 28.8784
    }
}
```

There is some debug output available through the `/dev/tty` serial console.  
Just set it to baud 9600 and have a look at whats happening.  
Without further ado, here's the source (also available as a Gist on Github)

```cpp
// AS LONG AS PULL REQUEST 2821 IS NOT MERGED, YOU NEED TO PATCH THE ESP8266 BOARD LIB!
// have a look here: https://www.esp8266.com/viewtopic.php?f=29&t=16473
// and here: https://github.com/esp8266/Arduino/pull/2821

#include "DHT.h"
#include <ESP8266HTTPClient.h>
#include <ESP8266WiFi.h>
#include <ArduinoJson.h>

#define DHTPIN 2     // This is silkscreened as ~D4 on my board
#define DHTTYPE DHT22   // Sensor type

DHT dht(DHTPIN, DHTTYPE);

// WiFi to connect to
const char* ssid = "xxxxxx";
const char* password = "xxxxxxx";

// Splunk server to send data to
// Get this: openssl s_client -connect host.local:port | openssl x509 -fingerprint -noout
const char* splunk_collector_cert = "AA:BB:CC:DD:EE"; // This is the SHA-1 hash of the cert
const char* splunk_collector_url = "https://your.splunk.target:8088/services/collector";
const char* splunk_HEC_Token = "Splunk abcde-splunk-hec-token-asdg"; // IMPORTANT: Leave the "Spunk " there!

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600);
  Serial.println("Starting ESP temp serve");
  Serial.printf("Connecting to %s \n", ssid);
  digitalWrite(LED_BUILTIN, LOW);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  {
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
  }
  Serial.println("connected!");
  digitalWrite(LED_BUILTIN, HIGH);

  dht.begin();

  Serial.printf("Local IP: %s\n", WiFi.localIP().toString().c_str());
}

void loop() {
  if(WiFi.status()== WL_CONNECTED){       // Only send on running WiFi

    // get all dem sensor readouts
    float temp = dht.readTemperature();
    float humid = dht.readHumidity();
    
    if(isnan(temp)) { temp = -1.0; }
    if(isnan(humid)) { humid = -1.0; }
    
    float hi = dht.computeHeatIndex(temp, humid, false);

    
    // prepare the JSON data structure
    StaticJsonBuffer<300> JSONbuffer;   //Declaring static JSON buffer
    JsonObject& JSONencoder = JSONbuffer.createObject(); 
    
    JsonObject& eventJSON = JSONencoder.createNestedObject("event");
    
    eventJSON["sensorLocation"] = "Sensor Location in the Room";
    eventJSON["temperature"] = temp;
    eventJSON["humidity"] = humid;
    eventJSON["heatindex"] = hi;

    char JSONmessageBuffer[300];
    JSONencoder.printTo(JSONmessageBuffer);
    Serial.println(JSONmessageBuffer);


    // and now build and send the HTTP POST
    HTTPClient http; 
    http.setIgnoreTLSVerifyFailure(true); // this ONLY works with the patched version of PR 2821!
    
    http.begin(splunk_collector_url, splunk_collector_cert);
    http.addHeader("Content-Type", "application/json");
    http.addHeader("Authorization", splunk_HEC_Token);  

    int httpStatus = http.POST(JSONmessageBuffer);
    String payload = http.getString();

    Serial.println(httpStatus);   //Print HTTP return code
    Serial.println(payload);    //Print request response payload

    http.end();
    
    delay(60000); // wait for 60 seconds
  }
}
```