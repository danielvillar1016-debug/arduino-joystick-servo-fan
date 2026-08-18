# arduino-joystick-servo-fan
Arduino Uno project using a joystick to control an SG90 servo and toggle a 5V fan through a 2N2222A transistor.

##Demo


https://github.com/user-attachments/assets/22b54b09-392a-4bfa-9032-a74b432c9780


https://github.com/user-attachments/assets/2426313a-76fb-4bd2-ad0a-ce30d578dec9






Joystick-Controlled Rotating Fan
Project Overview

This project is a small Arduino-controlled fan system that uses a joystick to control the direction of a servo motor and to turn a DC fan on and off.

The fan is physically mounted onto a Tower Pro SG90 9g micro servo using super glue. Moving the joystick left and right rotates the servo, allowing the direction of airflow to be changed. Pressing the joystick acts as an ON/OFF switch for the fan.

The project combines analog input, servo control, transistor switching, and a DC motor into one system.

Components Used
Arduino Uno
Analog joystick module
GND
+5V
VRX
VRY
SW
Tower Pro SG90 9g micro servo
5V DC fan/motor
2N2222A NPN transistor
1 kΩ resistor
Breadboard
Jumper wires
Super glue for mounting the fan to the servo
Optional flyback diode such as 1N4001–1N4007
Wiring
Joystick
Joystick Pin	Arduino
GND	GND
+5V	5V
VRX	A0
VRY	A1
SW	D2
SG90 Servo
Servo Wire	Arduino
Brown	GND
Red	5V
Yellow	D9
Fan Control Circuit

The fan is controlled using a 2N2222A transistor instead of connecting the motor directly to an Arduino digital pin.

The control path is:

Arduino D6
   |
1 kΩ resistor
   |
2N2222A Base

The fan circuit is:

5V
 |
Fan RED
 |
Fan Motor
 |
Fan BLACK
 |
2N2222A
 |
GND

In my breadboard layout:

D6 → E20

1 kΩ resistor:
A20 → A26

2N2222A:
Middle leg → E26
Ground-side leg → E25
Fan-side leg → E27

B25 → Ground rail
B27 → Fan black wire

The breadboard ground rail is connected to Arduino GND.

Arduino Code
#include <Servo.h>

Servo myServo;

const int xPin = A0;
const int yPin = A1;
const int swPin = 2;

const int servoPin = 9;
const int fanPin = 6;

bool fanOn = false;

int buttonState = HIGH;
int lastButtonReading = HIGH;

unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

void setup() {

  Serial.begin(9600);

  pinMode(swPin, INPUT_PULLUP);
  pinMode(fanPin, OUTPUT);

  digitalWrite(fanPin, LOW);

  myServo.attach(servoPin);
  myServo.write(90);

  Serial.println("JOYSTICK + SERVO + FAN STARTED");
}

void loop() {

  // Read joystick
  int xValue = analogRead(xPin);
  int yValue = analogRead(yPin);

  // Convert joystick X position to servo angle
  int servoAngle = map(xValue, 0, 1023, 0, 180);

  servoAngle = constrain(servoAngle, 0, 180);

  myServo.write(servoAngle);

  // Read joystick button
  int reading = digitalRead(swPin);

  // Debounce the button
  if (reading != lastButtonReading) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {

    if (reading != buttonState) {

      buttonState = reading;

      if (buttonState == LOW) {

        fanOn = !fanOn;

        if (fanOn) {
          digitalWrite(fanPin, HIGH);
        } else {
          digitalWrite(fanPin, LOW);
        }
      }
    }
  }

  lastButtonReading = reading;

  // Serial Monitor debugging
  Serial.print("X: ");
  Serial.print(xValue);

  Serial.print("   Y: ");
  Serial.print(yValue);

  Serial.print("   SERVO: ");
  Serial.print(servoAngle);
  Serial.print(" degrees");

  Serial.print("   FAN: ");

  if (fanOn) {
    Serial.println("ON");
  } else {
    Serial.println("OFF");
  }

  delay(15);
}
How It Works

The Arduino continuously reads the X-axis of the joystick using analog pin A0.

The joystick produces a value from approximately:

0 → 1023

