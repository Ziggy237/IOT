# Manual for controlling LEDs with a node based on the WorldTimeAPI

## Introduction
In this manual, I will guide you through the process of turning the LEDs on a LED strip on or off based on your own specified time, using the WorldTimeAPI as the reference for the current time.

## Requirements
- NodeMCU ESP 8266
- LEDstrip
- USB/USB-C to Micro USB
- Arduino IDE
- Wi-Fi

## Step 1: Setting up Arduino and installing libraries
The first thing we want to do, is setting up Arduino and downloading all libraries we will use over the course of this manual.
Lets start by opening Arduino IDE, go to **File > Preferences**, and under "Additional Boards Manager URLs," add the following **URL**: 

http://arduino.esp8266.com/stable/package_esp8266com_index.json

![Schermopname (474)](https://github.com/user-attachments/assets/860e5bf8-fb79-4359-9810-800d2cffd72d)


Then go to Tools > Board > Boards Manager, search for ESP8266, and install the latest version.

![Schermopname (475)](https://github.com/user-attachments/assets/f8ee2894-1bb4-4ccb-9568-cc72046df4f4)


Now navigate to the libraries tab on the left hand side of the screen.

![Schermopname (473)](https://github.com/user-attachments/assets/fef90d6b-f2c8-45ae-a6a8-7c69fe80003d)

And install the following libraries:
- Adafruit NeoPixel
- ArduinoJson

## Step 2: Setting up the code






