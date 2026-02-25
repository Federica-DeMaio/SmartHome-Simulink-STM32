# Smart Home - Embedded Systems Project (STM32 & Simulink) 🏠🔌

This repository contains the code and models for a smart home system developed in the Simulink/Stateflow environment. The system allows managing the opening of blinds, thermostat temperature, and lighting via an interactive menu and physical buttons. 

Project developed for the Embedded Systems course (Academic Year 2024-2025).

---

## 🛠️ System Features

The system manages 7 inputs and 6 outputs. The core of the project is a finite state machine (FSM) with two main macro-states: **INACTIVE** and **ACTIVE**.

### 📥 Inputs 
Interactions occur via UART serial communication and physical buttons connected to the STM32 board:
* **UART Terminal:** Allows system configuration via text input. The input is handled by a custom `CharBuffer` function that buffers up to 50 bytes and manages special characters like *Enter* and *Backspace*.
* **ENABLE Button (PC13):** Connected to the board's "User Button", it turns the entire system on and off.
* **CONFIG Button (PB0):** Allows entering or exiting the configuration mode.
* **LIGHTS Button (PC10):** Allows manual toggling (on/off) of the LED/relay.
* **TMP_LIGHTS Button (PA15):** Triggers the timed activation of the lights according to a pre-configured timer.

*Hardware Note:* External buttons are configured with pull-down resistors to avoid floating readings (the pin reads HIGH/1 only when the button is physically pressed).

### 📤 Outputs & Actuators
The outputs drive real hardware devices via the STM32 board:
* **Lights (PC2):** Digital output (0 or 1) connected to a 5V powered relay (COM port), which in turn closes the circuit to light up a LED protected by a 220 Ohm resistor.
* **Temperature (PWM via TIM 3 - CH1, CH2, CH4):** Allows the user to select a temperature from 16°C to 30°C. This value maps 3 PWM input signals to a common-cathode RGB LED, mixing the R, G, and B channels to output the corresponding color. The signal is set at a 2 KHz frequency.
* **Blinds (PWM at 50 Hz):** Controls a servomotor to simulate the opening (from 0° to 180° in 20° steps). The mapping ensures a linearly increasing duty cycle from 2.5% (for 0°) up to 12.5% (for 180°).
* **LED Status (PA5):** A digital output used to blink the board's LED at 2Hz, indicating the active working state of the system.
* **Message / UART (msg):** Provides printouts on the terminal for bidirectional interaction, error messages, and on-screen typing.

---

## 🧠 Software Architecture (Stateflow)

The behavior is managed by macro-states that make the system secure and interactive:
1. **INACTIVE_STATE:** This is the initial state where the system sets devices to their default conditions: lights off, blinds closed, minimum temperature (16°C), and cleared UART terminal.
2. **ACTIVE_STATE:** Reached by pressing the ENABLE button. Inside, two parallel logics operate: the LED blinking at 2Hz (`led` state) and the operational management (`system` state).
   * **OPERATIONAL State:** Listens and responds in real-time to physical button commands (e.g., light switches and timed activation) and periodically prints a status log on the UART terminal (every 10 seconds).
   * **CONFIGURATION State:** Reached via the CONFIG button, it enables keyboard reading. It guides the user through on-screen messages in 7 steps to customize lights, activation timer, desired temperature, and servomotor degrees. It uses "sentinel" variables to validate the changes before the final confirmation.

