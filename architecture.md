
Each layer has a **single responsibility** to maximize stability and debuggability.

---

## 1. Battery Layer – Liontron LiFePO₄ (JBD BMS)

- Liontron batteries expose their BMS via **Bluetooth Low Energy**
- Uses a proprietary but JBD-compatible protocol
- Intended for mobile apps, not industrial communication
- BLE behavior is timing-sensitive and sometimes unreliable

**Key limitation:**  
BLE connections can drop or stop sending notifications without warning.

---

## 2. ESP32 Layer – BLE + JBD Protocol

**Role:**  
Reliable BLE client and JBD protocol handler.

### Responsibilities

- Scan and identify batteries by BLE name
- Maintain two independent BLE connections
- Send JBD commands:
  - CMD 0x03 – Basic information
  - CMD 0x04 – Cell voltages
- Alternate commands with fixed timing
- Parse raw BMS frames
- Output clean JSON via serial

### Why ESP32 is limited to this role

- BLE timing requires tight control
- Modbus timing conflicts with BLE handling
- Debugging BLE + Modbus on one MCU is impractical
- ESP32 watchdog resets become hard to diagnose

**Design decision:**  
ESP32 does **only BLE + data extraction**, nothing else.

---

## 3. Serial Transport – JSON Stream

The ESP32 outputs battery data as line-based JSON:

```json
{
  "bat1": { ... },
  "bat2": { ... }
}
Why JSON?

Human-readable

Easy to debug with serial monitor

Robust against partial data loss

Easy to extend without breaking compatibility

This link is intentionally kept stateless.

4. Raspberry Pi Layer – Logic + Modbus Gateway

Role:
System brain and industrial protocol bridge.

Responsibilities

Read JSON stream from ESP32

Validate and time-stamp incoming data

Detect timeouts and offline states

Calculate:

SOC

Cell delta

Status bits

Overall battery state

Expose data as Modbus RTU registers

Serve as long-term stable Modbus slave

Why Raspberry Pi?

Stable Linux serial handling

Better error recovery

Easy logging and debugging

Clean Python implementation

Easy future extensions (MQTT, logging, HA, etc.)

5. Modbus RTU Layer

Standard Modbus RTU over RS485

Compatible with industrial HMIs

Deterministic timing

Long cable runs supported

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

Clear text-based alarms

The HMI trusts the Raspberry Pi to provide already-processed data.

Why This Architecture Works
Problem	Solution
Unstable BLE	Isolated to ESP32
BLE reconnect complexity	No Modbus on ESP32
Timing conflicts	Split responsibilities
Debugging difficulty	JSON inspection
HMI limitations	Preprocessed registers
Why ESP32-Only Was Rejected

Although technically possible, an ESP32-only design was rejected because:

BLE and Modbus timing interfere

Error handling becomes fragile

Debugging becomes guesswork

Long-term stability suffers

Reliability beats minimal hardware.

Conclusion

This architecture prioritizes:

Stability over elegance

Simplicity over cleverness

Explicit state over implicit assumptions

It was shaped by real-world failures, not theory.

End of document


---

