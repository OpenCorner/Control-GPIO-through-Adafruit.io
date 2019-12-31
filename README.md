# Controll-GPIO-through-Adafruit.io
You can control your hardware GPIO's through Adafruit.io

# First of all, you need to create an account on io.adafruit
##################################################################################
Now, follow the given steps to create your new dashboard. 
**Dashboards > Actions > Create a New Dashboard > Create.**

Now you can add a block according to your requirements.

Click on "+" sign located at right top corner of your dashboard.

Click on "Toggle" block.

One window named "choose your feed name" has been appeared on screen.

Now, provide a new feed name and hit on create (note: make sure your feed name doesn't contain any white spaces and special char's).

Select the feed name which you have just created and go to the "Next Step".

Provide a name for your block just for an identification. e.g. Smart Plug, Hall Dimmer Light, etc.


Provide a name "ON" and "OFF" as your button on text and button off text respectively. You can add "1" and "0" as well.

Now, hit on "Create" to create a block.

Whoaaa!!!  You have successfully created a block!

###################################################################################
# Arduino code

/***************************************************
Control GPIO through Adafruit MQTT
****************************************************/
#include <ESP8266WiFi.h>
#include "Adafruit_MQTT.h"
#include "Adafruit_MQTT_Client.h"

/************************* WiFi Access Point *********************************/

#define WLAN_SSID       "INCORE_AP_IoT" //SSID of your router
#define WLAN_PASS       "Welcome@Incore" // Password of your router

/************************* Adafruit.io Setup *********************************/

#define AIO_SERVER      "io.adafruit.com"
#define AIO_SERVERPORT  1883                   // use 8883 for SSL
#define AIO_USERNAME    "bhavikjp"
#define AIO_KEY         "55c0d1019570446cae5c095b77a8532c"

/************ Global State (you don't need to change this!) ******************/

// Create an ESP8266 WiFiClient class to connect to the MQTT server.
WiFiClient client;
// or... use WiFiFlientSecure for SSL
//WiFiClientSecure client;

// Setup the MQTT client class by passing in the WiFi client and MQTT server and login details.
Adafruit_MQTT_Client mqtt(&client, AIO_SERVER, AIO_SERVERPORT, AIO_USERNAME, AIO_KEY);

/****************************** Feeds ***************************************/

// Setup a feed called 'photocell' for publishing.
// Notice MQTT paths for AIO follow the form: <username>/feeds/<feedname>
//Adafruit_MQTT_Publish photocell = Adafruit_MQTT_Publish(&mqtt, AIO_USERNAME "/feeds/photocell");

// Setup a feed called 'onoff' for subscribing to changes.
const char Relay1[] PROGMEM = AIO_USERNAME "/feeds/Light";
Adafruit_MQTT_Subscribe Light= Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME "/feeds/smartPlug");

const char Relay2[] PROGMEM = AIO_USERNAME "/feeds/Fan";
Adafruit_MQTT_Subscribe Fan= Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME "/feeds/smartLight");

/*************************** Sketch Code ************************************/

// Bug workaround for Arduino 1.6.6, it seems to need a function declaration
// for some reason (only affects ESP8266, likely an arduino-builder bug).
void MQTT_connect();

void setup() {
  Serial.begin(115200);
  delay(10);
  pinMode(D1,OUTPUT);
  pinMode(D2,OUTPUT);

  Serial.println(F("Adafruit MQTT demo"));

  // Connect to WiFi access point.
  Serial.println(); Serial.println();
  Serial.print("Connecting to ");
  Serial.println(WLAN_SSID);

  WiFi.begin(WLAN_SSID, WLAN_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.println("WiFi connected");
  Serial.println("IP address: "); Serial.println(WiFi.localIP());

  // Setup MQTT subscription for onoff feed.
  mqtt.subscribe(&Light);
  mqtt.subscribe(&Fan);
}

uint32_t x=0;

void loop() {
  // Ensure the connection to the MQTT server is alive (this will make the first
  // connection and automatically reconnect when disconnected).  See the MQTT_connect
  // function definition further below.
  MQTT_connect();

  // this is our 'wait for incoming subscription packets' busy subloop
  // try to spend your time here

  Adafruit_MQTT_Subscribe *subscription;
  while ((subscription = mqtt.readSubscription(5000))) {
    if (subscription == &Light) {
      Serial.print(F("Got_Light: "));
      Serial.println((char *)Light.lastread);
      uint16_t num = atoi((char *)Light.lastread);
      digitalWrite(D1,num);
    }
    if (subscription == &Fan) {
      Serial.print(F("Got_Fan: "));
      Serial.println((char *)Fan.lastread);
      uint16_t num = atoi((char *)Fan.lastread);
      digitalWrite(D2,num);
    }
  }

  // Now we can publish stuff!
//  Serial.print(F("\nSending photocell val "));
//  Serial.print(x);
//  Serial.print("...");
//  if (! photocell.publish(x++)) {
//    Serial.println(F("Failed"));
//  } else {
//    Serial.println(F("OK!"));
//  }

  // ping the server to keep the mqtt connection alive
  // NOT required if you are publishing once every KEEPALIVE seconds
  /*
  if(! mqtt.ping()) {
    mqtt.disconnect();
  }
  */
}

// Function to connect and reconnect as necessary to the MQTT server.
// Should be called in the loop function and it will take care if connecting.
void MQTT_connect() {
  int8_t ret;

  // Stop if already connected.
  if (mqtt.connected()) {
    return;
  }

  Serial.print("Connecting to MQTT... ");

  uint8_t retries = 3;
  while ((ret = mqtt.connect()) != 0) { // connect will return 0 for connected
       Serial.println(mqtt.connectErrorString(ret));
       Serial.println("Retrying MQTT connection in 5 seconds...");
       mqtt.disconnect();
       delay(5000);  // wait 5 seconds
       retries--;
       if (retries == 0) {
         // basically die and wait for WDT to reset me
         while (1);
       }
  }
  Serial.println("MQTT Connected!");
}

#####################################################################################
