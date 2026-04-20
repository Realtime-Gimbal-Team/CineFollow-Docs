---
layout: default
title: Operation & Protocol
nav_order: 4
---

# 4. Operation, Communication & Calibration

## 4.1 First Boot: What to Expect

When you power on the system for the first time, the gimbal undergoes a safety initialization sequence. 

1. **Silence (0-2 Seconds)**: After the USB/Power is connected, the system remains silent for about 2 seconds to allow the Raspberry Pi Pico and sensors to stabilize.
2. **Soft-Start Phase**: You will notice the motors slowly "stiffening" up. The voltage is ramped up at a rate of **2V/s** to reach the safety limit (4.5V). 
3. **Idle State**: Once soft-start is complete, the gimbal will maintain its current position.

> **WARNING**
> 
> **DO NOT** manually twist or force the gimbal during the first 5 seconds of boot-up. Forcing the motors during the soft-start phase can disrupt the internal alignment and may lead to overcurrent.
{: .warning }

---

## 4.2 Serial Communication Protocol

To interface with the gimbal system via an external controller (e.g., Raspberry Pi 5), the following Finite State Machine (FSM) based protocol must be followed. This protocol ensures robust data transmission with error checking.

### 1. Serial Configuration
* **Baud Rate**: `115200`
* **Data Bits**: 8
* **Stop Bits**: 1
* **Parity**: None
* **Hardware Flow Control**: Disabled

### 2. Data Frame Structure
The system utilizes a dynamic-length packet. For dual-axis velocity control, a **14-byte** frame is required.

| Byte Index | Field | Length | Description |
| :--- | :--- | :---: | :--- |
| 0 | Head 1 | 1 Byte | Start Frame 1: Fixed at `0x55` |
| 1 | Head 2 | 1 Byte | Start Frame 2: Fixed at `0xAA` |
| 2 | CMD | 1 Byte | Command ID: Defines the packet function |
| 3 | LEN | 1 Byte | Data Length: Fixed at `0x08` for dual velocity |
| 4 - 7 | Data (Pitch) | 4 Bytes | Target Pitch Velocity (IEEE 754 `float`, Little-Endian) |
| 8 - 11 | Data (Yaw) | 4 Bytes | Target Yaw Velocity (IEEE 754 `float`, Little-Endian) |
| 12 | Checksum | 1 Byte | Error Detection: Sum of CMD, LEN, and Data |
| 13 | Tail | 1 Byte | End Frame: Fixed at `0x0D` (CR) |

### 3. Payload Interpretation
The 8 bytes of data are mapped directly into memory using `memcpy` to represent two standard 32-bit floating-point numbers:
* **Pitch Velocity**: Bytes 4 through 7.
* **Yaw Velocity**: Bytes 8 through 11.

> **IMPORTANT**
> 
> The Raspberry Pi 5 (Sender) must use **Little-Endian** byte order when packing float values. In Python, use `struct.pack('<ff', pitch, yaw)`.
{: .highlight }

### 4. Checksum Calculation
The checksum is the 8-bit sum of the Command, Length, and all Data bytes. Any overflow is discarded (effectively a modulo 256 operation).

**Calculation Logic:**
```text
Checksum = (CMD + LEN + Sum_of_All_Data_Bytes) MOD 256
```
## 4. Frequently Asked Questions (FAQ)

### Q1: I plugged in the 12V power supply, but the motors aren't spinning at all. What should I check?
**A:** First, verify the physical connection between the Raspberry Pi 5 and the Pico. 
The 12V power supply only powers the SimpleFOC driver board. The Pico microcontroller relies on the Raspberry Pi 5 for its 5V power and data connection (typically via USB or GPIO). If the cable between the Pi 5 and the Pico is disconnected or faulty, the Pico's control logic will not boot. Furthermore, ensure that all components (Pi 5, Pico, and Driver) share a **Common Ground (GND)**. Without a shared ground, the PWM control signals will float and the motors will not respond.

### Q2: The gimbal powers on normally, but goes out of control at maximum speed the moment it receives a serial command. Why?
**A:** This is a classic **Endianness mismatch** error. 
The Pico parses the 8-byte payload directly into floats using `memcpy`. Since the Pico operates on a Little-Endian architecture, it expects the bytes in reverse order. If your tracking script on the Raspberry Pi 5 sends the data in Big-Endian format, a safe target velocity like `1.5` will be misinterpreted as a massive, uncontrollable number. Ensure your sender script explicitly packs data in Little-Endian (e.g., using `struct.pack('<ff', pitch_vel, yaw_vel)` in Python).

### Q3: My Pi 5 is sending data, but the gimbal doesn't move. The serial monitor shows `[SAFE] Timeout!`.
**A:** The Pico is actively rejecting your data frames or timing out. Check these two critical points:
1. **Invalid Checksum/Tail**: If your checksum algorithm is wrong, or you forgot the `0x0D` tail byte, the FSM (Finite State Machine) drops the packet.
2. **Blocking Vision Code**: The gimbal's safety mechanism requires a valid packet at least every **500ms**. If the computer vision algorithm on your Pi 5 is too heavy and drops the frame rate below 2 FPS, the serial transmission will stall, triggering the Pico's hardware timeout safety lock.
