# HEART-RATE-MONITORING
#define BLYNK_TEMPLATE_ID "HEART MONITOR"
#define BLYNK_TEMPLATE_NAME "Pulse Monitor"
#define BLYNK_AUTH_TOKEN "rflv2oV6Q5CW_kt9CPibyZ-KfrDScbgp"

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>

// ---- OLED setup ----
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// ---- WiFi & Blynk ----
char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "Realme 12x 5G";
char pass[] = "00000000";

// ---- Pulse Sensor Setup ----
const int pulsePin = 34; // analog input
int Signal;               
int Threshold = 550;      // Adjust based on your sensor readings

// Variables for BPM calculation
unsigned long lastBeatTime = 0;
int BPM = 0;
boolean Pulse = false;
unsigned long lastDisplay = 0;
int beatCount = 0;

void setup() {
  Serial.begin(115200);
  Blynk.begin(auth, ssid, pass);

  // OLED init
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 init failed"));
    for (;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 10);
  display.println("Initializing...");
  display.display();
}

void loop() {
  Blynk.run();

  Signal = analogRead(pulsePin);

  unsigned long currentTime = millis();

  // Detect rising edge
  if (Signal > Threshold && Pulse == false) {
    Pulse = true;
    unsigned long timeBetweenBeats = currentTime - lastBeatTime;
    lastBeatTime = currentTime;

    if (timeBetweenBeats > 300 && timeBetweenBeats < 2000) { // Filter false readings
      BPM = 60000 / timeBetweenBeats;
      Serial.print("BPM: ");
      Serial.println(BPM);
      beatCount++;
    }
  }

  if (Signal < Threshold && Pulse == true) {
    Pulse = false;
  }

  // Update OLED & Blynk every 1 sec
  if (currentTime - lastDisplay > 1000) {
    display.clearDisplay();
    display.setTextSize(1);
    display.setCursor(0, 10);
    display.print("Heart Rate: ");
    display.println(BPM);
    display.print("Signal: ");
    display.println(Signal);
    display.display();

    // Send BPM to Blynk
    Blynk.virtualWrite(V1, BPM);

    lastDisplay = currentTime;
  }
}
