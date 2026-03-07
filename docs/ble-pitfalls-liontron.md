# BLE Pitfalls with Liontron / JBD BMS

This document describes the **real-world BLE problems** encountered when
communicating with Liontron LiFePO₄ batteries using an ESP32.

These issues are **not theoretical** – they were observed during long-term
operation in a camper / motorhome environment.

---

# 1. Liontron BLE Is Not Designed for Continuous Clients

Liontron batteries expose their JBD BMS via BLE primarily for **mobile apps**.

Key consequences:

- BLE service is optimized for **short, interactive sessions**
- Not intended for permanent 24/7 connections
- No official documentation from Liontron

**Result:**

A naive ESP32 BLE client will disconnect randomly or stop receiving data.

---

# 2. ESP32 BLE Stack Pitfalls

## Problem: Unstable Connections with Default BLE Stack

Using the default ESP32 BLE library (`BLEDevice`) caused:

- Random disconnects
- Missed notifications
- Complete lockups after several minutes or hours

**Solution:**

Switch to **NimBLE-Arduino**

Why NimBLE works better:

- Lower memory usage
- More deterministic behavior
- Better handling of reconnects
- Cleaner notification handling

---

# 3. Dual Battery BLE Is Especially Tricky

Running **two Liontron batteries in parallel** over BLE introduces additional problems:

- Simultaneous scanning and connecting is unreliable
- Connection timing matters
- Notifications may stop for one battery while the other still works

## What works

- Scan first → store addresses
- Connect batteries sequentially
- Maintain **separate BLE clients**
- Alternate command sending (CMD 0x03 / 0x04)

---

# 4. JBD Protocol Timing Sensitivity

The JBD BMS protocol is **timing-sensitive**.

Observed behavior:

- Sending commands too fast → no response
- Sending commands too slow → BLE connection times out
- Mixing commands without delays → corrupted frames

## Stable solution

Command interval ≈ **1000 ms**

Alternate commands:

CMD 0x03 → Basic Info  
CMD 0x04 → Cell Voltages

Never spam write requests.

---

# 5. Notifications Can Silently Stop

One of the most dangerous issues:

BLE connection stays **connected**, but notifications stop arriving.

No error. No disconnect event.

## Countermeasures used in this project

- Track `valid_basic` and `valid_cells`
- Use **timeout logic on the Raspberry Pi**
- Treat missing updates as *battery offline*

This is why **timeout-based online detection** is mandatory.

---

# 6. Why ESP32-Only Is a Bad Idea

It is technically possible to implement Modbus directly on the ESP32.

However, in practice:

- BLE reconnect logic becomes extremely complex
- Modbus timing conflicts with BLE timing
- Debugging becomes nearly impossible
- No persistent logging or recovery logic

**Splitting responsibilities solves this**

| Component | Responsibility |
|-----------|---------------|
| ESP32 | BLE + JBD protocol |
| Raspberry Pi | JSON parsing, logic, Modbus RTU |

This separation is a **key design decision** of this project.

---

# 7. Lessons Learned (Hard Truths)

- BLE ≠ reliable field bus
- BLE batteries behave like consumer devices
- Stability comes from **defensive design**
- Timeouts are not optional
- Status bits are essential
- HMI logic must be kept simple

---

# Conclusion

Getting stable BLE data from Liontron batteries requires:

- NimBLE
- Conservative timing
- Explicit reconnect logic
- Separation of concerns
- Acceptance that BLE will *never* be perfect

This project works **because it assumes failure and handles it**.

---

_End of document_
