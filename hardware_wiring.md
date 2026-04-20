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
![Pinout Connection Table 1](https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal/blob/main/assets/images/pinout%20connection%20table%201.png?raw=true)

![Pinout Connection Table 2](https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal/blob/main/assets/images/pinout%20connection%20table%202.png?raw=true)
---

## 3.  Power Supply Requirements & Warnings

> [!CAUTION]
> **English Power Warnings**
> * The Micro-USB port on the Raspberry Pi Pico is **ONLY** for connecting to a PC to write code or flash firmware. Its 5V output is absolutely insufficient to drive the brushless motors!
> * To drive the motors, you **MUST** connect an independent 12V power supply directly to the power input terminals (VIN/GND) of the SimpleFOC driver boards.
> * **EXTREME DANGER:** Never accidentally connect the 12V power supply to any of the Pico's GPIO pins, as this will instantly fry the microcontroller.
