Architecture Overview

This project is intentionally split into multiple layers.
Each layer has a single responsibility to maximize stability, reliability, and debuggability.

1. Battery Layer – Liontron LiFePO₄ (JBD BMS)

Liontron batteries expose their BMS via Bluetooth Low Energy (BLE)

Uses a proprietary but JBD-compatible protocol

Designed for mobile apps, not for industrial or continuous communication

BLE behavior is timing-sensitive and sometimes unreliable

Key limitation:
BLE connections may silently stall or stop sending notifications without disconnecting.

2. ESP32 Layer – BLE Client & JBD Protocol Handler

Role:
Reliable BLE client and low-level JBD protocol handler.

Responsibilities

Scan and identify batteries by BLE name

Maintain two independent BLE connections

Send JBD commands:

0x03 – Basic information

0x04 – Cell voltages

Alternate commands with fixed timing

Parse raw BMS frames

Output clean, structured JSON via serial

Why the ESP32 is limited to this role

BLE requires tight timing control

Modbus RTU timing conflicts with BLE scheduling

Debugging BLE + Modbus on a single MCU is impractical

Watchdog resets become hard to diagnose

Design decision:
The ESP32 does only BLE communication and data extraction — nothing else.

3. Serial Transport Layer – JSON Stream

The ESP32 outputs battery data as line-based JSON:

{
  "bat1": { ... },
  "bat2": { ... }
}

Why JSON?

Human-readable

Easy to debug via serial monitor

Robust against partial data loss

Easy to extend without breaking compatibility

This interface is intentionally stateless.

4. Raspberry Pi Layer – Logic & Modbus Gateway

Role:
System brain and industrial protocol bridge.

Responsibilities

Read JSON stream from ESP32

Validate and timestamp incoming data

Detect timeouts and offline states

Calculate:

State of Charge (SOC)

Cell voltage delta

Status bitfields

Overall battery state

Expose all data via Modbus RTU registers

Act as a long-term stable Modbus slave

Why Raspberry Pi?

Stable Linux serial handling

Robust error recovery

Excellent logging and debugging

Clean Python implementation

Easy future extensions (MQTT, logging, Home Assistant, etc.)

5. Modbus RTU Layer

Standard Modbus RTU over RS485

Compatible with industrial HMIs

Deterministic timing

Supports long cable runs

Register Philosophy

Simple numeric registers

One status bitfield per battery

One overall state register per battery

HMI logic kept minimal

6. HMI Layer

Role:
Visualization only.

Design Rules

No complex logic

No calculations

Only comparisons and bit checks

Clear, text-based alarms

The HMI fully trusts the Raspberry Pi to provide already-processed data.

Why This Architecture Works
Problem	Solution
Unstable BLE	Isolated to ESP32
BLE reconnect complexity	No Modbus on ESP32
Timing conflicts	Split responsibilities
Debugging difficulty	JSON inspection
HMI limitations	Preprocessed registers
Why an ESP32-Only Design Was Rejected

Although technically possible, an ESP32-only approach was rejected because:

BLE and Modbus timing interfere

Error handling becomes fragile

Debugging turns into guesswork

Long-term stability suffers

Reliability beats minimal hardware.

Conclusion

This architecture prioritizes:

Stability over elegance

Simplicity over cleverness

Explicit state over implicit assumptions

It was shaped by real-world failures, not theory.