The Arduino maps this range to:

0° → 180°

for the SG90 servo.

For example:

Joystick Left   → Servo ≈ 0°
Joystick Center → Servo ≈ 90°
Joystick Right  → Servo ≈ 180°

The joystick also contains a push button.

When the joystick is clicked, the Arduino toggles digital pin D6.

D6 controls the base of the 2N2222A transistor through a 1 kΩ resistor. The transistor acts as an electronic switch for the fan.

The control sequence is therefore:

Joystick click
      ↓
Arduino D6
      ↓
2N2222A transistor
      ↓
Fan ON / OFF

Pressing the joystick once turns the fan on, and pressing it again turns the fan off.

Mechanical Design

The 5V fan was attached directly to the SG90 servo assembly using super glue.

This allows the servo to physically rotate the fan as the joystick is moved.

The result is a small directional fan where the user can:

Aim the fan using the joystick.
Turn the fan on using the joystick button.
Turn the fan off using the same button.
Current Issue

The complete system works, but the fan currently has a startup problem.

When the joystick button is pressed:

Arduino reports FAN: ON

but the fan does not always begin spinning by itself.

If the fan blade is given a small manual push, it begins spinning and continues running normally.

When the joystick is clicked again, the fan shuts off automatically as intended.

Therefore:

Fan OFF command → Works
Fan running → Works
Fan ON command → Detected
Automatic startup → Inconsistent
Manual kick-start → Fan begins running
Possible Causes

The behavior suggests that the motor is receiving enough power to continue rotating once it is moving, but may not be receiving enough starting current or torque to overcome its initial resistance.

Several factors could contribute to this.

1. Motor Startup Current

DC motors normally require significantly more current when starting than they require once they are already spinning.

The Arduino's 5V supply may not be able to comfortably provide the starting current required by the fan, especially because the SG90 servo is also powered from the Arduino.

2. Servo and Fan Sharing the 5V Supply

Both of these devices can draw relatively large amounts of current:

Arduino 5V
   ├── SG90 Servo
   └── DC Fan

When the fan starts, its voltage may temporarily drop because both motors are sharing the same supply.

3. Transistor Voltage Drop

The 2N2222A transistor introduces some voltage loss between the fan and ground.

If the fan already has limited startup torque, losing additional voltage across the transistor can make starting more difficult.

4. Mechanical Resistance

The fan was attached to the servo using super glue.

The mounting could have introduced:

Additional friction
Slight fan misalignment
Blade interference
Uneven loading
Additional mechanical resistance

Any of these could increase the torque required for the fan to begin rotating.

5. Fan Balance

Because the fan and servo are physically connected, the added weight and mounting position may also affect the fan's mechanical balance.

Planned Improvements

A major improvement would be to power the fan and servo from a dedicated regulated 5V power supply rather than relying entirely on the Arduino's 5V output.

A future power arrangement could be:

External regulated 5V supply
       |
       ├── Fan
       |
       └── Servo

Arduino GND ───── External supply GND

The Arduino would still control:

D6 → transistor → fan

D9 → servo signal

but would no longer have to provide all of the motor current.

Other possible improvements include:

Using a logic-level N-channel MOSFET instead of the 2N2222A.
Adding a flyback diode across the motor.
Improving the physical mounting of the fan.
Testing the fan separately from the servo.
Measuring fan voltage during startup.
Adding fan speed control using PWM.
Using the joystick Y-axis to control fan speed.
Designing or 3D-printing a proper fan-to-servo mounting bracket instead of using super glue.
Future Version

A future version could use both joystick axes:

Joystick X → Fan direction
Joystick Y → Fan speed
Joystick button → Fan ON/OFF

This would turn the current prototype into a more complete manually controlled directional cooling system.

Result

The project successfully demonstrates several Arduino concepts in one device:

Analog joystick input
Digital push-button input
Servo motor positioning
DC motor control
Transistor switching
Button debouncing
Breadboard circuit design
Serial Monitor debugging
Mechanical and electrical system integration

The servo successfully follows the joystick position, and the joystick button successfully controls the fan's ON/OFF state.

The main remaining engineering problem is improving the fan's startup reliability so that it can begin spinning without requiring a manual push.
