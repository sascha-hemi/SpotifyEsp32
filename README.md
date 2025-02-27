# Spotify Library for ESP32 
This library is a wrapper for the [Spotify Web API](https://developer.spotify.com/documentation/web-api/) and is designed to work with the [ESP32](https://www.espressif.com/en/products/socs/esp32/overview) microcontroller. 
## Dependencies
- [ArduinoJson](https://arduinojson.org/) </br>
- [WebServer](https://github.com/espressif/arduino-esp32/blob/master/libraries/WebServer/src/WebServer.h) </br>
- [WifiClientSecure](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFiClientSecure);
- [WiFi](https://www.arduino.cc/en/Reference/WiFi) </br>
- [UrlEncode](https://github.com/plageoj/urlencode) </br>
- [base64](https://github.com/Densaugeo/base64_arduino) </br>

## Setup
1. Create a new application on the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications) and copy the Client ID and Client Secret into your code. Leave the developer dashboard open as you will need to set the callback url later. </br>
2. Now you will have to use the login without a refresh token which can be implemented the following way:
```c++
#include <Arduino.h>
#include <WiFi.h>
#include "SpotifyESP32.h"

const char* SSID = "your_ssid";
const char* PASSWORD = "your_password";
const char* CLIENT_ID = "your_client_id";
const char* CLIENT_SECRET = "your_client_secret";

//Create an instance of the Spotify class Optional: you can set the Port for the webserver the debug mode(This prints out data to the serial monitor) and number of retries
Spotify sp(CLIENT_ID, CLIENT_SECRET);

void setup() {
    Serial.begin(115200);
    connect_to_wifi();//Connect to your wifi
    
    sp.begin();//Start the webserver
    while(!sp.is_auth()){//Wait for the user to authenticate
        sp.handle_client();//Handle the client, this is necessary otherwise the webserver won't work
    }
    Serial.println("Authenticated");
}

void loop() {
    //Add your code here
}
void connect_to_wifi(){
    WiFi.begin(SSID, PASSWORD);
    Serial.print("Connecting to WiFi...");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.printf("\nConnected to WiFi\n");
}
```
3. When exectuting this code you will have to open the serial monitor and copy the url into your browser. On this page you Login with your spotify account (Most endpoints only work with premium) and authorize your application to access your data. Now you will be redirected to page where your refresh token will be shown. So you don't have to open the browser every time you restart your esp i would recommend to copy the refresh token and use the other constructor afterwards:
```c++
#include <Arduino.h>
#include <WiFi.h>
#include "SpotifyESP32.h"

const char* SSID = "your_ssid";
const char* PASSWORD = "your_password";
const char* CLIENT_ID = "your_client_id";
const char* CLIENT_SECRET = "your_client_secret";
const char* REFRESH_TOKEN = "your_refresh_token";

//Create an instance of the Spotify class Optional: Debug mode and number of retries
Spotify sp(CLIENT_ID, CLIENT_SECRET, REFRESH_TOKEN);

void setup() {
    Serial.begin(115200);
    connect_to_wifi();//Connect to your wifi
}

void loop() {
    //Add your code here
}
void connect_to_wifi(){
    WiFi.begin(SSID, PASSWORD);
    Serial.print("Connecting to WiFi...");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.printf("\nConnected to WiFi\n");
}
```
4. Now you can use the library. </br>
## Usage
- The normal functions return an response object. You can get the http code of type int with ```response_obj.status_code``` and the response message of type String with ```response_obj.reply```. </br>
To print the response you can use the ```print_response(response_obj)``` function. </br>
- To search for methods you can use the [Spotify Web API Reference](https://developer.spotify.com/documentation/web-api/reference/), all of the methods shown there are implemented </br>
- There are also some helper and some simplified functions. </br>
```c++
    // Get the current track name and return it as String
    String current_track_name();
    // Get the current track id and return it as String
    String current_track_id();
    // Get the current device id and return it as String
    String current_device_id();
    // Get the current artist names and return it as String
    String current_artist_names();
    // Get the current device id and return a pointer to the char, can be used as parameter for other functions
    char* current_device_id(char* device_id);
    // Get the current track id and return a pointer to the char, can be used as parameter for other functions
    char* current_track_id(char* track_id);
    // Get the current track name and return a pointer to the char, can be used as parameter for other functions
    char* current_track_name(char* track_name);
    // Get the current artist names and return a pointer to the char, can be used as parameter for other functions
    char* current_artist_names(char* artist_names);
    //Check if the device is playing and return true or false
    bool is_playing();
    //Check if volume of the current device is modifyable and return true or false
    bool volume_modifyable();
    //Convert a context id to a uri and return it as a char
    char convert_id_to_uri(char* id, char* type);
    //Convert a context id to a uri and return it as a pointer to the char
    char* convert_id_to_uri(char* id, char* type, char* uri); 
```
- You can also include the namespace: ```using namespace Spotify_types;```. These are some types used in the library eg. TYPES, SIZES of uris/ids </br>
## Useful Information
Due to the Wifi and HTTPS this library is quite big, so i recommend setting the partitions table of your esp to use more than the default 1.2MB of flash as most esp32's have 4MB of which the biggest part is set for SPIFFS which in most cases isn't used. Here are some tutorials about setting the partitions table of your esp: </br>
- [Official Espressif Documentation](https://espressif-docs.readthedocs-hosted.com/projects/arduino-esp32/en/latest/tutorials/partition_table.html?highlight=partitions)</br>
- [Partition Table in the Arduino IDE](https://robotzero.one/arduino-ide-partitions/)</br>
- [Partition Table in Platform IO](https://docs.platformio.org/en/latest/platforms/espressif32.html)</br>
## Trouble Shooting
- If you have any problems with the library you can use the debug mode to print out the data to the serial monitor. I recommend not setting the baud rate lower than 115200 as the data printed can be quite large which can lead to crash if the Serial communication is too slow. </br>
- If you have any problems with the library you can also use the [Spotify Web API Console](https://developer.spotify.com/console/) to test the endpoints. </br>
- If there are still issues you can open an issue on this repository. </br>
## Working Devices
- ESP32 WROOM</br>
- Should also work on ESP2866 and other ESP32 models(Untested).</br>
