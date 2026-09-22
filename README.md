# Arduino Bluetooth 6DOF Robotic Arm

A Bluetooth-controlled **6-DOF robotic arm** built with Arduino and six servo motors.

The project allows the user to control each servo individually through Bluetooth commands. Servo positions can be saved as a sequence and later replayed automatically. The system also provides commands for resetting the arm, testing all motors, changing movement speed, and pausing or resuming a recorded sequence.

This project was developed as an Arduino-based robotic arm control system for learning and experimenting with **robotics, servo control, Bluetooth communication, and motion sequencing**.

---

# Features

* 6 servo motor control
* Bluetooth-based wireless control
* Individual control of all six servos
* Configurable servo movement limits
* Initial/home position for each servo
* Save current arm positions
* Store up to 50 positions
* Replay saved positions automatically
* Pause a running sequence
* Resume a paused sequence
* Reset saved positions
* Return arm to home position
* Adjustable movement speed
* Test all six motors
* Servo position tracking
* Bluetooth command-based control

---

# Hardware Used

| Component             | Purpose                |
| --------------------- | ---------------------- |
| Arduino Uno           | Main controller        |
| Bluetooth Module      | Wireless communication |
| Servo Motor 1         | Robotic arm movement   |
| Servo Motor 2         | Robotic arm movement   |
| Servo Motor 3         | Robotic arm movement   |
| Servo Motor 4         | Robotic arm movement   |
| Servo Motor 5         | Robotic arm movement   |
| Servo Motor 6         | Robotic arm movement   |
| External Power Supply | Powers servo motors    |
| Jumper Wires          | Connections            |
| Robotic Arm Frame     | Mechanical structure   |

> The exact Bluetooth module depends on the module used during assembly. The code communicates through serial RX/TX using `SoftwareSerial`.

---

# Software Used

The project is programmed using **Arduino C/C++**.

Required libraries:

```cpp
SoftwareSerial
Servo
```

`Servo` is used to control the six servo motors.

`SoftwareSerial` is used to create a serial communication interface for the Bluetooth module.

---

# Pin Connections

## Servo Motors

The project uses six servo motors.

| Servo   | Arduino Pin | Purpose      |
| ------- | ----------: | ------------ |
| Servo 1 |          D2 | Joint/axis 1 |
| Servo 2 |          D3 | Joint/axis 2 |
| Servo 3 |          D4 | Joint/axis 3 |
| Servo 4 |          D5 | Joint/axis 4 |
| Servo 5 |          D6 | Joint/axis 5 |
| Servo 6 |          D7 | Joint/axis 6 |

The code attaches the servos as follows:

```cpp
servo01.attach(2);
servo02.attach(3);
servo03.attach(4);
servo04.attach(5);
servo05.attach(6);
servo06.attach(7);
```

### Typical Servo Connections

Each servo normally has three wires:

| Servo Wire | Connection          |
| ---------- | ------------------- |
| Signal     | Arduino digital pin |
| VCC        | External 5V supply  |
| GND        | GND                 |

### Important Power Note

Six servo motors should **not normally be powered directly from the Arduino Uno's 5V pin**.

Use a suitable external 5V power supply for the servos and connect:

```text
External Power GND
        |
        +---- Arduino GND
```

The Arduino and servo power supply need a **common ground** so that the control signals work correctly.

---

# Bluetooth Connections

The code contains:

```cpp
SoftwareSerial Bluetooth(0, 1);
```

This means:

```text
Bluetooth RX → Arduino D0
Bluetooth TX → Arduino D1
```

| Bluetooth | Arduino Uno     |
| --------- | --------------- |
| RX        | D0              |
| TX        | D1              |
| VCC       | Module-specific |
| GND       | GND             |

## Important Note About Pins 0 and 1

Pins **D0 and D1** are the Arduino Uno's hardware serial RX/TX pins.

Your current project deliberately uses these pins for Bluetooth:

```cpp
SoftwareSerial Bluetooth(0, 1);
```

However, this is **not recommended for a normal Arduino Uno setup**, because these pins are also used when communicating with the computer through USB.

