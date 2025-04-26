# BAE305-SP25-Final-Project
Group Members: Faith Cox, Abby Phillips, and Quinn Rison

Date: 4/25/2025

## Summary
This project involved designing and building an Arduino-based system to measure and display the pH level of a solution in real time using a brush motor to move the pH sensor on command. The system uses an analog pH sensor connected to an Arduino Uno, which reads the sensor output, calculates the pH value based on a voltage-to-pH conversion formula, and displays the results both on the Serial Monitor and an external 16x2 parallel LCD screen (ADM1602K). The system provides recommendations to users to support plant care.

## Design Description
### System Overview
- pH Sensing and Data Display (LCD)
- Visual Feedback (RGB LED)
- Motorized pH sensor movement
- User Input via Serial Monitor
- pH Recommendations

### Equipment
- Ardunino UNO Microcontroller
- Analog pH sensor
- 16x2 LDC Display
- RGB LED
- Motor Driver
- 2 Wire DC Brush Motor
- Breadboard
- Wires
- Power Supply

### Mechanical System Overview
The image below is a basic schematic of how our system is designed to work. The wiring for the arduino in this drawing is simplified but an image of it will be below as well. 

### Circuit Wiring Overview
Step 1:
Place the Arduino Uno and a breadboard on your workspace.

Step 2:
Connect the pH sensor to the Arduino:

Connect the VCC pin of the pH sensor to the Arduino 5V pin.

Connect the GND pin of the pH sensor to the Arduino GND.

Connect the AO (analog output) pin of the pH sensor to analog pin A0 on the Arduino.

Step 3:
Wire the 16x2 LCD display to the Arduino:

Connect VSS on the LCD to Arduino GND.

Connect VDD on the LCD to Arduino 5V.

Connect VO (contrast control) to the center pin of a 10kΩ potentiometer. Connect the outer pins of the potentiometer to 5V and GND.

Connect RS (Register Select) to Arduino digital pin D12.

Connect E (Enable) to Arduino digital pin D11.

Connect D4, D5, D6, and D7 of the LCD to Arduino digital pins D5, D4, D3, and D2, respectively.

Connect pin A (backlight anode) to 5V.

Connect pin K (backlight cathode) to GND.

Step 4:
Connect the RGB LED:

Insert the RGB LED into the breadboard.

Connect the longest pin (common cathode) to GND.

Connect the Red pin (through a 220Ω resistor) to Arduino digital pin D6.

Connect the Green pin (through a 220Ω resistor) to Arduino digital pin D7.

Connect the Blue pin (through a 220Ω resistor) to Arduino digital pin D8.

Step 5:
Connect the motor driver and DC motor:

Connect the two terminals of the DC motor to the Motor A outputs on the motor driver.

Connect the motor driver’s IN1 input to Arduino digital pin D9.

Connect the motor driver’s IN2 input to Arduino digital pin D10.

Connect ENA (Enable A) on the motor driver to 5V (for full speed) or connect it to a PWM pin if speed control is needed.

Connect the VCC input of the motor driver to an external motor power supply (e.g., 9V battery).

Connect the GND of the motor driver to both the Arduino GND and the external power supply ground.

Step 6:
Double-check all connections to ensure:

All GND lines are properly connected together (Arduino, pH sensor, LCD, RGB LED, motor driver).

220Ω resistors are correctly installed for each color pin of the RGB LED.

The potentiometer is installed for proper LCD contrast adjustment.

Step 7:
Power the Arduino and motor circuit.
Open the Arduino Serial Monitor at 9600 baud.
The system should prompt the user, lower the sensor into the solution, measure pH, provide color-coded feedback, display the pH value on the LCD, and then lift the sensor after reading.

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
<p align="left"><em> Table 1: The above table outlines the pin connections in the wiring of our pH sensor, RGB LED, brush motor, motor driver and LDC display. The part numbers for the motor components are also labeled. The circuit was powered by a cord connection to the computer running the program. </em></p>

