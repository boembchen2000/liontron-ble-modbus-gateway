# liontron-ble-modbus-gateway

Stable BLE → Modbus RTU gateway for Liontron LiFePO₄ batteries  
(ESP32 + Raspberry Pi, JBD BMS).

Reliable integration of Liontron batteries into wired HMI systems
for camper / motorhome environments.

---

# Overview

This project provides a **stable gateway to read Liontron LiFePO₄ batteries**
via Bluetooth Low Energy (BLE) and expose the data as **Modbus RTU registers**
for industrial HMIs and automation systems.

Liontron batteries only provide **BLE access through the BMS**, which makes
reliable integration into wired systems difficult.  
This project solves that limitation by using a **two-stage architecture**:

* **ESP32** handles all BLE communication with the battery
* **Raspberry Pi** converts the data into a deterministic Modbus RTU interface

The system is designed for **24/7 operation in a motorhome environment**.

---

# Features

- Reads **two Liontron batteries** with JBD BMS via BLE
- Robust BLE handling using **ESP32 + NimBLE**
- JSON data stream over serial
- Raspberry Pi **Modbus RTU server**
- Designed for **HMI panels (e.g. Mochuan)**
- Clear **status bits and alarm logic**
- Long-term **connection stability**
- Designed for **real-world camper installations**

---

# System Architecture
