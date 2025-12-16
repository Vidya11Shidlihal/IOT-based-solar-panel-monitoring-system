Project Title:IoT Based Solar Panel Monitoring System Using ESP32

Problem Statement:The main problem was that solar panel performance was checked manually, so faults like voltage drop, overheating, or reduced output were not detected early, leading to energy loss and inefficient operation.

purpose/Objective:The purpose of this project is to monitor a solar panel’s performance continuously, detect any faults quickly, and ensure it produces maximum energy efficiently.
Solution Approach: An ESP-WROOM-32 (ESP32) microcontroller is used to collect solar panel voltage, temperature, and sunlight intensity through sensors. The data is displayed locally and transmitted to the ThingSpeak cloud platform for real-time and historical analysis.

Methodology:
*ESP32 is used as the main controller
*Voltage, temperature, and light sensors are interfaced with the solar panel
*Sensor data is displayed on an LCD
*Data is transmitted to ThingSpeak using Wi-Fi
*Performance is analyzed using real-time and historical graphs

Results:
*Successful real-time monitoring of solar panel parameters
*Remote access to data through cloud platform
*Early detection of faults such as voltage drops and overheating
*Reduced manual inspection and improved system efficiency

Applications:
*Solar power plants
*Rooftop solar systems
*Renewable energy monitoring
*Smart energy management systems

Future Scope:
*Integration of current sensor for power calculation
*Automatic alert system using SMS or mobile app
*AI-based performance prediction
*Integration with battery management systems

Key Skills Used:
Internet of Things (IoT)
ESP32 Microcontroller
Arduino Programming
Sensor Interfacing
Wi-Fi Communication
ThingSpeak Platform

code:#include <WiFi.h>
#include <LiquidCrystal_I2C.h>
#include "DHT.h"

// LCD setup
LiquidCrystal_I2C lcd(0x27, 16, 2);

// DHT22 setup
#define DHTPIN 13       // Pin connected to DHT22 Data pin
#define DHTTYPE DHT22   // DHT sensor type
DHT dht(DHTPIN, DHTTYPE);

// Light sensor setup
const int ldrPin = 34; // Light sensor connected to GPIO34

// Voltage sensor setup
const int voltageSensor = 32; // Voltage sensor connected to GPIO32
const float R1 = 30000.0, R2 = 7500.0; // Resistor values for voltage divider

// WiFi and ThingSpeak configuration
String apiKey = "LW0C9OA5N3Q4S720";
const char* ssid = "EC Dept";
const char* pass = "Vidya7892";
const char* server = "api.thingspeak.com";

// Initialize WiFi client
WiFiClient client;

void setup() {
  // LCD initialization
  lcd.begin();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Group 15 Initializing...");
  delay(2000);
  lcd.clear();

  // WiFi connection
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
  lcd.setCursor(0, 0);
  lcd.print("WiFi Connected");
  delay(2000);
  lcd.clear();

  // DHT sensor initialization
  dht.begin();
}

void loop() {
  // Read temperature
  float tempC = dht.readTemperature();
  if (isnan(tempC)) {
    lcd.setCursor(0, 0);
    lcd.print("DHT Error");
    delay(2000);
    return;
  }

  // Read light intensity
  float lux = readLightIntensity();

  // Read solar voltage
  float solarVoltage = readSolarVoltage();

  // Display Voltage and Light Intensity
  lcd.setCursor(0, 0);
  lcd.print("Voltage: ");
  lcd.print(solarVoltage, 2);
  lcd.setCursor(0, 1);
  lcd.print("Light: ");
  lcd.print(lux, 1);
  delay(3000);

  // Display Temperature and Light Intensity
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(tempC, 1);
  lcd.print("C");
  lcd.setCursor(0, 1);
  lcd.print("Light: ");
  lcd.print(lux, 1);
  lcd.print("%");
  delay(3000);

  // Send data to ThingSpeak
  sendDataToThingSpeak(tempC, solarVoltage, lux);

  delay(1500);
}

// Solar Voltage Reading Function
float readSolarVoltage() {
  int adcValue = analogRead(voltageSensor);
  float vOUT = adcValue * (3.3 / 4095.0);
  float vIN = vOUT * ((R1 + R2) / R2);
  return vIN > 0 ? vIN : 0.0;
}

// Light Intensity Reading Function (Modified with merged code)
float readLightIntensity() {
  // Read the analog value from the LDR
  int ldrValue = analogRead(ldrPin);
  
  // Map the value to a percentage (for simplicity)
  float lightIntensity = map(ldrValue, 0, 4095, 0, 100);

  // Return light intensity as lux
  return lightIntensity;
}

// Send Data to ThingSpeak
void sendDataToThingSpeak(float tempC, float solarVoltage, float lux) {
  if (client.connect(server, 80)) {
    String postStr = apiKey;
    postStr += "&field1=" + String(tempC, 2);
    postStr += "&field2=" + String(solarVoltage, 2);
    postStr += "&field3=" + String(lux, 2);
    postStr += "\r\n\r\n";

  client.print("POST /update HTTP/1.1\n");
    delay(100);
    client.print("Host: api.thingspeak.com\n");
    delay(100);
    client.print("Connection: close\n");
    delay(100);
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    delay(100);
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    delay(100);
    client.print("Content-Length: ");
    delay(100);
    client.print(postStr.length());
    delay(100);
    client.print("\n\n");
    delay(100);
    client.print(postStr);
    delay(100);
  }
  client.stop();
  Serial.println("Sending....");
  delay(15000);
}
 
float mapfloat(float x, float in_min, float in_max, float out_min, float out_max) 
{
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}
