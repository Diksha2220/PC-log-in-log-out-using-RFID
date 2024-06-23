<div align="center">
    <img src="https://github.com/Diksha2220/Media/blob/main/pc%20lock.gif" alt="PC Login/Logout using RFID " width="500"/>
</div>

### Project Idea: 🔐💻 PC Login/Logout using RFID 

#### Objective
🔧 Develop a system where an RFID reader connected to an Arduino Uno can log a user into and out of a PC.

#### Components Needed
- 🛠️ Arduino Uno
- 🌟 RFID Reader (e.g., RC522)
- 📛 RFID Tags
- 💡 LEDs (optional, for indication)
- ⚡ Resistors (220 ohms for LEDs)
- 🍞 Breadboard
- 🔗 Jumper wires
- 💻 PC with USB connection to Arduino

#### Circuit Setup

**RFID Reader Connections:**
- SDA -> Arduino pin 10
- SCK -> Arduino pin 13
- MOSI -> Arduino pin 11
- MISO -> Arduino pin 12
- IRQ -> Not connected
- GND -> Ground
- RST -> Arduino pin 9
- 3.3V -> 3.3V

**LED Connections (Optional):**
- Connect the anode of the green LED to digital pin 5 through a 220-ohm resistor.
- Connect the cathode of the green LED to ground.
- Connect the anode of the red LED to digital pin 6 through a 220-ohm resistor.
- Connect the cathode of the red LED to ground.

### Arduino Sketch (Code)

#### RFID Reader Code (Arduino)

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9
MFRC522 rfid(SS_PIN, RST_PIN);

const int green_led_pin = 5;
const int red_led_pin = 6;

void setup() {
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();
  pinMode(green_led_pin, OUTPUT);
  pinMode(red_led_pin, OUTPUT);
  digitalWrite(green_led_pin, LOW);
  digitalWrite(red_led_pin, LOW);
  Serial.println("Place your RFID card/tag near the reader...");
}

void loop() {
  if (!rfid.PICC_IsNewCardPresent())
    return;
  
  if (!rfid.PICC_ReadCardSerial())
    return;
    
  String content = "";
  for (byte i = 0; i < rfid.uid.size; i++) {
    content.concat(String(rfid.uid.uidByte[i] < 0x10 ? " 0" : " "));
    content.concat(String(rfid.uid.uidByte[i], HEX));
  }
  content.toUpperCase();

  // Compare the UID of the scanned card to predefined UIDs
  if (content.substring(1) == "04 A3 58 4B 7C") { // Change this to your card's UID
    Serial.println("Access Granted");
    digitalWrite(green_led_pin, HIGH);  // Green LED on
    digitalWrite(red_led_pin, LOW);     // Red LED off
    // Send signal to PC for login
    Serial.println("LOGIN");
    delay(1000);
  } else {
    Serial.println("Access Denied");
    digitalWrite(red_led_pin, HIGH);  // Red LED on
    digitalWrite(green_led_pin, LOW); // Green LED off
    // Send signal to PC for logout
    Serial.println("LOGOUT");
    delay(1000);
  }
  rfid.PICC_HaltA();
}
```

### PC Integration

To log in and log out of the PC, you can use a serial communication script on the PC to interpret the messages from the Arduino. For this, you can use Python with the `pyserial` library.

#### Python Script for PC

1. Install `pyserial`:
```sh
pip install pyserial
```

2. Create a Python script to read serial data:
```python
import serial
import time
import os

# Change the COM port to the one your Arduino is connected to
ser = serial.Serial('COM3', 9600, timeout=1)

def login():
    # Replace this with the actual command to login
    os.system("rundll32.exe user32.dll,LockWorkStation")  # Example: Locks the workstation

def logout():
    # Replace this with the actual command to logout
    os.system("shutdown -l")  # Example: Logs out the user

while True:
    try:
        if ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').rstrip()
            print(line)
            if "LOGIN" in line:
                login()
            elif "LOGOUT" in line:
                logout()
        time.sleep(1)
    except KeyboardInterrupt:
        print("Exiting program.")
        break
    except Exception as e:
        print(f"Error: {e}")

ser.close()
```

### How It Works
1. **RFID Reader**: The RFID reader reads the UID of the RFID tag.
2. **Arduino**: The Arduino compares the UID to predefined UIDs and sends a login or logout signal to the PC via serial communication.
3. **PC Script**: The Python script running on the PC reads the serial data from the Arduino and executes the appropriate login or logout command.

### Important Considerations
- Make sure the COM port in the Python script matches the port your Arduino is connected to.
- Customize the login and logout commands in the Python script to suit your needs.
- Ensure your Arduino and PC have proper permissions to perform login and logout operations.

This setup allows for a simple and effective way to control PC login/logout using RFID tags and Arduino.


<div align="center">
    <img src="https://github.com/Diksha2220/Media/blob/main/RFID%20scan.gif" alt="PC Login/Logout using RFID " width="500"/>
</div>