Possible problems include:

* Upload problems
* Serial Monitor conflicts
* Bluetooth communication conflicts
* Unexpected data during programming

A future version could move Bluetooth to other digital pins, for example:

```cpp
SoftwareSerial Bluetooth(8, 9);
```

provided those pins are available and the wiring is changed accordingly.

---

# Complete Pin Summary

| Component | Signal Pin | Arduino Uno |
| --------- | ---------- | ----------: |
| Servo 1   | Signal     |          D2 |
| Servo 2   | Signal     |          D3 |
| Servo 3   | Signal     |          D4 |
| Servo 4   | Signal     |          D5 |
| Servo 5   | Signal     |          D6 |
| Servo 6   | Signal     |          D7 |
| Bluetooth | RX         |          D0 |
| Bluetooth | TX         |          D1 |

---

# Servo Initial Positions

When the Arduino starts, every servo is moved to its configured initial position.

The initial positions are:

| Servo   | Initial Position |
| ------- | ---------------: |
| Servo 1 |              60° |
| Servo 2 |              60° |
| Servo 3 |              40° |
| Servo 4 |             200° |
| Servo 5 |             170° |
| Servo 6 |             100° |

These values are defined in the code:

```cpp
const int servo01Initial = 60;
const int servo02Initial = 60;
const int servo03Initial = 40;
const int servo04Initial = 200;
const int servo05Initial = 170;
const int servo06Initial = 100;
```

The initial position acts as the **home position** of the robotic arm.

---

# Servo Movement Limits

Each servo has its own configured movement range.

| Servo   | Minimum | Maximum |
| ------- | ------: | ------: |
| Servo 1 |     55° |      0° |
| Servo 2 |     40° |    120° |
| Servo 3 |     30° |    130° |
| Servo 4 |    100° |    290° |
| Servo 5 |     80° |    170° |
| Servo 6 |      0° |    290° |

These values are configured in the code.

> The project uses values above the normal 0–180° range for some servos. Whether these values are physically valid depends on the servo library, servo hardware, and mechanical setup. Test carefully before using the full range.

---

# How the Project Works

The basic system works like this:

```text
Bluetooth Command
       ↓
Arduino Receives Command
       ↓
Identify Command
       ↓
Identify Servo / Function
       ↓
Execute Action
       ↓
Update Servo Position
```

For example:

```text
s690
```

means:

```text
Servo 1 → 90°
```

The Arduino receives the command through Bluetooth, identifies `s6` as Servo 1, extracts `90`, and moves the servo.

---

# Bluetooth Command System

The project uses text-based commands.

Each command is sent as a line ending with:

```text
\n
```

The Arduino reads the command using:

```cpp
Bluetooth.readStringUntil('\n');
```

The received command is then cleaned using:

```cpp
dataIn.trim();
```

---

# Servo Control Commands

The six servo commands are:

| Command Prefix | Controls |
| -------------- | -------- |
| `s6`           | Servo 1  |
| `s5`           | Servo 2  |
| `s4`           | Servo 3  |
| `s3`           | Servo 4  |
| `s2`           | Servo 5  |
| `s1`           | Servo 6  |

The command format is:

```text
prefix + angle
```

---

# Servo 1

Servo 1 uses:

```text
s6
```

Example:

```text
s690
```

means:

```text
Servo 1 → 90°
```

Another example:

```text
s660
```

means:

```text
Servo 1 → 60°
```

---

# Servo 2

Servo 2 uses:

```text
s5
```

Example:

```text
s580
```

means:

```text
Servo 2 → 80°
```

---

# Servo 3

Servo 3 uses:

```text
s4
```

Example:

```text
s4100
```

means:

```text
Servo 3 → 100°
```

---

# Servo 4

Servo 4 uses:

```text
s3
```

Example:

```text
s3150
```

means:

```text
Servo 4 → 150°
```

---

# Servo 5

Servo 5 uses:

```text
s2
```

Example:

```text
s2120
```

means:

```text
Servo 5 → 120°
```

---

# Servo 6

Servo 6 uses:

```text
s1
```

Example:

