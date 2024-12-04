# Controlling-Actuators-using-Web-Server
#include <WiFi.h>
#include <WiFiAP.h>
#include <WiFiClient.h>
#include <WiFiGeneric.h>
#include <WiFiMulti.h>
#include <WiFiSTA.h>
#include <WiFiScan.h>
#include <WiFiServer.h>
#include <WiFiType.h>
#include <WiFiUdp.h>
#include <Wire.h>
#include <TinyGPS++.h>
#include <Adafruit_MQTT.h>
#include <Adafruit_MQTT_Client.h>

#define GAS_SENSOR_PIN 32
#define BUZZER_PIN 2
#define TOUCH_SENSOR_PIN 33  // Pin to which the touch sensor is connected

TinyGPSPlus gps;

bool gasDetected = false;

#define WLAN_SSID       "realme"
#define WLAN_PASS       "7549903908"

// Adafruit IO Setup
#define AIO_SERVER      "io.adafruit.com"
#define AIO_SERVERPORT  1883
#define AIO_USERNAME    "himanshuraz"
#define AIO_KEY         "aio_xnvn44s1xfu2L4QH8hm89Cvbnd8p"

WiFiClient client;
Adafruit_MQTT_Client mqtt(&client, AIO_SERVER, AIO_SERVERPORT, AIO_USERNAME, AIO_KEY);

// Adafruit IO Feeds                                                               
Adafruit_MQTT_Publish gasFeed = Adafruit_MQTT_Publish(&mqtt, AIO_USERNAME "/feeds/gas");
Adafruit_MQTT_Publish touchFeed = Adafruit_MQTT_Publish(&mqtt, AIO_USERNAME "/feeds/piezor");

void MQTT_connect();

void setup() {
  Serial.begin(115200);
  pinMode(GAS_SENSOR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(TOUCH_SENSOR_PIN, INPUT);
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
}

void loop() {
  MQTT_connect();
  int gasValue = analogRead(GAS_SENSOR_PIN);
  publishGasValue(gasValue);
  int touchValue = analogRead(TOUCH_SENSOR_PIN);
  publishTouchValue(touchValue);
  if (gasValue > 1400) {
    buzzerAlert();
  }
  delay(10000);
}

void buzzerAlert() {
  digitalWrite(BUZZER_PIN, HIGH); // Turn the buzzer on
  delay(500); // Buzz for 0.5 seconds
  digitalWrite(BUZZER_PIN, LOW); // Turn the buzzer off
}

void publishGasValue(int value) {
  Serial.print("Publishing gas value to Adafruit IO: ");
  Serial.println(value);
  gasFeed.publish(value);
}

void publishTouchValue(int value) {
  Serial.print("Publishing touch sensor value to Adafruit IO: ");
  Serial.println(value);
  touchFeed.publish(value);
}

void MQTT_connect() {
  int8_t ret;
  if (mqtt.connected()) {
    return;
  }
  Serial.print("Connecting to MQTT... ");
  uint8_t retries = 3;
  while ((ret = mqtt.connect()) != 0) {
    Serial.println(mqtt.connectErrorString(ret));
    Serial.println("Retrying MQTT connection in 5 seconds...");
    mqtt.disconnect();
    delay(5000);
    retries--;
    if (retries == 0) {
      while (1);
    }
  }
  Serial.println("MQTT Connected!");
}
