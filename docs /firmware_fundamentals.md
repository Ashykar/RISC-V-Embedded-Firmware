
# Embedded Firmware Fundamentals

---

## What is Embedded Firmware?
**Embedded firmware** is specialized, low-level software programmed directly into hardware to control its physical functions and manage resource-constrained components such as GPIO, timers, serial interfaces, and sensors. Firmware is the software that tells hardware how to operate. Unlike general-purpose software (like a web browser on a laptop), embedded firmware is tightly coupled with a specific microcontroller or microprocessor to perform dedicated, predefined tasks.<br>
* It resides permanently in non-volatile memory chips directly on the device, such as internal Flash ROM, EEPROM, or NOR Flash and fits in the architectural layer directly above the physical silicon chips and below high-level application software
* It features deterministic execution, meaning it responds to real-world physical inputs within precise, predictable time limits.
<br>
**Real world application:** It Interfaces with radar and camera modules to process distance calculations and trigger autonomous emergency braking in **Advanced Driver Assistance Systems (ADAS)**.
  
---

## Role of Firmware in Microcontroller Systems
In a microcontroller-based architecture (such as an VSD Squadron Mini RISC-V device), firmware functions as the indispensable foundation layer:
* Executes the bootloader on power-up to configure core system clocks, initialize RAM.
* Translates complex hardware registers into clean functional wrappers, allowing subsequent code layers to interact safely with peripherals like timers, ADCs, and DACs.
* Manages high-priority Interrupt Service Routines (ISRs) to immediately process asynchronous hardware inputs (e.g., stopping a motor if a limit switch is struck).
* Directly manipulates the processor's power states, placing the microcontroller into deep-sleep modes to conserve power until awakened by an external signal.
  
---

## What is Application code?
**Application code** is high-level software written to perform specific tasks directly for the end-user or to fulfill a business function. Unlike low-level firmware, it runs inside an environment provided by an operating system , which completely shields it from the underlying physical hardware.
* **main.c** contains the application code . It implements the product's functional behavior—initializing an LED and a button, turning the LED on, reading a button input, and turning the LED off —using the API wrappers without ever touching raw memory addresses or physical registers.

---

## What is a firmware library ?
A firmware library is the actual implementation code that contains low-level execution logic to manipulate physical hardware components. 
* **gpio.c** represents the firmware library. It simulates the hardware logic required to initialize pins, write physical states, and read raw data. In a non-simulated system, the inner code of these functions would directly access and manipulate hardware configuration registers.

---

## What is a firmware API ?
The Firmware API is the formal specification that defines the rules, names, and parameters used to interface with the library. It contains declarations without containing the underlying hardware logic.
* **gpio.h** serves as the Firmware API . It exposes the macro definitions (GPIO_OUTPUT, GPIO_INPUT) and defines the function signatures available to the application :
* void gpio_init(int pin, int direction);
* void gpio_write(int pin, int value);
* int gpio_read(int pin);

---

## Why re-usable APIs are Crucial in Industry?
* Microchip shortages frequently force factories to swap microcontroller vendors mid-production. When firmware APIs are reusable, the application layer remains completely untouched. Only the underlying hardware driver library is swapped out preventing from having to rewrite application and also eliminating Hardware Supply Chain Risk.
* They shield application developers from complex silicon architecture. Instead of spending hours reading a datasheet to configure a micro controller SPI or I2C clock timing registers, a developer simply calls a standard API function.
* Vendor-supplied firmware libraries are heavily tested to ensure they configure hardware clocks and power states safely and reliably.

---

## Lab code

---

The provided code establishes a classic three-tier embedded software architecture

**The API Layer (gpio.h)**

* The macros #define GPIO_OUTPUT 1 and #define GPIO_INPUT 0 abstract away binary flags instead of forcing a developer to remember a hardware-specific config bit (e.g., writing a 1 or 0 into a configuration register)
* By declaring gpio_init, gpio_write, and gpio_read, it guarantees the structural layout for any application interacting with the input/output systems.

**The Firmware Library Layer (gpio.c)**

* This file represents the implementation layer. 
* The functions take simple parameters (like pin and value) and map them out . In real applications, this layer hides the tedious bitwise shifting (<<, |=, &=) required to toggle hardware.

**The Application Layer (main.c)** 

* The application file holds the product definition logic.
* It is entirely hardware-blind. 
* It does not know how the silicon tracks pin 5 or pin 3, nor does it matter.
* If the underlying microchip is swapped through different vendors, main.c remains the same. Only gpio.c needs a vendor rewrite
* The application establishes explicit system states: configuring inputs/outputs, toggling an actuator high, sampling a digital input state, and restoring safe power boundaries by resetting the pin. 

**Detailed Breakdown of the Timeline**

* Boot Phase: The system starts, logging Starting firmware application.
* Hardware Provisioning: gpio_init(LED_PIN, GPIO_OUTPUT) executes. The library decodes the argument, mapping pin 5 as an active driver state (OUTPUT).
* Sensor Allocation: gpio_init(BTN_PIN, GPIO_INPUT) runs, setting up pin 3 to read incoming voltages.
* Actuation Trigger: gpio_write(LED_PIN, 1) pulls the simulated line to high voltage (turning on the LED).
* Telemetry Sampling: gpio_read(BTN_PIN) checks the state of pin 3. The simulation returns a hardcoded high logic state (1), which is captured in the variable button_state.
* Data Processing: The app writes Button state: 1 directly to the display terminal.
* Safe State: gpio_write(LED_PIN, 0) safely turns off the LED to conserve power.
* Teardown Phase: The program hits its final branch, reporting clean software termination with an exit code of 0.
  
---

When compiled and run, the code proceeds through a precise linear timeline, translating the high-level logic into sequenced output logs shown below 

<img width="1920" height="1020" alt="Task1" src="https://github.com/user-attachments/assets/a52955ab-baeb-4432-b595-589fd44999b7" />