```text
s190
```

means:

```text
Servo 6 → 90°
```

---

# Why Are the Servo Commands Reversed?

The code maps:

```text
s6 → Servo 1
s5 → Servo 2
s4 → Servo 3
s3 → Servo 4
s2 → Servo 5
s1 → Servo 6
```

This appears to be based on the command format expected by your Bluetooth controller/application.

It is not a problem as long as your controlling application uses the same command mapping.

---

# SAVE Command

The `SAVE` command stores the current position of all six servos.

Send:

```text
SAVE
```

The system stores:

```text
Servo 1 position
Servo 2 position
Servo 3 position
Servo 4 position
Servo 5 position
Servo 6 position
```

as one saved position.

---

# Position Storage

The project creates six arrays:

```cpp
int servo01SP[50];
int servo02SP[50];
int servo03SP[50];

int servo04SP[50];
int servo05SP[50];
int servo06SP[50];
```

The maximum number of saved positions is:

```text
50
```

Each index represents one complete robotic-arm position.

For example:

```text
Position 0
  Servo 1 = 60°
  Servo 2 = 80°
  Servo 3 = 100°
  Servo 4 = 150°
  Servo 5 = 120°
  Servo 6 = 90°

Position 1
  Servo 1 = 70°
  Servo 2 = 90°
  Servo 3 = 110°
  Servo 4 = 160°
  Servo 5 = 130°
  Servo 6 = 100°
```

---

# RUN Command

The `RUN` command plays all saved positions.

Send:

```text
RUN
```

The Arduino starts from the first saved position and moves through the sequence.

Example:

```text
SAVE
SAVE
SAVE
RUN
```

The robotic arm will replay the three saved positions.

After all positions are completed, the arm returns to its home position.

---

# RUN Process

The sequence works like:

```text
Saved Position 1
       ↓
Saved Position 2
       ↓
Saved Position 3
       ↓
Saved Position 4
       ↓
...
       ↓
Last Saved Position
       ↓
Return Home
```

The number of positions depends on how many times `SAVE` was used.

---

# PAUSE Command

While a saved sequence is running, the project can receive:

```text
PAUSE
```

When the command is detected, the system enters a waiting state.

The robotic arm stops advancing through the sequence until another command is received.

---

# Resume After PAUSE

To continue the sequence, send:

```text
RUN
```

The program exits the waiting state and continues the movement sequence.

The basic flow is:

```text
RUN
 ↓
Play Positions
 ↓
PAUSE
 ↓
Wait
 ↓
RUN
 ↓
Continue
```

---

# RESET Command

The `RESET` command clears all saved positions and returns the arm to its initial positions.

Send:

```text
RESET
```

The system:

1. Clears all saved position arrays.
2. Resets the saved-position index.
3. Moves Servo 1 to its initial position.
4. Moves Servo 2 to its initial position.
5. Moves Servo 3 to its initial position.
6. Moves Servo 4 to its initial position.
7. Moves Servo 5 to its initial position.
8. Moves Servo 6 to its initial position.

---

# RESET Example

Before reset:

```text
Saved Positions:
1
2
3
4
5
```

Send:

```text
RESET
```

After reset:

```text
Saved Positions:
0
```

and the arm returns to:

```text
Servo 1 → 60°
Servo 2 → 60°
Servo 3 → 40°
Servo 4 → 200°
Servo 5 → 170°
Servo 6 → 100°
```

---

# TEST Command

The `TEST` command tests all six servo motors.

Send:

```text
TEST
```

The Arduino moves each servo toward its configured maximum and minimum positions.

The sequence is:

```text
Servo 1
  ↓
Maximum
  ↓
Minimum

Servo 2
  ↓
Maximum
  ↓
Minimum

Servo 3
  ↓
Maximum
  ↓
Minimum

...
```

This can be useful when:

* Testing new hardware
* Checking servo wiring
* Checking servo movement
* Finding mechanical problems
* Testing the arm before running a saved sequence

---

# Speed Control

The project includes a speed-delay variable:

```cpp
int speedDelay = 5;
```

The Bluetooth command for changing it is:

