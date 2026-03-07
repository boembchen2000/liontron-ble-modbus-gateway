# BLE Pitfalls with Liontron (JBD) Batteries

This document describes the **real-world problems and pitfalls** encountered
when working with Liontron LiFePO₄ batteries via Bluetooth Low Energy (BLE),
and why a naïve implementation usually fails.

These issues were discovered through extensive testing in a camper / motorhome
environment and are **not obvious from datasheets or libraries**.

---

# 1. BLE Is Not a Continuous Data Stream

Liontron batteries expose their BMS via BLE primarily for **mobile apps**.

This implies:

- Short-lived connections
- Burst-style communication
- No guarantee of continuous notifications
- No guarantee that notifications resume after reconnect

**Observed behavior:**

- BLE connection stays “connected”
- Notifications silently stop
- No error is raised
- Only a reconnect restores data

➡️ A system must **actively supervise data flow**, not just connection state.

---

# 2. Notifications Can Stall Without Disconnect

A critical pitfall:

The ESP32 can remain connected, but receive **no more notifications**.

This happens especially when:

- Commands are sent too fast
- BLE timing jitter occurs
- The battery enters a low-power internal state

**Important:**  
There is no BLE error and no disconnect callback.

**Mitigation used in this project:**

- Strict command timing (fixed intervals)
- Alternating command pattern (Basic ↔ Cell)
- External timeout detection on the Raspberry Pi

---

# 3. BLE Timing Is Extremely Sensitive

Liontron / JBD BMS expects:

- Clean request → response cycles
- No flooding
- No overlapping commands

Common mistakes:

- Sending commands in a tight loop
- Polling too fast “because it works sometimes”
- Assuming BLE behaves like UART

**Result:**

- Partial frames
- Stalled notifications
- Undefined behavior

➡️ BLE must be treated as a **timing-critical protocol**, not a stream.

---

# 4. ESP32 + Modbus Is a Trap

Many attempts try to:

“Just do everything on the ESP32”

This usually fails because:

- BLE needs precise timing
- Modbus RTU needs deterministic timing
- Serial + RS485 interrupts interfere with BLE
- Watchdog resets become hard to debug

**Observed outcomes:**

- Random disconnects
- Missed Modbus frames
- Hard-to-reproduce bugs
- System instability after hours or days

➡️ This project **intentionally forbids Modbus on the ESP32**.

---

# 5. Dual Battery BLE Is Not Trivial

Handling two batteries over BLE introduces additional pitfalls:

- Two independent connection states
- Different response timings
- One battery can stall while the other works
- Reconnect logic must be per-battery

**Design decision:**

- Each battery has its own BLE context
- No shared state
- No assumptions that both behave identically

---

# 6. Why JSON over Serial Works

Instead of pushing Modbus directly:

- ESP32 outputs **line-based JSON**
- Raspberry Pi acts as supervisor and gateway

Advantages:

- Human-readable debugging
- Easy logging
- Safe handling of partial frames
- Stateless transport
- Easy future extensions

This separation is the **key stability factor** of the system.

---

# 7. Timeout Detection Is Mandatory

Never trust BLE connection state alone.

This project uses:

- Timestamped data updates
- Explicit timeout detection
- “Battery offline” state derived from data flow, not BLE status

➡️ If data stops, the battery is considered offline — even if BLE says “connected”.

---

# 8. Lessons Learned

**What does NOT work reliably:**

- ESP32-only solutions
- BLE + Modbus on the same MCU
- Aggressive polling
- Blind trust in BLE notifications

**What DOES work:**

- Strict role separation
- Conservative BLE timing
- External supervision
- Explicit state machines

---

# Final Note

This architecture was not designed on paper.

It was shaped by:

- Broken connections
- Silent failures
- Real-world vehicle usage
- Long-term stability testing

If your BLE solution “works most of the time” —  
it is already broken.

---

# Why Battery Temperature Was Intentionally Omitted

During development, battery temperature values were investigated in detail.

According to JBD protocol documentation, temperature sensors are typically encoded
as **Kelvin ×10**, located at higher byte offsets in the Basic Info (0x03) response.

Temperature calculation:

Temperature = (raw_value - 2731) / 10

In theory, Liontron batteries with JBD-compatible BMS should expose one or more
temperature sensors using this format.

---

# What Was Observed in Practice

When querying Liontron batteries via BLE, the following issues were consistently observed:

- The 0x03 Basic Info response often ends before the documented temperature offsets
- Bytes containing temperature data are frequently missing or zero-filled
- Different Liontron firmware versions return different frame lengths
- Temperature fields are not reliably populated, even while the battery is in use
- The Liontron mobile app may show temperatures that are not exposed via BLE

As a result, reading battery temperature via BLE produced values such as:

- Constant 0 °C
- Invalid negative temperatures
- Unchanging values regardless of load or environment

---

# Why This Is a Problem

Battery temperature is a **safety-relevant parameter**.

Displaying unreliable or guessed temperature values is worse than displaying none at all:

- False alarms may trigger unnecessary user actions
- Missing alarms may hide real thermal problems
- Inconsistent values reduce trust in the system

For a motorhome / camper environment, this risk is unacceptable.

---

# Design Decision

Battery temperature was intentionally omitted from:

- ESP32 JSON output
- Modbus registers
- HMI visualization

This was a conscious design decision, not a technical limitation.

---

# What Is Used Instead

Instead of temperature monitoring, the system relies on:

- Cell voltage delta as an early indicator for imbalance and stress
- Direct monitoring of BMS protection and warning bits
- Raspberry Pi CPU temperature monitoring to protect the gateway hardware

The system relies on the internal **BMS protections** for thermal safety.

---

# Conclusion

Although the JBD protocol technically supports temperature reporting,
Liontron batteries do **not expose this information reliably via BLE**.

For long-term stability and safety, battery temperature was excluded by design.

**Reliability and correctness were prioritized over completeness.**

---

_End of document_
