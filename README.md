# liontron-ble-modbus-gateway
Stable BLE-to-Modbus gateway for Liontron LiFePO₄ batteries (ESP32 + Raspberry Pi, JBD BMS).

# Liontron BLE Modbus Gateway

This project provides a stable gateway to read Liontron LiFePO₄ batteries
using an ESP32 via BLE and expose the data as Modbus RTU registers
through a Raspberry Pi.

It was developed for use in a camper / motorhome environment
where reliable wired communication to an HMI is required,
but the battery BMS only offers Bluetooth Low Energy.

## Features

- Reads two Liontron batteries with JBD BMS via BLE
- Robust BLE handling using ESP32 + NimBLE
- JSON data stream over serial
- Raspberry Pi Modbus RTU server
- Designed for HMI panels (e.g. Mochuan)
- Clear status bits and alarm logic

## Hardware Overview

- ESP32 (BLE client)
- Raspberry Pi
- RS485 adapter
- Liontron LiFePO₄ batteries (JBD BMS)
- Modbus RTU HMI

## Project Status

This project is in active use and considered **stable**.
Further extensions are planned.

More detailed documentation will follow.


Known Issues & BLE Pitfalls with Liontron Batteries

Working with Liontron LiFePO₄ batteries over Bluetooth Low Energy (BLE) turned out to be significantly more complex than expected.
The following points document the main challenges and lessons learned during development.

1. Unstable BLE Connections

Liontron batteries use a JBD-based BMS with BLE primarily designed for mobile apps, not for permanent connections.

Observed behavior:

BLE connections may silently drop after minutes or hours

The BMS sometimes stops sending notifications without disconnecting

Reconnecting too aggressively can lock up the BMS BLE stack temporarily

Solution:

Use NimBLE instead of the classic ESP32 BLE stack

Implement strict timing and alternating command cycles

Avoid reconnect storms by detecting stale data instead of link state alone

2. No Continuous Streaming – Polling Is Mandatory

The BMS does not push data continuously.
All values must be actively requested using JBD commands.

Important details:

Command 0x03 (Basic Info) and 0x04 (Cell Voltages) must be alternated

Sending commands too fast causes missed or corrupted frames

Notifications may arrive delayed or out of order

Solution:

Fixed command interval (≈1 s)

Alternating command strategy (03 ↔ 04)

Validate every received frame before processing

3. Partial or Missing Data Frames

It is common to receive:

Valid basic info but missing cell voltages

Cell voltages without a preceding basic frame

Frames with correct header but invalid payload length

Solution:

Track valid_basic and valid_cells independently

Never assume data completeness

Build higher-level logic on last known valid values

4. Charge / Discharge Flags Are Not Intuitive

The charge_ok and discharge_ok flags are permission flags, not activity indicators.

Examples:

Battery can be discharging while charge_ok = true

discharge_ok = false does not mean current is zero

Flags reflect allowed state, not actual power flow

Lesson:

Do not derive system state from current alone

Always interpret flags and current separately in the HMI

5. Cell Imbalance Thresholds Differ from the App

The Liontron mobile app applies internal filtering and hysteresis.
Raw BLE values may show small deltas that the app ignores.

Typical observations:

70–100 mV delta shown in raw data

App still reports “OK”

Immediate alarm logic leads to false positives

Solution used in this project:

Warning threshold: ≥120 mV

Alarm threshold: ≥200 mV

All thresholds implemented on the Raspberry Pi, not the ESP32

6. Long-Term Stability Requires Watchdogs

BLE may appear connected while data silently stops updating.

Implemented safeguards:

Timestamp-based timeout detection

“Battery online” status derived from data freshness

Modbus status bits instead of raw BLE states

Summary

Liontron BLE works, but only if treated as an unreliable transport layer.

Key principles:

Never trust connection state alone

Validate every frame

Expect missing data

Separate permissions from measurements

Shift safety and logic decisions to the Modbus layer

This project prioritizes stability over speed and is designed for
24/7 operation in a motorhome environment.

## Documentation

This project is documented in detail in the following files: 

- 📘 [System Architecture](docs/architecture.md)
- ⚠️ [BLE Pitfalls & Liontron Quirks](docs/ble-pitfalls.md)
- 📟 [Modbus Register Map](docs/modbus-registers.md)