```text
ss
```

For example:

```text
ss10
```

sets:

```text
speedDelay = 10
```

The value is restricted between:

```text
1 → 50
```

using:

```cpp
constrain(dataIn.substring(2).toInt(), 1, 50);
```

---

# Important Note About Speed Control

The current version stores the value in `speedDelay`, but the actual servo movement function:

```cpp
moveServoFast()
```

uses a fixed:

```cpp
delay(100);
```

Therefore, changing `speedDelay` currently does **not significantly change the movement delay** in `moveServoFast()`.

The variable is prepared for speed control, but the current code does not use it to control the servo delay.

A future version can modify:

```cpp
delay(100);
```

to use:

```cpp
delay(speedDelay);
```

or implement a smoother step-by-step servo movement system.

---

# Complete Command List

| Command   | Function                        |
| --------- | ------------------------------- |
| `s6ANGLE` | Move Servo 1                    |
| `s5ANGLE` | Move Servo 2                    |
| `s4ANGLE` | Move Servo 3                    |
| `s3ANGLE` | Move Servo 4                    |
| `s2ANGLE` | Move Servo 5                    |
| `s1ANGLE` | Move Servo 6                    |
| `SAVE`    | Save current arm position       |
| `RUN`     | Play saved positions            |
| `PAUSE`   | Pause running sequence          |
| `RESET`   | Clear positions and return home |
| `TEST`    | Test all six motors             |
| `ssVALUE` | Set speed-delay value           |

---

# Example Control Session

A typical robotic-arm operation could look like:

```text
s690
```

Servo 1 moves to 90°.

Then:

```text
s580
```

Servo 2 moves to 80°.

Then:

```text
s4100
```

Servo 3 moves to 100°.

Then:

```text
s3150
```

Servo 4 moves to 150°.

Then:

```text
s2120
```

Servo 5 moves to 120°.

Then:

```text
s190
```

Servo 6 moves to 90°.

Now the complete arm position is ready.

Send:

```text
SAVE
```

The position is stored.

Move the arm to another position and send:

```text
SAVE
```

again.

After creating several positions:

```text
RUN
```

plays the saved sequence.

---

# Complete Project Workflow

```text
                  START
                    |
                    v
             Initialize Arduino
                    |
                    v
            Initialize Bluetooth
                    |
                    v
             Initialize Servos
                    |
                    v
            Move to Home Position
                    |
                    v
             Wait for Bluetooth
                    |
          +---------+---------+
          |         |         |
       Servo      SAVE      RUN
       Command     |         |
          |        v          v
          v     Store      Play Saved
       Move Servo Position   Positions
          |        |          |
          +--------+----------+
                   |
                   v
              Wait Again
```

Other commands:

```text
TEST
 ↓
Test All Motors
 ↓
Return to Normal

RESET
 ↓
Clear Saved Positions
 ↓
Move to Home
 ↓
Return to Normal

PAUSE
 ↓
Pause RUN
 ↓
Wait for RUN
```

---

# Project Architecture

The project can be understood as four main parts.

### 1. Bluetooth Communication

Receives commands from the Bluetooth controller.

```cpp
SoftwareSerial Bluetooth(0, 1);
```

### 2. Servo Control

Controls the six robotic-arm servos.

```cpp
Servo servo01, servo02, servo03;
Servo servo04, servo05, servo06;
```

### 3. Position Recording

Stores the positions of all six servos.

```text
Maximum saved positions = 50
```

### 4. Sequence Playback

Reads the stored positions and moves the robotic arm through them using:

```text
RUN
```

---

# Installation

## Step 1 — Install Arduino IDE

Install the Arduino IDE on your computer.

## Step 2 — Install Required Libraries

Open:

```text
Arduino IDE
→ Library Manager
```

Install:

```text
Servo
SoftwareSerial
```

`Servo` is used for controlling the motors.

`SoftwareSerial` is used for Bluetooth communication.

## Step 3 — Connect the Hardware

Connect the six servo signal wires and Bluetooth module according to the pin tables.

## Step 4 — Power the Servos

Use an appropriate external power supply for the servo motors.

