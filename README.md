# Profiling Float Journey

## May 9, 2026

Today I received my LoRa modules (SX1278 433MHz LoRa Module v4.0) from eBay.
I was very excited to start working on my float project.
I previously bought my receiver and adapter a few weeks before.

Parts:
1. 1x Arduino Uno R3
2. 1x Arduino Duemilanove (Can be another board)
3. 16x Jumper Wires
4. 2x [LoRa Modules](https://www.ebay.com/itm/116899013863)
5. 2x [Antennas](https://www.ebay.com/itm/287003416768?var=589046837530)
6. 2x [U.fl IPX to SMA Remale Adapters](https://www.ebay.com/itm/372806488321?var=641627670983)


LoRa Modules:
1. Count: 2
2. Link: [LoRa Modules](https://www.ebay.com/itm/116899013863)
3. Unit Price: $5
4. Shipping Price: $5.8
5. Total Price: ~$15.8

Antennas:
1. Count: 2
2. Link: [Antennas](https://www.ebay.com/itm/287003416768?var=589046837530)
3. Unit Price: $4.3
4. Total Price: ~$4.3

U.fl IPX to SMA Remale Adapters:
1. Count: 2
2. Link: [U.fl IPX to SMA Remale Adapters](https://www.ebay.com/itm/372806488321?var=641627670983)
3. Unit Price: $5.34
4. Shipping Price: $2.99
5. Total Price: ~$13.67

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

| Arduino Duemilanove  |  Arduino Uno R3 |
|:-:|:-:|
| ![Image of Aruino Duemilanove to LoRa module connection](https://github.com/AadarshDevi/Profiling_Float_Journey/blob/main/may_9_2026/PXL_20260510_050831077.MP.jpg)| ![Image of Aruino Uno R3 to LoRa module connection](https://github.com/AadarshDevi/Profiling_Float_Journey/blob/main/may_9_2026/PXL_20260510_050900585.MP.jpg)|

I opened the Arduino IDE and downloaded the _**LoRa library by Sandeep Mistry**_ and opened the _**LoRaSender**_ and _**LoRaReceiver**_ example sketches. I uploaded both and checked out how they worked.

After fiddling around with the example sketches, I created my own **_LoRa_Sender_v1_** and _**LoRa_Receiver_v1**_ sketches. The data received was gibberish and I used Google Gemini to debug the issue.
After working with Gemini on the code, I was able to send and receive data from the LoRa modules without problems.

Problems when working with the LoRa modules:

When I was sending the data, the endDataTransfer flag was not fully sent. To fix this, I added delays in a few places which has helped to send all the data.

Another issue was that the first data point was not being recorded by the reciever. This was fixed by the delays added to the sender's code.

below are the sender's and receiver's code.

**_LoRa_Sender_v1:_**
```c
#include <SPI.h>
#include <LoRa.h>

const unsigned int baudRate = 9600;
const String startDataFlag = "--start-data-transfer";
const String endDataFlag = "--end-data-transfer";

int dataCounter = 0;
int profile = 0;

const int maxDataCount = 20;
const int maxProfiles = 2;

void setup() {

  Serial.begin(baudRate);
  randomSeed(analogRead(A0));
  
  if (!LoRa.begin(433E6)) {
    while (1);
  }

  LoRa.setSignalBandwidth(62.5E3); // Lower bandwidth is more robust
  LoRa.setSpreadingFactor(9);     // Higher SF handles noise better
  LoRa.setCodingRate4(8);         // Maximum error correction
  LoRa.setSyncWord(0xF3);         // Ensure both use the same "network ID"

  delay(3000);

  sendPacket(startDataFlag);
  
  delay(1000);
}

void loop() {
  
  if (dataCounter == maxDataCount) {
    profile++;
    dataCounter = 0;
  }

  if(profile == maxProfiles) {
    sendPacket(endDataFlag);
    delay(100);
    LoRa.end();
    exit(0);
  }

  float time_in_s = ((float)micros()) / 1000000; // s

  double depth = random(0, 500) / 10;
  double pressure = random(100, 1000) / 10;

  sendPacket("PN12-MiramarWaterJets,pkt-"+String(profile + 1)+","+String(time_in_s)+","+String(depth)+","+String(pressure));
  dataCounter++;

  delay(750);
}

void sendPacket(String data) {
  LoRa.beginPacket();
  LoRa.print(data);
  Serial.println(data);
  LoRa.endPacket();
  delay(100);
}

```

_**LoRa_Receiver_v1**_
```c
#include <SPI.h>
#include <LoRa.h>

const unsigned int baudRate = 9600;
const String startDataFlag = "--start-data-transfer";
const String endDataFlag = "--end-data-transfer";
bool sendData = false;

void setup() {
  Serial.begin(baudRate);
  while (!Serial);

  if (!LoRa.begin(433E6)) {
    Serial.println("Starting LoRa failed!");
    while (1);
  }
  
  LoRa.setSignalBandwidth(62.5E3); // Lower bandwidth is more robust
  LoRa.setSpreadingFactor(9);     // Higher SF handles noise better
  LoRa.setCodingRate4(8);         // Maximum error correction
  LoRa.setSyncWord(0xF3);         // Ensure both use the same "network ID"
}

void loop() {
  // try to parse packet
  int packetSize = LoRa.parsePacket();
  
  if (packetSize) {

      String data = "";
      while (LoRa.available()) {
        data += (char)LoRa.read();
      }

      data.trim();

      if (data == startDataFlag) {
        sendData = true;
      } else if (data == endDataFlag) {
        sendData = false;
        Serial.println(endDataFlag);
      }

      if (sendData) {
        Serial.println(data);
      }
      // delay(500);
  }
}

```
