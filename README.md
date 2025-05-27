# SMART ASTHAMA-INHALER-TRACKER
#include <Wire.h>
#include <Adafruit_BMP085.h>
#include <LiquidCrystal_I2C.h>

// Initialize components
Adafruit_BMP085 bmp;
LiquidCrystal_I2C lcd(0x27, 16, 2); // Change I2C address if needed

// Pins
const int greenLED = 14;
const int redLED = 27;
const int xPin = 34;
const int yPin = 35;
const int zPin = 32;

// Thresholds
const float tiltThreshold = 0.2; // Adjust this based on testing
const float pressureDropThreshold = 2.0; // hPa drop indicating inhalation

// State variables
float baselinePressure = 0.0;
unsigned long lastCheck = 0;

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22); // SDA, SCL for ESP32

  // Init LCD
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Smart Inhaler");

  // Init BMP180
  if (!bmp.begin()) {
    lcd.setCursor(0, 1);
    lcd.print("BMP180 not found!");
    while (1);
  }

  // Set up LEDs
  pinMode(greenLED, OUTPUT);
  pinMode(redLED, OUTPUT);

  // Get baseline pressure
  baselinePressure = bmp.readPressure() / 100.0; // convert to hPa
  delay(1000);
}

void loop() {
  // Check every 500ms
  if (millis() - lastCheck > 500) {
    lastCheck = millis();

    float currentPressure = bmp.readPressure() / 100.0;
    float deltaPressure = baselinePressure - currentPressure;

    bool orientationOK = checkOrientation();

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Press: ");
    lcd.print(currentPressure);
    lcd.setCursor(0, 1);

    if (orientationOK && deltaPressure > pressureDropThreshold) {
      lcd.print("Dose Taken");
      digitalWrite(greenLED, HIG
