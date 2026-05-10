// --- BLYNK CREDENTIALS ---
#define BLYNK_TEMPLATE_ID "input id here"
#define BLYNK_TEMPLATE_NAME "your template name"
#define BLYNK_AUTH_TOKEN "your blink token"

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <Arduino.h>

// --- WiFi Credentials ---
char ssid[] = "your wifi name";
char pass[] = "your wifi password";

// --- ESP32 Pin Mapping ---
#define LIGHTS_PIN      14  
#define LEFT_FAN_PIN    26  
#define TEACHER_FAN_PIN 25  
#define RIGHT_FAN_PIN   27  
#define EXHAUST_FAN_PIN 32  
#define MQ135_PIN       33  

// --- Threshold & Timers ---
#define AIR_QUALITY_THRESHOLD 300 
BlynkTimer timer;

// --- Autonomous Sensor & App Sync Function ---
void updateSensorData() {
  int airQualityValue = analogRead(MQ135_PIN);
  
  // 1. Math: Raw value ko 0-100 percentage mein map karna
  int pollutionPercentage = map(airQualityValue, 0, 4095, 0, 100);
  
  // 2. Analytical Logic: Text status set karna
  String airStatus = "";
  if (airQualityValue < 150) {
    airStatus = "Clean";
  } 
  else if (airQualityValue <= 300) {
    airStatus = "Normal";
  } 
  else {
    airStatus = "Suffocation!";
  }

  // 3. Data Packaging: Percentage aur text ko mila kar V0 par bhejna
  String finalDisplay = "Pollution: " + String(pollutionPercentage) + "% | Status: " + airStatus;
  Blynk.virtualWrite(V0, finalDisplay);

  // Debugging output
  Serial.print("[SENSOR] ");
  Serial.println(finalDisplay);

  // 4. Autonomous Hardware Control (Exhaust)
  if (airQualityValue > AIR_QUALITY_THRESHOLD) {
    if (digitalRead(EXHAUST_FAN_PIN) == HIGH) { // Agar OFF tha to ON karo
        digitalWrite(EXHAUST_FAN_PIN, LOW); 
        Blynk.virtualWrite(V1, 1);
        Serial.println("[AUTO] Exhaust ON - Gas Detected!");
    }
  } else {
    if (digitalRead(EXHAUST_FAN_PIN) == LOW) { // Agar ON tha to OFF karo
        digitalWrite(EXHAUST_FAN_PIN, HIGH); 
        Blynk.virtualWrite(V1, 0);
        Serial.println("[AUTO] Exhaust OFF - Air Clean.");
    }
  }
}

void setup() {
  Serial.begin(115200);
  analogReadResolution(12); // ESP32 ADC 0-4095
  
  // Pin Modes
  pinMode(LIGHTS_PIN, OUTPUT);
  pinMode(LEFT_FAN_PIN, OUTPUT);
  pinMode(TEACHER_FAN_PIN, OUTPUT);
  pinMode(RIGHT_FAN_PIN, OUTPUT);
  pinMode(EXHAUST_FAN_PIN, OUTPUT);
  
  // Hardware Reset: Sab relays Active LOW hain (HIGH = OFF)
  digitalWrite(LIGHTS_PIN, HIGH);
  digitalWrite(LEFT_FAN_PIN, HIGH);
  digitalWrite(TEACHER_FAN_PIN, HIGH);
  digitalWrite(RIGHT_FAN_PIN, HIGH);
  digitalWrite(EXHAUST_FAN_PIN, HIGH);

  Serial.println("--- Booting Smart Classroom System ---");
  
  // Blynk & WiFi Init
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
  
  // Timer setup (Har 2 second baad data update hoga)
  timer.setInterval(2000L, updateSensorData); 
  
  Serial.println("System Ready. Waiting for Python Commands...");
}

void loop() {
  Blynk.run();
  timer.run();

  // --- Python Command Parsing ---
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');
    command.trim(); 

    // --- Lights (V2) ---
    if (command == "LIGHTS_ON") { 
      digitalWrite(LIGHTS_PIN, LOW); 
      Blynk.virtualWrite(V2, 1); 
      Serial.println("Action: Lights ON"); 
    } 
    else if (command == "LIGHTS_OFF") { 
      digitalWrite(LIGHTS_PIN, HIGH); 
      Blynk.virtualWrite(V2, 0); 
      Serial.println("Action: Lights OFF"); 
    }
    
    // --- Left Fan (V3) ---
    else if (command == "LEFT_ON") { 
      digitalWrite(LEFT_FAN_PIN, LOW); 
      Blynk.virtualWrite(V3, 1); 
      Serial.println("Action: Left Fan ON"); 
    } 
    else if (command == "LEFT_OFF") { 
      digitalWrite(LEFT_FAN_PIN, HIGH); 
      Blynk.virtualWrite(V3, 0); 
      Serial.println("Action: Left Fan OFF"); 
    }

    // --- Teacher Fan (V4) ---
    else if (command == "TEACHER_ON") { 
      digitalWrite(TEACHER_FAN_PIN, LOW); 
      Blynk.virtualWrite(V4, 1); 
      Serial.println("Action: Teacher Fan ON"); 
    } 
    else if (command == "TEACHER_OFF") { 
      digitalWrite(TEACHER_FAN_PIN, HIGH); 
      Blynk.virtualWrite(V4, 0); 
      Serial.println("Action: Teacher Fan OFF"); 
    }

    // --- Right Fan (V5) ---
    else if (command == "RIGHT_ON") { 
      digitalWrite(RIGHT_FAN_PIN, LOW); 
      Blynk.virtualWrite(V5, 1); 
      Serial.println("Action: Right Fan ON"); 
    } 
    else if (command == "RIGHT_OFF") { 
      digitalWrite(RIGHT_FAN_PIN, HIGH); 
      Blynk.virtualWrite(V5, 0); 
      Serial.println("Action: Right Fan OFF"); 
    }
  }
}
