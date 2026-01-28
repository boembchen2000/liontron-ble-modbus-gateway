# Roadmap

This document outlines **possible future extensions** of the Liontron BLE Modbus Gateway.

The project is already **stable and in productive use**.  
All roadmap items are **optional** and driven by real-world needs, not feature pressure.

---

## Current Status

✅ Dual Liontron battery support  
✅ Stable BLE handling via ESP32  
✅ Deterministic Modbus RTU registers  
✅ Clear status bits and alarm logic  
✅ Proven operation with industrial HMI panels  

---

## Planned / Possible Extensions

### 🔹 1. Extended BMS Diagnostics

- Additional warning bit decoding
- More detailed BMS fault classification
- Optional exposure of raw BMS warning flags

Status: *planned, low priority*

---

### 🔹 2. Logging & Diagnostics (Raspberry Pi)

- Optional CSV or JSON logging
- Timestamped battery events
- Easy post-mortem analysis

Status: *optional*

---

### 🔹 3. MQTT / Home Assistant Bridge

- Publish processed battery data via MQTT
- Keep Modbus logic untouched
- Optional HA integration

Status: *future idea*

---

### 🔹 4. Configuration File Support

- Battery names
- Register offsets
- Threshold values (SOC, delta mV)

Status: *nice-to-have*

---

## Explicit Non-Goals

The following items are **intentionally not planned**:

- ❌ ESP32-only Modbus implementation  
- ❌ BLE reconnection hacks inside HMI  
- ❌ Complex logic inside the HMI  
- ❌ Battery write/configuration commands  

This project focuses on **read-only, reliable monitoring**.

---

## Philosophy

This roadmap follows one rule:

> **Stability first – features second**

Only extensions that do not compromise reliability will be considered.

---

_End of roadmap_
