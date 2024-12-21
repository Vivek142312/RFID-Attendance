# RFID-Attendance
## Introduction 
Attendance management is crucial for the effectiveness of educational institutions, workplaces, and organizations as it influences accountability, productivity, and resource allocation. It is essential to maintain precise attendance records to track individual performances and ensure organizational norms. Manual and biometric methods of attendance management have disadvantages, such as being simple and inexpensive but prone to human error and manipulation. Recording attendance in manual books is time-consuming, especially in populated areas like universities or corporate offices. Additionally, these systems require extensive administrative work, leading to inefficiency in time and resources. As organizations grow and technological advancements transform operational activities, there is a growing need for more efficient, scalable, and reliable systems. A Smart RFID Attendance Monitoring System uses Radio Frequency Identification (RFID) and Embedded Systems to provide a solid alternative in attendance management. RFID enables identification without physical contact and at a high speed, while embedded systems provide a processing databank for effective collection, storage, and handling of attendance information.
This specialized technology operates under minimal overhead in terms of infrastructure or internet connection, reducing both price and complexity. Embedded systems ensure compactness, low power consumption, and independent performance, making them ideal for areas with limited and unreliable internet access.

## Overview 
The Smart RFID Attendance Monitoring System aims to improve attendance management by combining RFID technology with embedded systems. This innovative approach combines RFID technology with IoT to provide a dependable, scalable, and automated platform for real-time attendance monitoring. The system's infrastructure supports smooth communication among devices and cloud servers, allowing for high data loads while maintaining reliability and speed. It also includes intelligent data analytics tools, enabling administrators to generate sophisticated reports on attendance, analyze trends, and identify patterns for efficient decision-making. The Smart RFID Attendance Monitoring System offers insights on effective resource allocation and policy enforcement, and offers real-time updates, cloud storage, and remote management. It does not require flawing operating systems or RFID working systems, allowing for contactless operations regardless of organizational scale. The system also aims to lower power consumption, maintenance, and better confidentiality by avoiding sensitive body measurement indexes.
The project is designed for practical implementation and real-world usability, unlike many existing research efforts and traditional systems. Its modular architecture consists of an RFID module for data collection and a security and backup module for reliability and protection. This modular design boosts performance and simplifies upgrades and customization. The Smart RFID Attendance Monitoring System is innovative and useful for overcoming the drawbacks of conventional and automated attendance systems. Its focus on contactless operation, data analytics, user privacy, and energy efficiency makes it an innovative solution for future attendance management applications in education, business, and industry.

## Components Used
|Sl No.| Items | Qty | Description |
|1| ------------- | ------------- | ------------- |
|2|Arduiino Uno|1|Main microcontroller board|
|3|RC522|1|RFID sensor|
|4|Cards|5|RFID Cards|
|5|DS3231|1|RTC Module|
|6|16*2 I2C LCD|1|LCD Screen|
|7|Buzzer|1|Buzzer for notification|
|8|Jumper Wires|1 Set|Wires for connection|

## Circuit Diagram