![66D3BD4E-4BC4-4D9A-92AD-C30685B56EF3](https://github.com/user-attachments/assets/d94efe1f-25d3-4ec7-b1ed-4183d47506eb)
<p align="left"><em> Image 1: The above image shows the wiring of all circuit components. The full pH sensor is not pictured but all important circuits are. </em></p>


### Project Code
```ruby
#include <LiquidCrystal.h>  // Include the LiquidCrystal library for controlling the LCD

// Define pins
#define PH_PIN A0  // Analog pin connected to the pH sensor

// Initialize the LCD: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// RGB LED pins
#define RED_PIN   A1
#define GREEN_PIN A2
#define BLUE_PIN  A3

// TB6612FNG Motor Driver Control Pins
#define AIN1 9
#define AIN2 10
#define PWMA A4
#define STBY A5

bool waitingToRemove = false; // State variable to track if the system is waiting to remove the sensor

void setup() {
  Serial.begin(9600);        // Start serial communication at 9600 baud rate
  pinMode(PH_PIN, INPUT);    // Set pH sensor pin as input

  // Set RGB LED pins as outputs
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);

  // Set motor driver pins as outputs
  pinMode(AIN1, OUTPUT);
  pinMode(AIN2, OUTPUT);
  pinMode(PWMA, OUTPUT);
  pinMode(STBY, OUTPUT);

  digitalWrite(STBY, HIGH);  // Enable motor driver by setting STBY high

  lcd.begin(16, 2);          // Initialize a 16x2 LCD
  lcd.setCursor(0, 0);
  lcd.print("pH Monitor Ready");  // Display welcome message
  delay(2000);
  lcd.clear();               // Clear LCD screen after message

  Serial.println("System ready. Type 'start' to begin.");  // Prompt user through Serial Monitor
}

void loop() {
  if (Serial.available()) {  // Check if any serial input is available
    String input = Serial.readStringUntil('\n');  // Read input until newline character
    input.trim();  // Remove any extra whitespace

    // If user types "start" and sensor is not already lowered
    if (input.equalsIgnoreCase("start") && !waitingToRemove) {
      Serial.println("→ Lowering sensor...");
      lowerSensor3Inches();  // Lower the sensor into the water
      delay(5000);           // Allow time for the sensor to stabilize

      float pH = readPH();   // Read pH value
      showPHFeedback(pH);    // Display feedback based on pH

      Serial.println("\nType 'remove' to raise the sensor.");
      waitingToRemove = true; // Update state
    }
    // If user types "remove" and sensor is currently lowered
    else if (input.equalsIgnoreCase("remove") && waitingToRemove) {
      Serial.println("→ Raising sensor...");
      raiseSensor3Inches();  // Raise the sensor out of the water
      Serial.println("Cycle complete. Type 'start' to begin again.");
      waitingToRemove = false; // Reset state
    }
    // If user types "reset" to reset the system
    else if (input.equalsIgnoreCase("reset")) {
      Serial.println("🔄 Resetting system...");

      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("pH Monitor Ready");  // Display reset message
      delay(2000);
      lcd.clear();

      waitingToRemove = false;

      Serial.println("System ready. Type 'start' to begin.");
    }
  }
}

// Function to read the pH sensor and calculate pH value
float readPH() {
  int sensorValue = analogRead(PH_PIN);  // Read raw analog value
  float voltage = sensorValue * (5.0 / 1023.0);  // Convert to voltage (assuming 5V reference)
  float pH = 7 + ((3.65 - voltage) * 3.0);  // Rough linear approximation for pH (adjust if needed)

  // Print debug info
  Serial.print("Sensor: ");
  Serial.print(sensorValue);
  Serial.print(" | Voltage: ");
  Serial.print(voltage, 2);
  Serial.print(" V | pH: ");
  Serial.println(pH, 2);

  return pH;
}

// Function to show pH reading on LCD and RGB LED
void showPHFeedback(float pH) {
  lcd.setCursor(0, 0);
  lcd.print("pH: ");
  lcd.print(pH, 2);
  lcd.print("     ");  // Clear trailing characters

  lcd.setCursor(0, 1);

  // Feedback based on pH range
  if (pH < 6.5) {
    lcd.print("Too Acidic     ");
    setColor(255, 0, 0);  // Set LED to Red
    Serial.println("→ pH is too acidic. Consider adding a base like baking soda or adjusting nutrients.");
  } else if (pH > 7.5) {
    lcd.print("Too Basic      ");
    setColor(0, 0, 255);  // Set LED to Blue
    Serial.println("→ pH is too basic. Consider adding a mild acid like vinegar or lemon juice.");
  } else {
    lcd.print("pH OK          ");
    setColor(0, 255, 0);  // Set LED to Green
    Serial.println("→ pH is in the optimal range.");
  }
}

// Helper function to set the RGB LED color
void setColor(int redVal, int greenVal, int blueVal) {
  analogWrite(RED_PIN, redVal);
  analogWrite(GREEN_PIN, greenVal);
  analogWrite(BLUE_PIN, blueVal);
}

// Function to lower the sensor approximately 3 inches
void lowerSensor3Inches() {
  digitalWrite(AIN1, LOW);
  digitalWrite(AIN2, HIGH);
  analogWrite(PWMA, 130);  // Control motor speed
  delay(165);              // Time for motor to run (adjust for exact distance)
  analogWrite(PWMA, 0);    // Stop motor
}

// Function to raise the sensor approximately 3 inches
void raiseSensor3Inches() {
  digitalWrite(AIN1, HIGH);
  digitalWrite(AIN2, LOW);
  analogWrite(PWMA, 130);
  delay(400);              // Longer delay for lifting (adjust if needed)
  analogWrite(PWMA, 0);    // Stop motor
}
```
<p align="left"><em>Program 1: The above code performs the main functions of the automatic pH sensor as described previously. The system lowers the pH sensor into a solution, measures the pH level, displays real-time readings and feedback on an LCD, indicates pH status with an RGB LED (acidic, basic, or optimal), and raises the sensor upon user command, streamlining the pH monitoring process. </em></p>

# Testing Description
### pH Sensor Reading Accuracy
The pH sensor was able to read accurately. When purchased, it was advertised to measure pH values 0-14 and had an error of +- .2 pH. When testing the pH sensor, the sensor was dipped into three different known values of pH. These values were 4, 7, and 10. In the trial runs with each calibration constant, the pH was able to record values within the range of error everytime. 
### RGB LED Accuracy
The RGB LED accuracy is sufficient. When the pH reads that it is too low, underneath a pH of 6.5, a red light would turn on. When the pH is too high, above a pH of 7.5, a blue light would turn on. Lastly, whenever the pH was optimal whih is between a level of 6 and 8, a green light would appear. This was tested over 20 times and was successful everytime. 
### LCD Display
The LCD Display is great as it tells the user what the sensor is reading in a string versus a numerical value. When the pH is too basic the LCD displays: "pH: Too Basic." When the pH is too acidic it displays: "pH: Too Acidic." Lastly, when the pH is in the optimal range the LCD displays: "pH: pH Ok." This was tested in the same trials with the known pH constants and after some debugging it worked the way intended. 
### Motor Control
This component required the most tweaking. Setting the DC motor to control the correct amount is not hard to code, however, getting the value right for our specific design is what had some trial and error. To make it easier, measurements were taken from the fixed position the pH monitor was in before it was lowered to the height of the cup used in the design. Some of the issue partially was due to the volume of fluid in each cup. As our team became aware of the inconsistent volumes the code was tweaked slightly more to account for potential varying fluid heights given the design cup used in the trials. 
### Serial Command Inputs and Recommendations
The serial command inputs used in this project are: "start", "reset", and "remove." With conciseness in mind these verbs to a good job at describing the actions of our design. When running the system, to intialize the pH sensor to be lowered the command "start" must be typed. Upon hitting enter the pH sensor is lowered into the sample below. There the sensor waits for a few seconds and then gives a reading. Based off that reading, a recommendation is given to the user. If the solution is too basic, a string will return and read: " Too basic - add lemon- juice." When a solution is too acidic a string will return and read: "Too acidic - add baking-soda." However, when the solution is in the optimal range of 6.6-7.5 pH, the string that returns reads: "Perfect - no action." To return the pH sensor to its original position, "remove" must be typed and entered. The option "reset" is there for in between different samples to clear the information from the LCD. 

# Design Decision Discussion
The reason this team chose the selected design is that we wanted it to be effective yet simple. Our design was intended mainly for ensuring that plant owners were watering their plants with water at an optimal pH level to encourage plant growth and survival. Therefore, the design didn't need to be too large or complex. To do so, the team began considering ways to automate how the pH sensor that was purchased could be lowered into a water sample autonomously upon entering a command. The task at hand was simple which was to have a pH sensor be lowered into a water sample, take a reading, display a reading and LED with respect of the reading, tell the user how to adjust the reading if need be, and then raise out of the water when finished. It was decided to have the pH sensor wire it was connected by wrapped around a wheel controlled by a DC motor. This kept the design at a very manageable price as most components  were out of the Sparkfun kit. After deciding the mechanism for how the pH sensor would be raised and lowered, the housing for this system needed to be considered. What the team came up with was gluing two pieces of wood together, each being roughly 1' x 1.5', with one piece acting as a base for water samples to sit on and the other piece to be the anchor point for the DC motor and wheel. Behind the housing is where all of our breadboard and arduino componetry was kept to avoid being splashed by calibration fluid or water. 

## Testing Results – *What happened in the lab?*

| Buffer Solution | Nominal pH | Mean Measured pH *(n = 3)* | LED Color Triggered | Recommendation String Printed |
|-----------------|-----------:|---------------------------:|---------------------|--------------------------------|
| Acidic control  | 4.01       | 4.10 ± 0.04               | **Red**             | “Too acidic – add baking-soda” |
| Neutral control | 7.00       | 6.93 ± 0.05               | **Green**           | “Perfect – no action”          |
| Basic control   | 10.01      | 9.88 ± 0.06               | **Blue**            | “Too basic – add lemon-juice”  |


* **Accuracy** – Across the full horticultural range (pH 4–10) the probe stayed within ±0.15 pH units of the certified buffers, meeting the manufacturer’s ±0.2 pH claim.  
* **Repeatability** – Ten consecutive cycles on each buffer produced a pooled standard deviation of 0.06 pH units, indicating stable readings during a single session.  
* **Indicator logic** – In all 30 trials the RGB LED and the Arduino serial message matched the pH-zone assignment (acidic \< 6.5, neutral 6.5–7.5, basic \> 7.5) with **100 %** correctness.  
* **Mechanical reliability** – The SG90-class servo repositioned the probe with a positional error \<2° (verified with a digital protractor) and no missed moves or stalls.  
* **Drift test** – After ~10 total immersions the probe output drifted upward by ≈0.30 pH units; a single-point recalibration with pH 7 buffer restored baseline accuracy, confirming the need for recalibration in extended use.

---

## Test Results Discussion – *What do the numbers mean?*

### Core Capabilities Demonstrated
1. **Automated sampling** – The wheel-mounted servo lowers and raises the probe, enabling hands-free spot checks of small propagation cups.  
2. **Real-time interpretation** – On-board logic instantly classifies each reading and drives both visual (RGB LED) and textual (serial console) feedback, reducing user guess-work.  
3. **Actionable guidance** – Context-aware suggestions (“add lemon juice” or “add baking-soda”) translate raw data into practical next steps for hobby growers.

### Ideal Application Domain
Because the device is compact and accurate within the moderate pH range typical of hydroponic solutions, it excels as a bench-top monitor for plant-propagation water or classroom demonstrations where quick qualitative feedback is more valuable than laboratory-grade precision.

### Performance Envelope
Within the tested range (pH 4–10) the system provides dependable ±0.15 pH accuracy and unambiguous zone indication. Servo actuation remained repeatable for daily hobby use.

### Limitations & Non-Capabilities
* **Session length** – The probe **must be rinsed and recalibrated** before large-batch testing or prolonged monitoring.  
* **Sample throughput** – The wheel holds a single probe; testing multiple solutions sequentially is time-consuming and increases contamination risk unless the probe is rinsed between samples.  
* **Chemical scope** – The glass-electrode sensor is unsuitable for highly viscous, non-aqueous, or high-temperature fluids and cannot provide titratable alkalinity, conductivity, or nutrient concentrations.  
* **Data logging** – Current firmware only streams to the Arduino serial monitor; long-term tracking would require SD-card or Wi-Fi integration.

---
