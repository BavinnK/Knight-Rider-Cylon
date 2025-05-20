# Knight Rider / Cylon LED Chaser (555 Timer + 74HC595)

This project implements a classic "Knight Rider" or "Cylon" style LED chaser effect using a 555 timer IC in astable mode to generate a clock signal, and a 74HC595 shift register to drive 8 LEDs in sequence.

## Project Overview

The circuit generates a visual effect where a single lit LED appears to scan across a row of 8 LEDs. This is achieved by:
1.  **Clock Generation:** A 555 timer is configured as an astable multivibrator, producing a continuous square wave (clock pulse).
2.  **Serial Data Shifting:** A 74HC595 8-bit shift register takes a serial data input and shifts it through its internal storage registers on each clock pulse.
3.  **Parallel Output:** The state of the shift register's internal storage is then latched to 8 parallel outputs, each driving an LED.
4.  **User Input:**
    *   A push button allows manual input of a '1' (HIGH) bit into the shift register's serial input (SER).
    *   Another push button allows manual clearing (resetting) of the shift register.

## Circuit Diagram

![20250520_191832](https://github.com/user-attachments/assets/485b2d71-ec9f-4b96-b708-1939f6c57cce)


## Breadboard Implementation

![20250520_191845](https://github.com/user-attachments/assets/fae7f986-4768-45ba-a42b-8c152279712b)


## Components Used

*   **ICs:**
    *   1x NE555 Timer IC (or equivalent, e.g., LM555)
    *   1x 74HC595 8-bit Shift Register
    *   *(Optional for robust looping: 1x 74HC32 Quad 2-Input OR Gate IC)*
*   **LEDs:**
    *   8x Red LEDs (or any color)
*   **Resistors:**
    *   1x 1kΩ (R1 for 555 timer)
    *   1x 10kΩ (R2 for 555 timer)
    *   8x 220Ω (Current limiting for LEDs, as per your schematic)
    *   1x 10kΩ (Pull-up for 74HC595 SRCLR)
    *   1x 10kΩ (Pull-down for 74HC595 SER)
    *   *(Note: The OE pin is directly tied to ground in the schematic, so no pull-down resistor is strictly needed there, but good practice if it were switched.)*
*   **Capacitors:**
    *   1x 10µF (Electrolytic, C1 for 555 timer timing)
    *   1x 10nF (Ceramic, C2 for 555 timer Control Voltage pin bypass - 0.01µF)
    *   1x 0.1µF (Ceramic, Bypass capacitor for 74HC595 VCC - 100nF)
    *   1x 0.1µF (Ceramic, Bypass capacitor for 555 Timer VCC - 100nF)
*   **Miscellaneous:**
    *   2x Push Button Switches
    *   1x Breadboard
    *   Jumper Wires
    *   5V DC Power Supply

## Circuit Explanation

### 1. 555 Timer (Astable Mode - Clock Generator)

The 555 timer is configured in astable mode to generate a continuous clock signal.
*   **VCC (Pin 8) and GND (Pin 1):** Connected to 5V and Ground respectively.
*   **RESET (Pin 4):** Connected to VCC to prevent accidental resets.
*   **DISCHARGE (Pin 7):** Connected to VCC via R1 (1kΩ) and to THRESHOLD/TRIGGER via R2 (10kΩ).
*   **THRESHOLD (Pin 6) and TRIGGER (Pin 2):** Connected together and to the junction of R2 and C1 (10µF). C1 is connected to Ground.
*   **CONTROL VOLTAGE (Pin 5):** Connected to Ground via C2 (10nF) for noise decoupling.
*   **OUTPUT (Pin 3):** Provides the clock signal. This output is connected to SRCLK (Pin 11) and RCLK (Pin 12) of the 74HC595.

**Clock Frequency and Duty Cycle (Approximate):**
With R1 = 1kΩ, R2 = 10kΩ, C1 = 10µF:
*   Time High (t_H) = 0.693 * (R1 + R2) * C1 = 0.693 * (1kΩ + 10kΩ) * 10µF ≈ 0.07623 seconds
*   Time Low (t_L) = 0.693 * R2 * C1 = 0.693 * 10kΩ * 10µF ≈ 0.0693 seconds
*   Period (T) = t_H + t_L ≈ 0.14553 seconds
*   **Frequency (f)** = 1 / T ≈ 1 / 0.14553 ≈ **6.87 Hz**
*   **Duty Cycle** = (t_H / T) * 100% ≈ (0.07623 / 0.14553) * 100% ≈ **52.38%**


### 2. 74HC595 Shift Register

The 74HC595 takes serial data in and outputs it in parallel to drive the LEDs.
*   **VCC (Pin 16) and GND (Pin 8):** Connected to 5V and Ground. A 0.1µF bypass capacitor is placed between VCC and GND close to the IC.
*   **SER (Pin 14 - Serial Data Input):**
    *   Connected to a push button that, when pressed, connects SER to 5V (HIGH).
    *   A 1kΩ pull-down resistor connects SER to Ground, ensuring it's LOW when the button is not pressed.
*   **SRCLK (Pin 11 - Shift Register Clock Input):** Connected to the 555 timer's Output (Pin 3). Data at SER is shifted into the register on the rising edge of this clock.
*   **RCLK (Pin 12 - Storage Register Clock Input / Latch):** Also connected to the 555 timer's Output (Pin 3). When SRCLK and RCLK are tied together, the data shifted into the shift register is immediately transferred to the storage/latch register on the same clock edge, making it visible on the outputs.
*   **SRCLR (Pin 10 - Shift Register Clear Input, Active LOW):**
    *   Connected to a push button that, when pressed, connects SRCLR to Ground (LOW), clearing all bits in the shift register to '0'.
    *   A 10kΩ pull-up resistor connects SRCLR to 5V, keeping it HIGH (inactive) when the button is not pressed.
*   **OE (Pin 13 - Output Enable, Active LOW):** Connected directly to Ground. This permanently enables the parallel outputs (QA-QH).
*   **QA - QH (Pins 15, 1-7 - Parallel Outputs):** Each output is connected to the anode of an LED. The cathode of each LED is connected through a current-limiting resistor (e.g., 220Ω) to Ground.
*   **QH' (Pin 9 - Serial Data Output):** In the basic configuration, this pin is unused. See "Potential Improvements" for looping functionality.

## How It Works

1.  **Power On:** The 555 timer starts generating a clock signal (approx. 6.87 Hz). Initially, the shift register might contain random data, or if the reset button was pressed, all outputs will be LOW (LEDs OFF).
2.  **Reset (Optional):** Pressing the "Clear" button (connected to SRCLR) pulls SRCLR LOW, resetting all internal bits of the shift register to '0'. All LEDs will turn OFF.
3.  **Data Input:**
    *   To start the sequence, press and hold the "Data Input" button (connected to SER). This makes the SER input HIGH.
    *   On the next rising edge of the clock signal from the 555 timer (which goes to both SRCLK and RCLK):
        *   The HIGH value at SER is shifted into the first bit of the shift register (QA's internal stage).
        *   Simultaneously, this new state is latched to the outputs. QA becomes HIGH, and the first LED turns ON.
4.  **Shifting the Data:**
    *   Release the "Data Input" button. SER is now LOW (due to the pull-down resistor).
    *   On each subsequent rising edge of the clock signal:
        *   The current value at SER (which is LOW) is shifted into the first bit.
        *   The existing bits in the register are shifted one position down (e.g., the bit that was at QA moves to QB, QB to QC, and so on).
        *   The new state of the register is latched to the outputs.
5.  **The Effect:** This process makes the single '1' bit (the lit LED) appear to "scan" or "chase" across the row of 8 LEDs. If QH' is not looped back to SER, once the '1' bit shifts out of QH (the last LED), all LEDs will be OFF (as LOWs are being shifted in) until a new HIGH bit is manually entered via the SER button.

## Potential Improvements / Further Development

*   **Understanding QH' (Pin 9 - Serial Data Output):**
    *   Currently, in the described circuit, QH' (Pin 9) of the 74HC595 is not connected to anything. This means that when you press the "Data Input" button to introduce a '1', the LEDs will light up sequentially from QA to QH. Once the '1' bit shifts out of QH, it is lost, and LOW signals (0s) continue to be shifted in (if the "Data Input" button is released), causing the sequence to stop unless a new '1' is manually entered.

*   **Automatic Looping (Creating a Continuous Chase):**
    1.  **Simple Jumper Loop (for Experimentation):**
        *   To make the '1' bit automatically loop back to the beginning after it reaches the last LED (QH), you can temporarily connect QH' (Pin 9) directly to SER (Pin 14 - Serial Data Input) using a jumper wire.
        *   Once a '1' is introduced into the register, it will continuously circulate, creating a single-LED chase.
        *   *Note:* With this direct connection, the "Data Input" push button will still inject a '1', but if a '1' is already circulating, pressing the button will effectively add another '1' if held, or might not have a noticeable effect if a '1' is already present at QH' during the clock pulse.

    2.  **Robust Looping with an OR Gate (Recommended for permanent loop + manual input):**
        *   For a more robust solution that allows both continuous looping and reliable manual input via the "Data Input" push button (e.g., to start the sequence if it's all zeros, or to introduce multiple lit LEDs if desired), you can use a 2-input OR gate (e.g., a 74HC32 IC).
        *   **Connections:**
            *   Input A of OR gate: Connect to QH' (Pin 9) of the 74HC595.
            *   Input B of OR gate: Connect to the output of your "Data Input" push button (the point that goes HIGH when the button is pressed, typically before the pull-down resistor if the pull-down is directly on SER).
            *   Output Y of OR gate: Connect to SER (Pin 14) of the 74HC595. The 1kΩ pull-down resistor should remain on SER.
        *   **How it works:** The SER pin will now go HIGH if *either* the last bit from QH' is HIGH (causing it to loop) *or* if the "Data Input" push button is pressed. This allows the sequence to loop automatically once started and also allows you to inject a new '1' bit at any time.

*   **True Knight Rider (Back and Forth Scan):** Requires more complex logic or a microcontroller. This typically involves using the shift register's ability to shift in both directions (if using a bidirectional shift register) or using additional logic/microcontroller programming to control the data being fed into SER and potentially reversing the LED connection order or using multiple shift registers.

*   **Adjustable Speed:** Replace the fixed resistor R2 (10kΩ) for the 555 timer with a potentiometer (e.g., 50kΩ or 100kΩ) in series with a smaller fixed resistor (e.g., 1kΩ, to prevent a zero-ohm condition for R2). This would allow manual adjustment of the clock frequency and thus the speed of the LED chase.



