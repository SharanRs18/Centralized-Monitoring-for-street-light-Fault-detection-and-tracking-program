# Centralized-Monitoring-for-street-light-Fault-detection-and-tracking-program
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

// Replace with your network credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Server URL
const char* serverUrl = "http://your_server.com/api/update_status";

// Pin for the light sensor
const int sensorPin = A0;

// Function to detect light status
bool isLightOn() {
  int sensorValue = analogRead(sensorPin);
  // Adjust the threshold value as needed
  return sensorValue > 500;
}

void setup() {
  Serial.begin(115200);

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin(serverUrl);

    // Prepare the JSON payload
    String payload = "{\"light_id\": \"street_light_1\", \"status\": ";
    payload += isLightOn() ? "\"ON\"" : "\"OFF\"";
    payload += "}";

    // Send the HTTP POST request
    http.addHeader("Content-Type", "application/json");
    int httpResponseCode = http.POST(payload);

    if (httpResponseCode > 0) {
      String response = http.getString();
      Serial.println(httpResponseCode);
      Serial.println(response);
    } else {
      Serial.print("Error on sending POST: ");
      Serial.println(httpResponseCode);
    }

    http.end();
  } else {
    Serial.println("WiFi Disconnected");
  }

  // Send data every 5 minutes
  delay(300000);
}
