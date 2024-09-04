# ESP32 Door Lock System with Servo, LCD I2C, and 4x3 Keypad

This project demonstrates how to create a simple door lock system using an ESP32 microcontroller, a 9g servo motor, an I2C LCD display, and a 4x3 keypad. The system allows you to enter a password on the keypad, and if the correct password is entered, the servo unlocks the door.

## Table of Contents
- [Introduction](#introduction)
- [Components](#components)
- [Circuit Diagram and Wiring](#circuit-diagram-and-wiring)
- [Libraries Installation](#libraries-installation)
- [Code](#code)
- [Images](#images)
- [Usage](#usage)

## Introduction
This project is a simple and effective way to implement a password-protected lock using an ESP32. The system displays the password input on an LCD and controls a servo motor to lock or unlock the door based on the entered password.

## Components
- **ESP32**
- **Servo Motor (9g)**
- **LCD I2C (16x2)**
- **Keypad 4x3**
- **Breadboard and Jumpers**
- **Power Supply**

## Circuit Diagram and Wiring
Below is the wiring setup for the components. The pinout follows this order from left to right:

- **Column 2:** GPIO 13
- **Row 1:** GPIO 33
- **Column 1:** GPIO 27
- **Row 4:** GPIO 14
- **Column 3:** GPIO 32
- **Row 3:** GPIO 26
- **Row 2:** GPIO 25

### Wiring Table

| Component | Pin on ESP32 | Pin on Component |
|-----------|--------------|------------------|
| Keypad C1 | GPIO 27      | Column 1         |
| Keypad C2 | GPIO 13      | Column 2         |
| Keypad C3 | GPIO 32      | Column 3         |
| Keypad R1 | GPIO 33      | Row 1            |
| Keypad R2 | GPIO 25      | Row 2            |
| Keypad R3 | GPIO 26      | Row 3            |
| Keypad R4 | GPIO 14      | Row 4            |
| LCD SDA   | GPIO 21      | SDA              |
| LCD SCL   | GPIO 22      | SCL              |
| Servo     | GPIO 15      | Control (Signal) |
| Servo     | 3.3V or 5V   | VCC              |
| Servo     | GND          | GND              |

### Circuit Diagram
_Include your circuit diagram here._

## Libraries Installation

Before uploading the code to your ESP32, make sure to install the following libraries:

1. **Keypad Library**
   - Go to the Arduino IDE: `Sketch` -> `Include Library` -> `Manage Libraries`.
   - Search for `Keypad` and install it.

2. **LiquidCrystal I2C Library**
   - Go to the Arduino IDE: `Sketch` -> `Include Library` -> `Manage Libraries`.
   - Search for `LiquidCrystal I2C` and install it.

3. **ESP32Servo Library**
   - Go to the Arduino IDE: `Sketch` -> `Include Library` -> `Manage Libraries`.
   - Search for `ESP32Servo` and install it.

## Code

```cpp
#include <Keypad.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <ESP32Servo.h>

#define I2C_ADDR    0x27
#define LCD_COLUMNS 16
#define LCD_LINES   2

// Initialize LCD
LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

// Keypad configuration
const uint8_t ROWS = 4;
const uint8_t COLS = 3;
char keys[ROWS][COLS] = {
  { '1', '2', '3' },
  { '4', '5', '6' },
  { '7', '8', '9' },
  { '*', '0', '#' }
};

// Keypad pin assignments
uint8_t rowPins[ROWS] = { 33, 25, 26, 14 };  // R1 -> GPIO 33, R2 -> GPIO 25, R3 -> GPIO 26, R4 -> GPIO 14
uint8_t colPins[COLS] = { 27, 13, 32 };      // C1 -> GPIO 27, C2 -> GPIO 13, C3 -> GPIO 32

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// Servo configuration
const int servoPin = 15;
const String correctPassword = "1234"; // Define the correct password

Servo servo;
String enteredPassword = ""; // Store entered password
bool isLocked = true; // Door is initially locked

void setup() {
  Wire.begin(21, 22);  // SDA -> GPIO 21, SCL -> GPIO 22
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.print("Enter Password:");
  
  servo.attach(servoPin, 500, 2400);
  servo.write(180); // Initial position is 180 (locked)
}

void loop() {
  char key = keypad.getKey();
  
  if (key) {
    if (key == '#') {
      // Check password
      if (enteredPassword == correctPassword) {
        lcd.clear();
        lcd.print("Access Granted");
        servo.write(90); // Unlock the door (move to position 90)
        isLocked = false;
      } else {
        lcd.clear();
        lcd.print("Access Denied");
        servo.write(180); // Lock the door (move to position 180)
        isLocked = true;
      }
      enteredPassword = ""; // Clear entered password
      delay(2000); // Display message for 2 seconds
      lcd.clear();
      lcd.print("Enter Password:");
    } else if (key == '*') {
      // Clear entered password
      enteredPassword = "";
      lcd.clear();
      lcd.print("Enter Password:");
    } else {
      // Add key to entered password
      enteredPassword += key;
      lcd.setCursor(0, 1); // Move to second line
      lcd.print("Pass: ");
      lcd.print(enteredPassword);
    }
  }
  
  // Optional: Implement a way to relock the door after some time or another event
  if (!isLocked && millis() > 5000) { // 5 seconds delay
    servo.write(180);
    isLocked = true; // Door is locked again
  }
}
```

## Images

### Wiring
_Include an image of the wiring diagram here._

### Components

- **Servo 9g**

![image](https://github.com/user-attachments/assets/68b3a5e2-b8f0-4e6d-b69b-0b5ba62687cb)


- **LCD I2C (16x2)**

![image](https://github.com/user-attachments/assets/a1bf6f6c-21d8-4c4c-bd72-bc2ae5ac26ea)


- **Keypad 4x3**

![424d2263-98f3-415d-8938-4a64298e387d(1)](https://github.com/user-attachments/assets/6a34080d-987b-4246-843d-5a2d11ffde7f)


- **ESP32**
  
![image](https://github.com/user-attachments/assets/7c672bcb-6fa9-4afc-bb24-f2c2c3f6fc9d)


## Usage

1. Connect the components as described in the wiring section.
2. Upload the provided code to your ESP32.
3. Power on the ESP32, and the LCD will prompt you to enter a password.
4. Enter the password using the keypad. If the password is correct, the servo will unlock the door. If incorrect, it will remain locked.

---