![cktdia](https://github.com/user-attachments/assets/c6fcb3b4-9b91-4dd3-9bf0-9983b0c90e30)

## Code
```
#include <SPI.h>
#include <Wire.h>
#include <MFRC522.h>
#include <LiquidCrystal_I2C.h>
#include <RTClib.h>
// RFID Pins
#define SS_PIN 10
#define RST_PIN 9
// Buzzer Pin
#define BUZZER_PIN 8
// Initialize RFID, RTC, and LCD
MFRC522 mfrc522(SS_PIN, RST_PIN);
RTC_DS1307 rtc;
LiquidCrystal_I2C lcd(0x27, 16, 2); // Adjust I2C address if needed (0x27 or 0x3F)
// Array of authorized card UIDs and names
struct Card {
String uid;
String name;
};
Card authorizedCards[] = {
{"C3B16927", "Krishna"}, // User 1
{"03B5FA00", "Ram"}, // User 2
{"23A87428", "Balaram"}, // User 3
{"0EA58C02", "Ra1dha"}, // User 4
{"431C0428", "Seeta"}, // User 5
{"D9DF75C2", "Lakshman"} // User 6
};
void setup() {
Serial.begin(9600);
while (!Serial); // Wait for Serial Monitor to connect
SPI.begin();
mfrc522.PCD_Init();
// Initialize LCD
lcd.begin(16, 2); // Specify the LCD size (16 columns, 2 rows)
lcd.backlight();
lcd.clear();
lcd.print("Ready to Scan...");
// Initialize Buzzer
pinMode(BUZZER_PIN, OUTPUT);
// Initialize RTC
if (!rtc.begin()) {
Serial.println("RTC module not found!");
while (1); // Halt execution if RTC is not detected
}
// Check if the RTC is running
if (!rtc.isrunning()) {
Serial.println("RTC is not running. Setting the time...");
// Uncomment the following line if you want to set the time manually
// rtc.adjust(DateTime(2024, 12, 9, 20, 30, 0)); // Example: YYYY, MM, DD, HH, MM, SS
}
Serial.println("Enter the current date and time in this format: YYYY MM DD HH MM SS");
Serial.println("Example: 2024 12 9 20 30 0");
}
void loop() {
// Check for user input from Serial Monitor to set the RTC time
if (Serial.available()) {
String input = Serial.readStringUntil('\n');
input.trim(); // Remove extra spaces or newlines
int year, month, day, hour, minute, second;
// Parse the input string
if (sscanf(input.c_str(), "%d %d %d %d %d %d", &year, &month, &day, &hour, &minute, &second) == 6) {
// Set the RTC time
rtc.adjust(DateTime(year, month, day, hour, minute, second));
Serial.println("RTC updated successfully!");
} else {
Serial.println("Invalid format. Please try again.");
}
}
// Check if a new card is present
if (!mfrc522.PICC_IsNewCardPresent()) {
return;
}
// Read the card serial number
if (!mfrc522.PICC_ReadCardSerial()) {
return;
}
// Get UID as a string
String uid = "";
for (byte i = 0; i < mfrc522.uid.size; i++) {
uid += String(mfrc522.uid.uidByte[i], HEX);
}
uid.toUpperCase(); // Convert UID to uppercase
Serial.print("Detected UID: ");
Serial.println(uid);
// Check if the UID matches any authorized card
bool authorized = false;
for (Card card : authorizedCards) {
if (card.uid == uid) {
displayAccess(card.name, true);
authorized = true;
break;
}
}
if (!authorized) {
displayAccess("Access Denied", false);
}
// Halt RFID communication
mfrc522.PICC_HaltA();
// Print the current time every second
DateTime now = rtc.now();
printDateTime(now);
delay(1000);
}
// Function to display access result
void displayAccess(String message, bool isAuthorized) {
lcd.clear();
if (isAuthorized) {
lcd.print("Access Granted");
lcd.setCursor(0, 1);
lcd.print(message); // Display user's name
Serial.println("Access Granted: " + message);
digitalWrite(BUZZER_PIN, HIGH); // Turn on buzzer for authorized card
delay(500);
digitalWrite(BUZZER_PIN, LOW); // Turn off buzzer
}
else {
lcd.print(message); // Display "Access Denied"
Serial.println(message);
for (int i = 0; i < 3; i++) { // Buzz 3 times for denied access
digitalWrite(BUZZER_PIN, HIGH);
delay(200);
digitalWrite(BUZZER_PIN, LOW);
delay(200);
}
}
delay(2000); // Wait for 2 seconds
}
// Function to print the current date and time in human-readable format
void printDateTime(DateTime now) {
Serial.print("Date: ");
Serial.print(now.year());
Serial.print("-");
Serial.print(now.month());
Serial.print("-");
Serial.println(now.day());
Serial.print("Time: ");
Serial.print(now.hour());
Serial.print(":");
Serial.print(now.minute());
Serial.print(":");
Serial.println(now.second());
lcd.setCursor(0, 0);
lcd.print("Date: ");
lcd.print(now.year());
lcd.print("-");
lcd.print(now.month());
lcd.print("-");
lcd.print(now.day());
lcd.setCursor(0, 1);
lcd.print("Time: ");
lcd.print(now.hour());
lcd.print(":");
lcd.print(now.minute());
lcd.print(":");
lcd.print(now.second());
}
```
## Hardware Components Description
### Arduino UNO
The Arduino Uno is the main microcontroller board that continuously reads and executes the instructions stored in its memory. In attendance systems, each student or user is assigned a unique RFID tag, and the reader scans the tag to record attendance. This eliminates manual processes and improves accuracy, as the system can uniquely identify each individual. Arduino Uno is based on the ATmega328P by Atmel. The Arduino Uno pinout consists of 14 digital pins, 6 analog inputs, a power jack, a USB connection, and an ICSP header. The board can be powered by 5-20 volts, but the manufacturer recommends to keep it between 7-12 volts.

![ARDUINO](https://github.com/user-attachments/assets/738253cf-de0f-4705-a9dd-a51abe3017d4)

### RFID-RC522
The RC522 is a 13.56MHz RFID module based on NXP semiconductors' MFRC522 controller. The module supports I2C, SPI, and UART and is typically delivered with an RFID card and key fob. It is often used in attendance systems and other applications that require identifying people or objects. The tag responds to the electromagnetic field generated by the RC522, transmitting its unique data back to the reader. In attendance systems, each student or user is assigned a unique RFID tag, and the reader scans the tag to record attendance. This eliminates manual processes and improves accuracy, as the system can uniquely identify each individual. It features a built-in crystal oscillator for high precision and low drift, ensuring reliable timekeeping.

![RC522](https://github.com/user-attachments/assets/889eaffe-f655-4264-b7d4-9bd607c10003)

### DS3231 RTC Module
The term RTC refers to a real-time clock. RTC modules are TIME as well as DATE remembering devices with a battery configuration that keeps the module working even when no external power is available. This maintains the TIME and DATE up to date. So, we can get precise TIME and DATE from the RTC module whenever we want. The DS3231 is a very accurate real-time clock module that provides time management capability, which is critical for systems that require precise timestamps. It contains a temperature-compensated crystal oscillator, which ensures minimal drift and long-term reliability. In attendance networks, the DS3231 captures the precise date and time of RFID tag scans, allowing for accurate attendance tracking data.

![DS3231](https://github.com/user-attachments/assets/f29fc3a7-9286-470f-a2e2-0f4f95b6e4dd)

### LCD Module
16x2 LCD modules are widely utilised in embedded projects due to their low cost, availability, programmer friendliness, and accessibility to educational resources. It consists of millions of small pixels used to show data. For example, when a person scans their RFID tag, the LCD may show messages such as "Attendance Recorded" or "Access Denied." This visual feedback improves user interaction and makes the system easier to operate.

![LCD](https://github.com/user-attachments/assets/d61d1646-7858-4306-9129-a843053703e2)


## Output Pictures
![op](https://github.com/user-attachments/assets/234f7fa1-4137-49f4-bea1-c9731bee49c9)
          **Output when the access card is accepted**

![opd](https://github.com/user-attachments/assets/f4c3e36f-99ad-4f1d-9a35-c299e501232f)
          **Output when the access card is denied**

## Output Progress video

https://github.com/user-attachments/assets/610a21fa-0fb0-43fe-891c-f56f36085781



