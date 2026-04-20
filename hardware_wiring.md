---
layout: default
title:  Hardware Wiring
nav_order: 2
---

# 1 Hardware BOM & Wiring Diagram

*(Please refer to the actual wiring diagram image below )*


## 1.  Bill of Materials 
To ensure the successful assembly and stable operation of the gimbal, please prepare your hardware materials strictly according to the following list. We have categorized the core components into Control, Drive, Power, and Mechanics.

| Component | Specification | Qty | Purpose |
| :--- | :--- | :---: | :--- |
| **🧠 Control & Vision** | | | |
| Raspberry Pi Pico 2 | Microcontroller board | 1 | Low-level motor control and embedded task execution |
| Raspberry Pi 5 | Single-board computer | 1 | Main processing unit for vision, control logic, and system coordination |
| Pi 5 Active Cooler | Official cooling module | 1 | Thermal management for stable Pi 5 operation |
| Pi Camera Module 3 | Official camera module | 1 | Visual input for object detection and tracking |
| Camera Ribbon Cable | CSI camera cable | 1 | Connects Camera Module 3 to Raspberry Pi 5 |
| **⚙️ Motors & Drivers** | | | |
| GM3506 Gimbal Motor | Brushless gimbal motor | 2 | Two-axis gimbal actuation for yaw and pitch movement |
| SimpleFOC Mini Dual Driver | One-driver-for-two-motors board | 1 | Drives and controls the two gimbal motors |
| XH2.54 5-Pin Cable | 5-pin Dupont/JST-style cable | 1 | Electrical connection between driver board and motors/sensors |
| **🔋 Power Supply** | | | |
| Auline 4800mAh LiPo Battery | Rechargeable LiPo battery | 1 | Portable power supply for the whole system |
| USB-C 5V Buck Converter | DC step-down module | 1 | Converts battery voltage to stable 5V supply for electronics |
| XT60 Cable | Standard XT60 cable | 1 | Main battery connection and power transmission |
| XT60 One-to-Two Cable | XT60 splitter cable | 1 | Power distribution from battery to multiple modules |
| **🏗️ Mechanics & Misc** | | | |
| 3D Printed Base | Custom structural part | 1 | Mechanical support for mounting the system components |
| 3D Printed Gimbal Arm | Custom axis arm structure | 1 set | Supports motor installation and gimbal movement |
| Phone Clamp | Adjustable holder | 1 | Holds the mobile phone securely on the gimbal platform |
| Dupont Wires | Various jumper wires | - | General signal and power wiring between modules |
| M2 / M2.5 Screws and Nuts | Fasteners | - | Mechanical assembly and structural fixing |

---

## 2.  Pinout Connection Table
To ensure system stability and prevent catastrophic hardware damage, the wiring is divided into three distinct logical links: Power Distribution, Master-Slave Communication, and Motor Control.

> **CRITICAL DANGER**
> 
> Never cross-wire the 12V battery power with the 5V logic circuitry. Double-check all connections with a multimeter before plugging in the battery.
{: .warning }

### 2.1 Power Distribution Routing 
The system relies on a single 12V LiPo battery. We use an XT60 splitter to physically separate the high-current motor power from the sensitive logic power.

1. **Battery to Splitter**: Connect the 12V LiPo battery to the XT60 One-to-Two Cable.
2. **Logic Power (5V)**: 
   * Connect one end of the XT60 splitter to the **USB-C 5V Buck Converter**.
   * Plug the USB-C output into the **Raspberry Pi 5** power port. *(Note: Ensure your buck converter supports at least 5V/3A output to prevent Pi 5 from browning out during heavy NCNN inference).*
3. **Motor Power (12V)**: 
   * Connect the other end of the XT60 splitter directly to the `VIN` and `GND` terminals of the **SimpleFOC Mini Dual Driver Board**.
4. **Pico Power**:
   * Power the Pico 2 directly from the Raspberry Pi 5 using a standard USB cable (Pi 5 USB-A to Pico Micro-USB/USB-C). This provides a stable 5V and implicitly connects their grounds.

### 2.2 Master-Slave Communication Link (Pi 5 ↔ Pico 2)
Since our architecture uses `/dev/serial0` on the Pi 5 to send the 14-byte velocity protocol, we must hardwire the UART pins. 

> **IMPORTANT: Common Ground**
> 
> Even if powered via USB, it is highly recommended to add a dedicated physical jumper wire between the Pi 5 GND and Pico GND to ensure the UART voltage reference is identical.
{: .highlight }

| Raspberry Pi 5 Pin (Master) | Pico 2 Pin (Slave) | Function |
| :--- | :--- | :--- |
| **Pin 8** (GPIO 14 / TXD) | **GP1** (UART0 RX) | Transmits Target Velocity to Pico |
| **Pin 10** (GPIO 15 / RXD) | **GP0** (UART0 TX) | Receives status from Pico (if any) |
| **Pin 6** (Ground / GND) | **Any GND Pin** (e.g., Pin 38) | Reference Ground (Common GND) |

### 2.3 Motor Control Link (Pico 2 ↔ SimpleFOC Mini)
This link transmits the high-frequency SPWM signals from the Pico to the motor driver. A strictly correct pinout is required for the trapezoidal acceleration and FOC algorithms to function.

| Pico 2 Pin (Controller) | SimpleFOC Mini Pin | Axis | Function |
| :--- | :--- | :--- | :--- |
| **GP2** | IN1 (Motor A) | Pitch | Phase U PWM |
| **GP4** | IN2 (Motor A) | Pitch | Phase V PWM |
| **GP6** | IN3 (Motor A) | Pitch | Phase W PWM |
| **GP16** | EN (Motor A) | Pitch | Motor Enable |
| **GP10** | IN1 (Motor B) | Yaw | Phase U PWM |
| **GP11** | IN2 (Motor B) | Yaw | Phase V PWM |
| **GP12** | IN3 (Motor B) | Yaw | Phase W PWM |
| **GP15** | EN (Motor B) | Yaw | Motor Enable |
| **Any GND Pin** | GND (Signal) | Both | Common GND for PWM signals |
![Pinout Connection Table 1](https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal/blob/main/assets/images/pinout%20connection%20table%201.png?raw=true)

![Pinout Connection Table 2](https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal/blob/main/assets/images/pinout%20connection%20table%202.png?raw=true)
---

## 3.  Power Supply Requirements & Warnings

> [!CAUTION]
> **English Power Warnings**
> * The Micro-USB port on the Raspberry Pi Pico is **ONLY** for connecting to a PC to write code or flash firmware. Its 5V output is absolutely insufficient to drive the brushless motors!
> * To drive the motors, you **MUST** connect an independent 12V power supply directly to the power input terminals (VIN/GND) of the SimpleFOC driver boards.
> * **EXTREME DANGER:** Never accidentally connect the 12V power supply to any of the Pico's GPIO pins, as this will instantly fry the microcontroller.
