#include <TM1637Display.h>
#include <ESP8266WiFi.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#include <ArduinoJson.h>
const char *ssid = "ssid";
const char *password = "password";

WiFiUDP ntpUDP;
ESP8266WebServer server(80);

//display config
const int CLK = 16; //clock pin
const int DIO = 14;//data? pin
TM1637Display display(CLK, DIO);

 //defaults
int daybrightness = 5; //display brightness daylight
int nightbrightness = 2; // display brightness night
int firsthourday = 6; //dawn hour
int lasthourday = 20; //dusk hour

int displaybrightness = 4;//display current brightness

//set ntp pool and substract 3 hours(-10800 seconds)
NTPClient timeClient(ntpUDP, "0.south-america.pool.ntp.org", -10800);



//handle not found on webserver
void handleNotFound() {
  server.send(200, "text/plain", "Not Found.");
}

//sets defaults to recieved parameters
void handleSetValues() {
  if (server.hasArg("plain") == false) {  //Check if body received
    server.send(200, "text/plain", "Body not received");
    return;
  }

  String body = server.arg("plain");
  DynamicJsonDocument doc(1024);
  auto error = deserializeJson(doc, body);

  if (error) {
    Serial.print(F("deserializeJson() failed with code "));
    Serial.println(error.c_str());

    server.send(200, "text/plain", "\"Fail\"");
    return;
  }

  daybrightness = doc["daybrightness"];
  nightbrightness = doc["nightbrightness"];
  firsthourday = doc["firsthourday"];
  lasthourday = doc["lasthourday"];
  server.send(200, "text/plain", "\"loud and clear\"");
  return;
}

//sends defaults to client
void handleGetData() {
  StaticJsonDocument<200> doc;
  doc["daybrightness"] = daybrightness;
  doc["nightbrightness"] = nightbrightness;
  doc["firsthourday"] = firsthourday;
  doc["lasthourday"] = lasthourday;
  //String(daybrightness) + "," + String(nightbrightness) + "," + String(firsthourday) + "," + String(lasthourday)
  String buf;
  serializeJson(doc, buf);
  server.send(200, "text/plain", buf);
  return;
}

void setup() {

  Serial.begin(115200);

  display.setBrightness(4);//sets initial brightness
  //--wifi config--
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    display.showNumberDecEx(0000, 0x40, false, 4, 0);
    delay(500);
    Serial.print(".");
  }
  //--wifi config--
  display.showNumberDecEx(WiFi.localIP()[3], 0, false, 4, 0);

  Serial.println();
  timeClient.begin();

  server.handleClient();
  if (MDNS.begin("esp8266")) {
    Serial.println("MDNS responder started");
  }
  //--webserver confi--
  server.on("/", handleNotFound);
  server.on("/setvalues", handleSetValues);
  server.on("/getdata", handleGetData);
  server.onNotFound(handleNotFound);

  server.enableCORS(true);
  server.begin();
  //--webserver confi--
}

//applyes brightness value for night / day
void applyValues(int hours) {
  if(hours >= firsthourday || hours <= lasthourday){
    displaybrightness = daybrightness;
    return;
  }
  displaybrightness = nightbrightness;
}


//--delay props--
const long interval = 1000;
unsigned long previousMillis = 0;
int colonState = 0;
//--delay props--


int hours = 0;//hours in global scope, maybe should work in another solution 
void loop() {
  unsigned long currentMillis = millis();
  applyValues(hours);
  if (currentMillis - previousMillis >= interval) {
    
    previousMillis = currentMillis;//resets interval
    timeClient.update();//updates ntp time

    //--sets minutes/hours--
    int minutes = timeClient.getMinutes();
    hours = timeClient.getHours();
    //--sets minutes/hours--
    
    //--makes colon blink--
    if (colonState == 0) {
      colonState = 1;
      display.showNumberDecEx(hours, 0x40, true, 2, 0);
      display.showNumberDecEx(minutes, 0, true, 2, 2);
    } else {
      colonState = 0;
      display.showNumberDecEx(hours, 0, true, 2, 0);
      display.showNumberDecEx(minutes, 0, true, 2, 2);
    }
    //--makes colon blink--
    MDNS.update();//idk if this is useful
  }
  server.handleClient();//handle webserver requests

  display.setBrightness(displaybrightness);//set display brightness
}


