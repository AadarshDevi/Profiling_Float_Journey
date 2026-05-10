# Profiling Float Journey

## May 9, 2026

Today I received my LoRa modules (SX1278 433MHz LoRa Module v4.0) from eBay.
I was very excited to start working on my float project.
I previously bought my receiver and adapter a few weeks before.

I connected my 2 LoRa modules to 2 Arduinos. I followed this tutorial: [Interfacing SX1278 (Ra-02) LoRa Module with Arduino](https://circuitdigest.com/microcontroller-projects/arduino-lora-sx1278-interfacing-tutorial)

| LoRa Module Pin | Arduino Uno Pin | Arduino Duemilanove Pin |
|:---------------:|:---------------:|:-----------------------:|
|MISO|Pin 12|Pin 12|
|DIOO|Pin 2|Pin 2|
|SCK|Pin 13|Pin 13|
|MOSI|Pin 11|Pin 11|
|RST|Pin 9|Pin 9|
|NSS|Pin 10|Pin 10|
|GND|GND|GND|
|3V3|3.3V|3.3V|

I opened the Arduino IDE and downloaded the _**LoRa library by Sandeep Mistry**_ and opened the _**LoRaSender**_ and _**LoRaReceiver**_ example sketches. I uploaded both and checked out how they worked.

After fiddling around with the example sketches, I created my own **_LoRa_Sender_v1_** and _**LoRa_Receiver_v1**_ sketches. The data received was gibberish and I used Google Gemini to debug the issue.
