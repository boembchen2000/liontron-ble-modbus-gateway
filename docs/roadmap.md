# Roadmap

This document outlines **possible future extensions** of the Liontron BLE Modbus Gateway.

The project is already **stable and in productive use**.  
All roadmap items are **optional** and driven by real-world needs, not feature pressure.

---

# Current Status

✅ Dual Liontron battery support  
✅ Stable BLE handling via ESP32  
✅ Deterministic Modbus RTU registers  
✅ Clear status bits and alarm logic  
✅ Proven operation with industrial HMI panels  
✅ Separation of BLE and Modbus logic  
✅ Production use in a camper environment

---

# Planned / Possible Extensions

## 🔹 1. Extended BMS Diagnostics

Possible improvements:

- Additional warning bit decoding
- More detailed BMS fault classification
- Optional exposure of raw BMS warning flags
- Improved cell imbalance diagnostics

Status: *planned, low priority*

---

## 🔹 2. Logging & Diagnostics (Raspberry Pi)

Optional long-term data logging:

- CSV logging
- JSON event logs
- Timestamped battery state changes
- Simple log rotation

Purpose:

- troubleshooting
- long-term battery analysis
- post-mortem diagnostics

Status: *optional*

---

## 🔹 3. MQTT / Home Assistant Bridge

Optional integration layer:

- Publish battery data via MQTT
- Home Assistant discovery
- Remote monitoring

Important design rule:

The MQTT layer must **not interfere with Modbus timing**.

Status: *future idea*

---

## 🔹 4. Configuration File Support

Introduce a central configuration file.

Possible parameters:

- battery BLE names
- Modbus register offsets
- SOC thresholds
- cell delta thresholds
- timeout values

Goal:

- easier customization
- fewer code modifications

Status: *nice-to-have*

---

## 🔹 5. Camper System Integration

Possible integration of additional vehicle sensors.

Examples:

- water tank level (4–20 mA sensor)
- interior temperature
- EBL system state
- pump status

These values would be exposed via additional Modbus registers.

Status: *planned for vehicle integration*

---

## 🔹 6. Gateway Self-Monitoring

Additional gateway safety features.

Examples:

- gateway uptime register
- watchdog restart counter
- ESP32 heartbeat monitoring
- serial communication watchdog

Purpose:

Detect gateway failures from the HMI.

Status: *future improvement*

---

## 🔹 7. Multi-Battery Scalability

Current system supports:

- 2 Liontron batteries

Future extension could support:

- 3–4 batteries
- configurable battery count

This would require dynamic register allocation.

Status: *unlikely but technically possible*

---

# Explicit Non-Goals

The following items are **intentionally not planned**:

❌ ESP32-only Modbus implementation  
❌ BLE reconnection hacks inside HMI  
❌ Complex logic inside the HMI  
❌ Battery write/configuration commands  
❌ Direct BLE exposure to external systems

This project focuses on **read-only, reliable monitoring**.

---

# Design Philosophy

This roadmap follows one rule:

> **Stability first – features second**

Only extensions that do not compromise reliability will be considered.

The system architecture prioritizes:

- deterministic behavior
- simple debugging
- long-term stability

---

# Real-World Focus

All features are evaluated based on:

- usefulness in a camper environment
- long-term reliability
- simplicity of operation

If a feature adds complexity without clear benefit,
it will not be implemented.

---

_End of roadmap_
