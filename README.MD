#include <ESP8266WiFi.h>
#include <FirebaseESP8266.h>

#define WIFI_SSID "your-ssid"
#define WIFI_PASSWORD "your-password"
#define FIREBASE_HOST "https://your-project-id.firebaseio.com/"
#define FIREBASE_AUTH "your-api-key"

FirebaseData firebaseData;

int pirPin = D5;  // PIR sensor pin
bool occupied = false;

void setup() {
  Serial.begin(115200);
  pinMode(pirPin, INPUT);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected!");

  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
  Firebase.reconnectWiFi(true);
}

void loop() {
  int motion = digitalRead(pirPin);
  bool newOccupied = (motion == HIGH);

  if (newOccupied != occupied) {
    occupied = newOccupied;
    String status = occupied ? "Occupied" : "Available";
    Serial.println("Room status: " + status);

    Firebase.setString(firebaseData, "/classrooms/room101/status", status);
    Firebase.setString(firebaseData, "/classrooms/room101/timestamp", String(millis()));
  }
  delay(500);
}
