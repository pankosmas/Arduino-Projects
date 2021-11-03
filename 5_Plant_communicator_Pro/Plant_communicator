#include "arduino_secrets.h"
#include "WiFi101.h"
#include "WiFiSSLClient.h"
#include "RTCZero.h"
#include "ThingSpeak.h"

const char* ssid = SECRET_SSID;    //  your network SSID (name)
const char* password = SECRET_PSWD;  // your network password
String httpsRequest = SECRET_REQUEST; // your Blynk API token

const char* host = "hooks.zapier.com";
WiFiSSLClient ZapierClient;
WiFiClient  ThingSpeakClient;

unsigned long myChannelNumber = 10;
const char * myWriteAPIKey = SECRET_WRITE_API;

RTCZero rtc; // create RTC object

/* Change these values to set the current initial time */
const byte seconds = 0;
const byte minutes = 28;
const byte hours = 17;

/* Change these values to set the current initial date */
const byte day = 4;
const byte month = 12;
const byte year = 17;


int lightPin = A0; //the analog pin the light sensor is connected to
int tempPin = A1; //the analog pin the TMP36's Vout (sense) pin is connected to
int moisturePin= A2; 

// Set this threeshold accordingly to the resistance you used
// The easiest way to calibrate this value is to test the sensor in both dry and wet earth 
int threeshold= 800; 

bool alert_already_sent=false;
bool email_already_sent=true;
bool already_sent_to_cloud=true;

void setup() {
    Serial.begin(9600);
    while(!Serial);
    delay(2000);
    Serial.print("Connecting Wifi: ");
    Serial.println(ssid);
    while (WiFi.begin(ssid, password) != WL_CONNECTED) {
      Serial.print(".");
      delay(500);
    }
  Serial.println("");
  Serial.println("WiFi connected");
  
  
  rtc.begin(); // initialize RTC 24H format
  rtc.setTime(hours, minutes, seconds);
  rtc.setDate(day, month, year);
  rtc.setAlarmTime(17, 30, 0);  // Set the time for the Arduino to send the email
  
  ThingSpeak.begin(ThingSpeakClient);
  
  rtc.setAlarmTime(0, 0, 0);    //in this way the request is sent every minute at 0 seconds
  rtc.enableAlarm(rtc.MATCH_SS);
  rtc.attachInterrupt(thingspeak_alarm);

}

void loop() {
  
  
  String warning="";
  // Send an extra email only if the plant needs to be waterd
  if(get_average_moisture() < threeshold && !alert_already_sent){ 
    warning ="Warning your plant needs water !"; // Insert here your emergency message
    warning.replace(" ", "%20");  // replace blank spaces with the URL encoded equivalent
    send_email(get_temperature(), get_average_moisture(),get_light(), warning);
    alert_already_sent=true; // Send the alert only once
  } 
  
  // Send the daily email
  if(rtc.getHours() == 17 && rtc.getMinutes() == 30){
    Serial.println("riggered");
    send_email(get_temperature(), get_average_moisture(),get_light(), warning);
    alert_already_sent = false;
  }
  
  if(!already_sent_to_cloud){
    ThingSpeak.setField(1,get_light());
    ThingSpeak.setField(2,get_temperature());
    ThingSpeak.setField(3,get_average_moisture());
    ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey); 
    already_sent_to_cloud=true;
    Serial.println("message sent to cloud");
  }
}

float get_temperature(){
  int reading = analogRead(tempPin);  
  float voltage = reading * 3.3;
  voltage /= 1024.0; 
  
 // Print tempeature in Celsius
 float temperatureC = (voltage - 0.5) * 100 ; //converting from 10 mv per degree wit 500 mV offset

 // Convert to Fahrenheit
 // float temperatureF = (temperatureC * 9.0 / 5.0) + 32.0;
 
 return temperatureC;
}
int get_average_moisture(){ // make an average of 10 values to be more accurate
  
  int tempValue=0; // variable to temporarly store moisture value
  for(int a=0; a<10; a++){
    tempValue+=analogRead(moisturePin);
    delay(10);
  }
  return tempValue/10;
}
int get_light(){
  int light_value=analogRead(A0);
  return light_value;
}

void thingspeak_alarm(){
    already_sent_to_cloud=false;
}

void send_email(float temperature, int moisture, int light, String warning){
  
  // convert values to String
  String _temperature = String(temperature);
  String _moisture = String(moisture);
  String _light = String(light);
  String _warning = warning;
  
  ZapierClient.stop(); 
  ThingSpeakClient.stop(); 

  if (ZapierClient.connect(host, 443)) {
    ZapierClient.println("POST "+httpsRequest+"?temperature="+_temperature+"&moisture="+_moisture+"&light="+_light+"&warning="+_warning+" HTTP/1.1");
    ZapierClient.println("Host: "+ String(host));
    ZapierClient.println("Connection: close");
    ZapierClient.println();
    
    delay(1000);
    while (ZapierClient.available()) { // Print on the console the answer of the server
      char c = ZapierClient.read();
      Serial.write(c);
    }
    ZapierClient.stop();  // Disconnect from the server
  }
  else {
    Serial.println("Failed to connect to client");
  }
}




