# System Architecture

## Architecture Overview

This project is intentionally split into multiple layers.  
Each layer has a single responsibility to maximize stability, reliability, and debuggability.

The system separates:

- BLE communication
- data transport
- system logic
- industrial protocol handling
- visualization

---

# 1. Battery Layer – Liontron LiFePO₄ (JBD BMS)

Liontron batteries expose their BMS via Bluetooth Low Energy (BLE).

Characteristics:

- Proprietary but JBD-compatible protocol
- Designed for mobile apps, not industrial communication
- BLE behavior is timing-sensitive and sometimes unreliable

### Key limitation

BLE connections may silently stall or stop sending notifications without disconnecting.

Because of this, BLE must be treated as an unreliable transport layer.

---

# 2. ESP32 Layer – BLE Client & JBD Protocol Handler

Role:  
Reliable BLE client and low-level JBD protocol handler.

### Responsibilities

- Scan and identify batteries by BLE name
- Maintain two independent BLE connections
- Send JBD commands:

0x03 – Basic information  
0x04 – Cell voltages

- Alternate commands with fixed timing
- Parse raw BMS frames
- Output clean structured JSON via serial

### Why the ESP32 is limited to this role

BLE requires tight timing control.

Running Modbus RTU at the same time causes problems:

- Modbus timing conflicts with BLE scheduling
- Debugging BLE + Modbus on a single MCU becomes difficult
- Watchdog resets become hard to diagnose

### Design decision

The ESP32 performs only BLE communication and data extraction.

Nothing else.

---

# 3. Serial Transport Layer – JSON Stream

The ESP32 outputs battery data as line-based JSON:

{
  "bat1": { ... },
  "bat2": { ... }
}

### Why JSON?

- Human-readable
- Easy debugging via serial monitor
- Robust against partial data loss
- Easy to extend without breaking compatibility

This interface is intentionally stateless.

---

# 4. Raspberry Pi Layer – Logic & Modbus Gateway

Role:  
System brain and industrial protocol bridge.

### Responsibilities

- Read JSON stream from ESP32
- Validate incoming data
- Timestamp received values
- Detect timeouts and offline states

Calculate system values:

- State of Charge (SOC)
- Cell voltage delta
- Status bitfields
- Overall battery state

Expose all values via Modbus RTU registers.

The Raspberry Pi acts as a stable Modbus slave device.

### Why Raspberry Pi?

- Stable Linux serial handling
- Robust error recovery
- Excellent logging and debugging
- Clean Python implementation
- Easy future extensions

Possible extensions:

- MQTT
- data logging
- Home Assistant integration
- remote diagnostics

---

# 5. Modbus RTU Layer

Communication between the gateway and the HMI uses:

Modbus RTU over RS485

Advantages:

- Industrial standard protocol
- Deterministic timing
- Very robust
- Supports long cable runs
- Compatible with many HMI systems

### Register Philosophy

The Modbus design follows a simple philosophy:

- Simple numeric registers
- One status bitfield per battery
- One overall state register per battery
- Preprocessed values only

The HMI should not perform complex calculations.

---

# 6. HMI Layer

Role:  
Visualization only.

### Design Rules

- No complex logic
- No calculations
- Only comparisons and bit checks
- Clear text-based alarms

The HMI fully trusts the Raspberry Pi  
to provide already processed data.

---

# Why This Architecture Works

Problem → Solution

Unstable BLE → Isolated to ESP32  
BLE reconnect complexity → No Modbus on ESP32  
Timing conflicts → Split responsibilities  
Debugging difficulty → JSON inspection  
HMI limitations → Preprocessed Modbus registers

---

# Why an ESP32-Only Design Was Rejected

Although technically possible, an ESP32-only approach was rejected because:

- BLE and Modbus timing interfere
- Error handling becomes fragile
- Debugging turns into guesswork
- Long-term stability suffers

Reliability beats minimal hardware.

---

# Conclusion

This architecture prioritizes:

- Stability over elegance
- Simplicity over cleverness
- Explicit state over implicit assumptions

It was shaped by real-world failures, not theory.