Do not rely on the Arduino Uno's 5V regulator to power six servos simultaneously.

Make sure the external servo supply and Arduino share GND.

## Step 5 — Open the Code

Open:

```text
Arduino-Bluetooth-6DOF-Robotic-Arm.ino
```

## Step 6 — Select Board

Select:

```text
Arduino Uno
```

## Step 7 — Upload

Upload the program to the Arduino.

> Because the current code uses D0 and D1 for Bluetooth, disconnect the Bluetooth module from those pins if necessary while uploading the program.

## Step 8 — Connect Bluetooth

After uploading, reconnect the Bluetooth module.

## Step 9 — Send Commands

Use a Bluetooth controller application capable of sending text commands ending with a newline.

---

# Project Structure

A recommended GitHub repository structure is:

```text
Arduino-Bluetooth-6DOF-Robotic-Arm/
│
├── Arduino-Bluetooth-6DOF-Robotic-Arm.ino
├── README.md
│
├── circuit/
│   └── circuit-diagram.png
│
├── images/
│   ├── robotic-arm.jpg
│   ├── circuit.jpg
│   └── project.jpg
│
└── documentation/
    └── project-report.pdf
```

---

# Important Hardware Notes

### Servo Power

Six servo motors can draw significant current, especially when moving simultaneously.

Use a suitable external power supply.

The basic power structure should be:

```text
External 5V Supply
       |
       +---- Servo 1
       +---- Servo 2
       +---- Servo 3
       +---- Servo 4
       +---- Servo 5
       +---- Servo 6
       |
      GND
       |
       +---- Arduino GND
```

### Common Ground

The Arduino and servo power supply should have a common ground.

Without a common ground, servo control signals may not work reliably.

---

# Important Code Notes

## 1. Bluetooth Uses D0 and D1

Current code:

```cpp
SoftwareSerial Bluetooth(0, 1);
```

These pins are also used by the Arduino Uno's USB serial connection.

For a future version, consider using other digital pins.

---

## 2. Servo Angles

Some configured servo limits are greater than 180°:

```text
Servo 4 → maximum 290°
Servo 6 → maximum 290°
```

Standard hobby servos normally operate around 0–180°.

If your servo hardware does not support the requested range, the actual movement may be limited or behave differently.

Test the mechanical range carefully to avoid damaging the servo or robotic-arm structure.

---

## 3. Speed Variable

The code includes:

```cpp
speedDelay
```

but the current movement function uses a fixed:

```cpp
delay(100);
```

Therefore, speed control is not fully implemented yet.

This is a good area for a future improvement.

---

## 4. Position Storage Is Temporary

Saved positions are stored in normal Arduino RAM arrays.

They are **not stored permanently**.

If the Arduino loses power or resets, the saved positions are lost.

A future version could use:

* EEPROM
* SD card
* External memory

to permanently store robotic-arm sequences.

---

# Future Improvements

Possible improvements for this project include:

* Move Bluetooth communication away from pins 0 and 1
* Add a Bluetooth HC-05/HC-06 configuration section
* Add an OLED/LCD display
* Add a graphical mobile controller
* Implement true variable servo speed
* Use smooth servo interpolation
* Save positions permanently using EEPROM
* Save multiple named sequences
* Add sequence selection
* Add forward/reverse playback
* Add emergency stop
* Add joystick control
* Add potentiometer/manual control
* Add object-grabbing automation
* Add sensors for object detection
* Add computer vision
* Add inverse kinematics
* Add autonomous movement
* Add Wi-Fi control
* Add a web-based robotic-arm controller

---

# Learning Concepts Demonstrated

This project demonstrates several important embedded-systems and robotics concepts:

* Arduino programming
* C/C++ programming
* Servo motor control
* Bluetooth communication
* Serial communication
* Digital output
* Command parsing
* Arrays
* Functions
* References
* Position tracking
* Motion sequencing
* State-based control
* Hardware/software integration
* Robotics movement control

---

# Author

**Aliyan Hussain**

Arduino / Robotics Project

---

# License

This project is created for educational and learning purposes.
