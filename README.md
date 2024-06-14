# students-attendance-system
#include <SPI.h>
#include <MFRC522.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h> // For I2C LCD
#include "RTClib.h" // For Real Time Clock

#define SS_PIN 21
#define RST_PIN 22
MFRC522 mfrc522(SS_PIN, RST_PIN);

LiquidCrystal_I2C lcd(0x27, 16, 2); //connect the Set the LCD address to 0x27 for a 16 chars and 2 line display
RTC_DS3231 rtc;

const int buzzer = 25;
void setup() {
  Serial.begin(115200);
  SPI.begin();
  mfrc522.PCD_Init();
  lcd.begin(16, 2);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("please Place your card");
  if (! rtc.begin()) {
    Serial.println("soory i Couldn't find RTC");
    while (1);
  }
  pinMode(buzzer, OUTPUT);
  digitalWrite(buzzer, LOW);

  Serial.println("please Place your card near the reader...");
  lcd.setCursor(0, 1);
  lcd.print("Near the reader");
}

void loop() {
  // Look for new cards
  if (!mfrc522.PICC_IsNewCardPresent()) {
    return;
  }
  // Select one of the cards
  if (!mfrc522.PICC_ReadCardSerial()) {
    return;
  }

  // Show UID on serial monitor
  Serial.print("UID tag :");
  String content = "";
  byte letter;
  for (byte i = 0; i < mfrc522.uid.size; i++) {
     Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
     Serial.print(mfrc522.uid.uidByte[i], HEX);
     content.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "));
     content.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  Serial.println();
  content.toUpperCase();

  // Check if the card matches any known ID
  if (content.substring(1) == "YOUR_CARD_ID_HERE") {
    Serial.println("Access Granted");
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Access Granted");
    tone(buzzer, 1000, 200); // Beep the buzzer
    logAttendance(content);
  } else {
    Serial.println("Access Denied");
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Access Denied");
    tone(buzzer, 1000, 500); // Beep the buzzer
  }
  delay(1000);
}

void logAttendance(String uid) {
  // Get current date and time from RTC module
  DateTime now = rtc.now();
  String currentTime = String(now.hour()) + ":" + String(now.minute());
  String currentDate = String(now.year()) + "-" + String(now.month()) + "-" + String(now.day());

  // Print attendance log to Serial Monitor
  Serial.print("Attendance Log: ");
  Serial.print("UID: ");
  Serial.print(uid);
  Serial.print(" Date: ");
  Serial.print(currentDate);
  Serial.print(" Time: ");
  Serial.println(currentTime);
  // Display attendance log on LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("UID:");
  lcd.setCursor(0, 1);
  lcd.print(uid.substring(1));
  delay(2000);
}
