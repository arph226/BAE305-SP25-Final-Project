# BAE305-SP25-Final-Project
Group Members: Faith Cox, Abby Phillips, and Quinn Rison
Date: 4/16/2024

# Summary


# Design Description
## System Overview
- pH Sensing and Data Display (LCD)
- Visual Feedback (RGB LED)
- Motorized pH sensor movement
- User Input via Serial Monitor
- pH Recommendations

## Equipment
- Ardunino UNO Microcontroller
- Analog pH sensor
- 16x2 LDC Display
- RGB LED
- Motor Driver
- 2 Wire DC Brush Motor
- Breadboard
- Wires
- Power Supply
## Mechanical System Overview


## Circuit Wiring Overview

| **Component**              | **Arduino Pin** | **Connection Description**                   |
|---------------------------|------------------|----------------------------------------------|
| **pH Sensor**             | A0               | Analog voltage input from pH probe           |
| **LCD (16x2)**            | 12               | RS – Register Select                         |
|                           | 11               | E – Enable                                   |
|                           | 5                | D4 – Data pin                                |
|                           | 4                | D5 – Data pin                                |
|                           | 3                | D6 – Data pin                                |
|                           | 2                | D7 – Data pin                                |
|                           | 5V               | VCC – Power                                   |
|                           | GND              | GND – Ground                                  |
| **RGB LED (Common Cathode)** | A1            | RED channel (PWM capable)                    |
|                           | A2               | GREEN channel (PWM capable)                  |
|                           | A3               | BLUE channel (PWM capable)                   |
|                           | GND              | Common cathode (ground)                      |
| **TB6612FNG Motor Driver**| 9                | AIN1 – Direction control                     |
|                           | 10               | AIN2 – Direction control                     |
|                           | A4               | PWMA – Motor speed (PWM signal)              |
|                           | A5               | STBY – Standby (set HIGH to enable driver)   |
|                           | VIN or ext 12V   | VM – Motor power supply (6–12V recommended)  |
|                           | 5V               | VCC – Logic voltage                          |
|                           | GND              | GND – Shared ground with Arduino             |
| **Motor (DG01D)**         | A01, A02         | Connect to A01 and A02 on TB6612FNG          |

## Project Code
# Testing Description
## pH Sensor Reading Accuracy
## RGB LED Accuracy
## LCD Display
## Motor Control
## Serial Command Inputs and Recommendations

# Design Decision Discussion
***Include information on why we made the decisions we did for users

# Test Results Discussion
talk about swinging of pH tester

# Testing Results
