#include <ESP8266WiFi.h>
#include "secrets.h"
#include <PubSubClient.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include "DHT.h"
#include <ThingSpeak.h>

char ssid[] = SECRET_SSID;   // your network SSID (name) 
char pass[] = SECRET_PASS;   // your network password
WiFiClient client;

unsigned long myChannelNumber = SECRET_CH_ID;
const char * myWriteAPIKey = SECRET_WRITE_APIKEY;

// DHT
#define DHTPIN 5 // Pin D1
#define DHTTYPE DHT22

// GPIO where the DS18B20 (soil temperature) is connected to
const int oneWireBus = 0;  // Pin D3

// Setup a oneWire instance to communicate with any OneWire devices
OneWire oneWire(oneWireBus);

// Pass our oneWire reference to Dallas Temperature sensor 
DallasTemperature sensors(&oneWire);

// utlrasonic pinout
#define ULTRASONIC_TRIG_PIN 12   // pin TRIG to D6
#define ULTRASONIC_ECHO_PIN 14   // pin ECHO to D5

// defines variables
long duration;
int distance;

//For capacitive soil sensor
const int AirValue = 852;   //you need to replace this value with Value_1
const int WaterValue = 480;  //you need to replace this value with Value_2
int soilMoistureValue = 0;
int soilmoisturepercent= 0;

//KY018 Photo resistor module (currently off)
//int sensorPin = A0; // select the input pin for the potentiometer
//int ledPin = 13; // select the pin for the LED
//int sensorValue = 0; // variable to store the value coming from the sensor

// Initialize DHT sensor.
DHT dht(DHTPIN, DHTTYPE);

int status = WL_IDLE_STATUS;
unsigned long lastSend;

void setup()
{
  Serial.begin(115200);
  dht.begin();
  sensors.begin();
  pinMode(ULTRASONIC_TRIG_PIN, OUTPUT); // Sets the ULTRASONIC_TRIG_PIN as an Output
  pinMode(ULTRASONIC_ECHO_PIN, INPUT); // Sets the ULTRASONIC_ECHO_PIN as an Input
  Serial.begin(9600); // Starts the serial communication
  WiFi.mode(WIFI_STA); 
  ThingSpeak.begin(client);  // Initialize ThingSpeak
  delay(10);

  InitWiFi();
  lastSend = 0;
  }
  
void loop()
{
  if ( status != WL_CONNECTED ) {
    reconnect();
  }

  if ( millis() - lastSend > 1000 ) { // Update and send only after 1 seconds
    getdata();
    lastSend = millis();
  }
  loop();
}

void getdata()
{
    Serial.println("Collecting data.");

    // Publishes new readings after every 'delay'   
    float h = dht.readHumidity();
    // Read temperature as Celsius (the default)
    float t = dht.readTemperature();

    //put Sensor insert into soil
    float sensorValue = analogRead(A0);
    soilmoisturepercent = map(sensorValue, AirValue, WaterValue, 0, 100);
    float s = soilmoisturepercent;    

    //sensor soil temperature code
    sensors.requestTemperatures(); 
    float temperatureC = sensors.getTempCByIndex(0);

    // Clears the ULTRASONIC_TRIG_PIN
    digitalWrite(ULTRASONIC_TRIG_PIN, LOW);
    delayMicroseconds(2);
    
    // Sets the ULTRASONIC_TRIG_PIN on HIGH state for 10 micro seconds
    digitalWrite(ULTRASONIC_TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(ULTRASONIC_TRIG_PIN, LOW);
    
    // Reads the ULTRASONIC_ECHO_PIN, returns the sound wave travel time in microseconds
    duration = pulseIn(ULTRASONIC_ECHO_PIN, HIGH);
    
    // Calculating the distance
    float distance = duration*0.034/2;   
    
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t) || isnan(s) || isnan(temperatureC) || isnan(distance)) 
    {
      Serial.println("Failed to read from sensors!");
      return;
    }

  // set the fields with the values for thingspeak
  ThingSpeak.setField(1, t);
  ThingSpeak.setField(2, h);
  ThingSpeak.setField(3, s);
  ThingSpeak.setField(4, temperatureC);
  ThingSpeak.setField(5, distance);

  // write to the ThingSpeak channel
  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  if(x == 200){
    Serial.println("Channel update successful.");
  }
  else{
    Serial.println("Problem updating channel. HTTP error code " + String(x));
  }  

  // for calibrating soil
  //Serial.print(sensorValue);

  Serial.println("Repeating in 15 seconds");
  delay(15000);
}

void InitWiFi()
{
  Serial.println("Connecting to AP ...");
  // attempt to connect to WiFi network

  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to AP");
}


void reconnect() {
  // Connect or reconnect to WiFi
  if(WiFi.status() != WL_CONNECTED){
    Serial.print("Attempting to connect to SSID: ");
    Serial.println(SECRET_SSID);
    while(WiFi.status() != WL_CONNECTED){
      WiFi.begin(ssid, pass);  // Connect to WPA/WPA2 network. Change this line if using open or WEP network
      Serial.print(".");
      delay(5000);     
    } 
    Serial.println("\nConnected.");
  }
}
