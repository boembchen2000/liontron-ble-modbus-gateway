# liontron-ble-modbus-gateway

Reliable BLE → Modbus gateway for Liontron LiFePO₄ batteries  
used in real-world camper / motorhome installations.

Stable BLE-to-Modbus gateway for Liontron LiFePO₄ batteries  
(ESP32 + Raspberry Pi, JBD BMS).

---

# Liontron BLE Modbus Gateway

This project provides a stable gateway to read Liontron LiFePO₄ batteries
using an ESP32 via Bluetooth Low Energy (BLE) and expose the data as
Modbus RTU registers through a Raspberry Pi.

It was developed for use in a camper / motorhome environment
where reliable wired communication to an HMI is required,
but the battery BMS only offers Bluetooth connectivity.

---

# Features

- Reads two Liontron batteries with JBD BMS via BLE
- Robust BLE handling using ESP32 + NimBLE
- JSON data stream over serial
- Raspberry Pi Modbus RTU server
- Designed for HMI panels (e.g. Mochuan)
- Clear status bits and alarm logic
- Designed for continuous 24/7 operation

---

# Hardware Overview

Typical setup:

Liontron Battery (BLE)  
↓  
ESP32 (BLE Client)  
↓ UART / Serial JSON  
Raspberry Pi  
↓ RS485  
Modbus RTU Network  
↓  
HMI Panel

Required hardware:

- ESP32 (BLE client)
- Raspberry Pi
- RS485 adapter
- Liontron LiFePO₄ batteries (JBD BMS)
- Modbus RTU capable HMI (e.g. Mochuan)

---

# Project Status

This project is in active use and considered **stable**.

The system is running in a real camper installation and has
been tested for long-term reliability.

Further extensions are planned.

---

# Project Scope & Design Goals

This project focuses on **reliability and long-term stability**
in real-world camper / motorhome environments.

It is **not** a demo, proof-of-concept, or minimal hardware experiment.

## Design goals

- Stable 24/7 operation
- Clear separation of responsibilities
- Deterministic Modbus behavior for HMIs
- Easy debugging and extendability
- No hidden logic inside the HMI

---

# What This Project Is

- A **production-grade BLE-to-Modbus gateway**
- Designed for **Liontron batteries with JBD BMS**
- Optimized for **industrial HMIs**
- Built from real-world failures and lessons learned

---

# What This Project Is NOT

- ❌ Not an ESP32-only solution
- ❌ Not intended to replace the Liontron app
- ❌ Not a generic BLE framework
- ❌ Not optimized for minimum code size

Reliability beats minimal hardware.

---

# Stability Note

Bluetooth Low Energy was never designed for continuous industrial data transport.

This project works around that limitation by:

- isolating BLE handling on the ESP32
- using stateless JSON transport
- moving all validation and logic to the Raspberry Pi

This architecture has proven stable in daily operation.

---

# Installation

## ESP32 Firmware

1. Install **Arduino IDE** or **PlatformIO**
2. Install the required libraries:

- NimBLE-Arduino
- ArduinoJson

3. Flash the firmware located in:

esp32/liontron_ble_reader/

4. Configure the BLE MAC addresses of your Liontron batteries if required.

---

## Raspberry Pi Setup

Install required Python packages:

pip install pymodbus pyserial

Start the Modbus gateway:

python3 modbus_gateway.py

The Raspberry Pi will:

- read JSON data from the ESP32 via serial
- parse battery information
- update Modbus registers
- expose them via Modbus RTU over RS485

---

# ESP32 → Raspberry Pi Data Format

The ESP32 sends battery data as JSON over serial.

Example:

{
  "battery": 1,
  "voltage": 13.42,
  "current": -8.5,
  "soc": 78,
  "temp": 24.1,
  "cells": [3.35,3.36,3.35,3.36],
  "charge_ok": true,
  "discharge_ok": true
}

The Raspberry Pi converts this data into Modbus registers.

---

# Example Modbus Usage

Example using modpoll:

modpoll -m rtu -a 1 -r 100 -c 10 -b 9600 -p none -t 4:float /dev/ttyUSB0

Example registers:

Register | Description
-------- | -------------
100 | Battery Voltage
101 | Battery Current
102 | State of Charge
103 | Battery Temperature

See full documentation in:

docs/modbus-registers.md

---

# Repository Structure

liontron-ble-modbus-gateway

esp32/  
└── liontron_ble_reader/

raspberry/  
├── modbus_gateway.py  
└── config.json

docs/  
├── architecture.md  
├── ble-pitfalls-liontron.md  
├── modbus-registers.md  
├── roadmap.md  
└── hmi-screenshots.md

README.md

---

# Known Issues & BLE Pitfalls with Liontron Batteries

Working with Liontron LiFePO₄ batteries over Bluetooth Low Energy (BLE)
turned out to be significantly more complex than expected.

The following points document the main challenges and lessons learned.

---

## 1. Unstable BLE Connections

Liontron batteries use a JBD-based BMS with BLE primarily designed
for mobile apps, not for permanent connections.

Observed behavior:

- BLE connections may silently drop after minutes or hours
- The BMS sometimes stops sending notifications without disconnecting
- Reconnecting too aggressively can lock up the BMS BLE stack

Solution:

- Use NimBLE instead of the classic ESP32 BLE stack
- Implement strict timing and alternating command cycles
- Avoid reconnect storms

---

## 2. No Continuous Streaming – Polling Is Mandatory

The BMS does not push data continuously.

All values must be requested using JBD commands.

Important details:

- Command 0x03 → Basic Info
- Command 0x04 → Cell Voltages
- Commands must alternate

Sending commands too fast can corrupt frames.

Solution:

- Fixed command interval (~1 second)
- Alternating command strategy
- Validate every frame

---

## 3. Partial or Missing Data Frames

It is common to receive:

- Valid basic info but missing cell voltages
- Cell voltages without basic frame
- Frames with invalid payload length

Solution:

- Track valid_basic and valid_cells independently
- Never assume data completeness
- Always use last valid values

---

## 4. Charge / Discharge Flags Are Not Intuitive

The charge_ok and discharge_ok flags represent **permission states**, not activity.

Examples:

- Battery can be discharging while charge_ok = true
- discharge_ok = false does not mean current is zero

Lesson:

Never derive system state from current alone.

---

## 5. Cell Imbalance Thresholds Differ from the App

The Liontron mobile app applies filtering and hysteresis.

Raw BLE values may show small deltas that the app ignores.

Typical observations:

- 70–100 mV delta in raw data
- App still shows "OK"

Solution used in this project:

Warning threshold: ≥120 mV  
Alarm threshold: ≥200 mV

All thresholds are implemented on the Raspberry Pi.

---

## 6. Long-Term Stability Requires Watchdogs

BLE may appear connected while data silently stops updating.

Safeguards implemented:

- timestamp based timeout detection
- battery online state derived from data freshness
- Modbus status bits instead of raw BLE states

---

# Documentation

Detailed documentation is available here:

docs/architecture.md  
docs/ble-pitfalls-liontron.md  
docs/modbus-registers.md  
docs/roadmap.md

---

# Real-World Operation

This system is actively used in a motorhome environment
and has been running reliably over extended periods.

HMI screenshots are available in:

docs/hmi-screenshots.md

---

# Contributions

Contributions, improvements and bug reports are welcome.

If you use this project in a camper / off-grid environment,
feedback about long-term stability is especially valuable.

---

# License

This project is released under the **MIT License**.

See the LICENSE file for details.

---

# Release

Latest stable release:

https://github.com/boembchen2000/liontron-ble-modbus-gateway/releases
