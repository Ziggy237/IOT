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
<br>
<br>

## Step 2: Setting up the code
Now that everything has been setten up, we can begin with making our code! 

1. First we will open a example file, **go to File > Examples > ESP8266WiFi > WiFiClient**

![Schermopname (476)](https://github.com/user-attachments/assets/c6495184-b992-44b5-afbb-8495ff55d037)

2. Now delete all the code except the void setup (Dont worry we will add more code later).
   
   Your code should look like this now:

```
void setup() {
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif
  pixels.begin(); // INITIALIZE NeoPixel strip object (REQUIRED)

  Serial.begin(115200);

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);  
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}
```
3. We will add back the code above the void setup, but this time it will have all the needed libraries + WiFi settings + Time options.

   Copy the following code and paste it above the void setup:

```
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif
#define PIN        D1
#define NUMPIXELS 12

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

#define DELAYVAL 500

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

#ifndef STASSID
#define STASSID "....." //Your wifi name
#define STAPSK "....." //Your wifi password
#endif

const char* ssid = STASSID;
const char* password = STAPSK;

const char* apiUrl = "http://worldtimeapi.org/api/ip"; // URL for automatic timezone detection

// Define the beginning and ending time (HH:MM format)
const String startTime = "22:00";  // Time when LEDS turn off
const String endTime = "09:59";    // Time LEDS will turn on
```

### Note
Dont forrget to actually fill in your own wifi info & preffered LEDs on/off times

![Schermafbeelding 2024-10-13 223339](https://github.com/user-attachments/assets/0441c6bf-fa7c-4b6f-8e0d-e49bb3c0afdf)
![Schermafbeelding 2024-10-13 225358](https://github.com/user-attachments/assets/d8d843ca-74ee-4889-8a50-59c747c5e1cf)


### Note
Instead of using the already written down URl "http://worldtimeapi.org/api/ip" who will find your timezone based on IP, you can also use a url specific to your prefered timezone. <br/> **Example** "http://worldtimeapi.org/api/timezone/Europe/Amsterdam"

You can see all available timezones checking out this url on internet:
http://worldtimeapi.org/api/timezone

![Schermopname (477)](https://github.com/user-attachments/assets/05cd6d22-477e-4f94-b6bd-c47271dd0010)

4. Now add the following code below the void setup:

```
void loop() {
    if (WiFi.status() == WL_CONNECTED) { // Check if the Wi-Fi is connected
        WiFiClient client; 
        HTTPClient http;   

        http.begin(client, apiUrl); // Connect to the API URL
        int httpResponseCode = http.GET(); // Send a GET request

        // Check if the request was successful
        if (httpResponseCode > 0) {
            String payload = http.getString(); // Receive the data
            Serial.println("Response received from API:");

            // Print the whole payload (response)
            Serial.println(payload);

            // Parse JSON
            DynamicJsonDocument doc(1024); // Create a JSON document
            DeserializationError error = deserializeJson(doc, payload); // Parse the JSON

            if (!error) {
                // Get the datetime from the JSON
                const char* datetime = doc["datetime"];
                const char* timezone = doc["timezone"]; // Get the timezone from the JSON
                Serial.print("The current time in ");
                Serial.print(timezone); // Print the timezone
                Serial.print(" is: ");
                Serial.println(datetime); // Print the time

                // Extract the HH:MM part from the datetime string
                String currentTime = String(datetime).substring(11, 16); 
                
                // Check if the current time is between startTime and endTime
                if (currentTime >= startTime && currentTime <= endTime) {
                    // LEDS turn off between the start and end time
                    Serial.println("LED Off");

                    // Turn off the last red LEDS
                    for (int i = 0; i < NUMPIXELS; i++) {
                        if (i < 8) { // First 8 LEDS stay on
                            if (i < 4) {
                                pixels.setPixelColor(i, pixels.Color(0, 150, 0)); // Green
                            } else {
                                pixels.setPixelColor(i, pixels.Color(255, 165, 0)); // Orange
                            }
                        } else { // Last 4 LEDs Off
                            pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Zet de LED uit (uit)
                        }
                    }
                } else {
                    // LEDS on outside the given time period
                    Serial.println("LED On");

                    // Turn all LEDS on
                    for (int i = 0; i < NUMPIXELS; i++) {
                        if (i < 4) { // First 4 LEDs: Green
                            pixels.setPixelColor(i, pixels.Color(0, 150, 0)); // Green
                        } else if (i < 8) { // Next 4 LEDs: Orange
                            pixels.setPixelColor(i, pixels.Color(255, 165, 0)); // Orange
                        } else { // Last 4 LEDs: Red
                            pixels.setPixelColor(i, pixels.Color(255, 0, 0)); // Red
                        }
                    }
                }
                
                pixels.show(); // Send the updated pixel colors to the hardware.
                delay(DELAYVAL); // Pause before next pass through loop
            } else {
                Serial.print("JSON parsing error: ");
                Serial.println(error.c_str()); // Print the error if the JSON is parsed incorrectly
            }
        } else {
            Serial.print("Error retrieving data: ");
            Serial.println(httpResponseCode); // Print the error code
        }

        http.end(); // Close the HTTP connection
    } else {
        Serial.println("No Wi-Fi connection");
    }

    delay(5000); // Next request time
}
```

### Note
You can play around with the RGB values so you can have your preferred colors. A handy online tool to find your preffered colors in RGB values is
https://www.rapidtables.com/web/color/RGB_Color.html
<br>
<br>
## Step 3: Setting up the hardware
For this part you will be needing your, NodeMCU ESP 8266, LEDstrip and USB/USB-C to Micro USB.

First thing you want to do is plug in your NodeMcu to your laptop/pc.
<br>
<br>
<img src="https://github.com/user-attachments/assets/d0a1f12e-8c23-4dbc-96e0-8046964235dc" width="500"/>


The next step is to connect the correct wires of your LED strip to the matching pins on the NodeMCU.
The combinations are as follows:
| LEDstrip | NodeMCU  |   
|---------|---------|
| Din (Yellow) | D1 pin |   
| +5V (Red) | 3 Volt pin |   
| G (Black) | G pin | 

<img src="https://github.com/user-attachments/assets/44f5dda2-d6c6-4956-90a4-bdbfbf205161" width="350"/>  <img src="https://github.com/user-attachments/assets/3d9bb9c3-de27-4541-8432-0b970aa7bed6" width="350"/>    
### Note
If your D1 pin is not working right you can change it to another "D" pin, just make sure you also specify this in your code

![Schermafbeelding 2024-10-17 210406](https://github.com/user-attachments/assets/79c67ee9-0419-471f-9165-29cb43f38c57)
<br>
<br>
## Step 4: Uploading the code 
Now that we’ve completed the setup, we can start uploading our code! First, make sure you’ve selected the correct port where your NodeMCU is connected.

![Schermafbeelding 2024-10-17 212750](https://github.com/user-attachments/assets/fd19d404-ceaa-4aad-8490-d1e3f4a452f7)


Now To upload your code to the NodeMcu you have to press the big green arrow on the top left of your screen.

![Schermafbeelding 2024-10-11 151022](https://github.com/user-attachments/assets/913d8b14-91bc-4bc9-8301-f09ae172ebf3)

If it is done correctly your output tab will be counting up to 100%
![Schermopname (482)](https://github.com/user-attachments/assets/c3e141a9-d089-4523-8111-c4b72baa03d1)


### Note 
If you see an error like this in the output tab at the bottom of the screen, it means your NodeMCU is still not properly connected to your laptop or PC. I recommend checking if the USB cable is functioning correctly

![Schermafbeelding 2024-10-17 211734](https://github.com/user-attachments/assets/120b2daf-53f0-4451-8317-297a3541a1f7)

After uploading your code, you can open the serial monitor (magnifying glass at the top right of your screen) and make sure yout put the baud rate on 115200
![Schermafbeelding 2024-10-17 214859](https://github.com/user-attachments/assets/9a2f02f9-a7a3-4a67-9a85-bc2467724d28)

<br>
<br>
<br>

 Now that that is done, there are multiple possible outcomes:

### It is working

![Schermopname (466)](https://github.com/user-attachments/assets/4ac50dfd-6953-47a4-9ffd-213f0fcfa92f)
You received a response of the World Time Api with your current time 

<br>
<br>
### Your NodeMCU can't connect to your WiFi

![Schermopname (463)](https://github.com/user-attachments/assets/16ed3233-5f97-47e8-9960-589d1115a0de)
Your NodeMCU is unable to connect to your Wi-Fi. To fix this, ensure that you’ve entered the correct Wi-Fi name (SSID) and password, and check that your Wi-Fi signal is strong enough.

<br>
<br>
### Too many API requests 

![Schermafbeelding 2024-10-17 215551](https://github.com/user-attachments/assets/7136510d-aaab-4661-a100-8890ab97671e)
You have sent too many requests to the API in a short period of time. For now, there’s nothing you can do except wait and try again later. To prevent this issue in the future, you can increase the delay time at the bottom of your code. The longer the delay, the less likely you are to encounter this problem.
















