# Modbus Register Map

This document describes the Modbus RTU register layout exposed by the  
**Liontron BLE Modbus Gateway**.

The register map is designed for:

- Industrial HMIs (e.g. Mochuan)
- Simple numeric comparisons
- Minimal logic inside the HMI
- Long-term stable operation

All values are **preprocessed by the Raspberry Pi**.

---

# General Notes

- Modbus function: **Holding Registers (Function Code 0x03)**
- Data type: **uint16**
- Signed currents are encoded as **int16 wrapped into uint16**
- Scaling is documented per register
- Addresses are **zero-based offsets**

---

# Addressing Scheme

| Component | Base Address |
|----------|--------------|
| Battery 1 | 0 |
| Battery 2 | 20 |
| System / Totals | 39 |
| Raspberry Pi Status | 100 |

Battery 2 uses **exactly the same layout** as Battery 1.

---

# Battery Registers (Bat1 / Bat2)

> Battery 2 addresses = Battery 1 address + 20

---

# Electrical Values

| Address | Name | Description | Unit / Scaling |
|-------:|------|-------------|----------------|
| +0 | Voltage | Battery voltage | V ×100 |
| +1 | Current | Charge (+) / Discharge (–) | A ×100 (signed) |
| +2 | State of Charge | SOC from BMS | % |
| +4 | Cell 1 Voltage | Cell 1 | V ×1000 |
| +5 | Cell 2 Voltage | Cell 2 | V ×1000 |
| +6 | Cell 3 Voltage | Cell 3 | V ×1000 |
| +7 | Cell 4 Voltage | Cell 4 | V ×1000 |
| +8 | Cell Delta | Max cell difference | mV |
| +9 | Battery Online | Timeout-based state | 0 / 1 |

---

# Status Bits Register

Register: **+11**

This register contains a **bitfield describing battery state**.

| Bit | Mask | Meaning |
|----:|-----:|---------|
| 0 | 1 | Battery online |
| 1 | 2 | Charging allowed |
| 2 | 4 | Discharging allowed |
| 3 | 8 | Cell delta warning (>120 mV) |
| 4 | 16 | Cell delta alarm (>200 mV) |
| 5 | 32 | BMS warning flag |
| 6 | 64 | SOC < 20 % |
| 7 | 128 | SOC < 10 % |
| 8–15 | — | Reserved |

### Example Values

| Value | Meaning |
|-----:|--------|
| 7 | Online + Charge allowed + Discharge allowed |
| 71 | Online + Charge + Discharge + SOC <20% |

HMI panels should use **bit tests**, not numeric comparisons.

---

# Capacity & Cycles

| Address | Name | Description | Unit |
|-------:|------|-------------|------|
| +12 | Nominal Capacity | Battery rated capacity | Ah |
| +13 | Remaining Capacity | Remaining capacity | Ah |
| +14 | Cycle Count | Charge cycles | — |

---

# Overall Battery State (Simplified HMI State)

Register: **+15**

This register summarizes the battery condition for simple HMI logic.

| Value | Meaning |
|-----:|--------|
| 0 | OK |
| 1 | Warning |
| 2 | Alarm |

### Simplified Logic

**Alarm**

- Battery offline
- Cell delta >200 mV
- SOC <10 %
- BMS warning flag

**Warning**

- Cell delta >120 mV
- SOC <20 %

**OK**

- No active conditions

This register allows HMIs to show a simple:

```
OK / Warning / Alarm
```

without complex bitmask logic.

---

# Battery Identifier (BLE Name Split)

Liontron batteries expose a **numeric BLE identifier**.

The gateway splits this ID for easier display on HMIs.

| Address | Description |
|-------:|-------------|
| +16 | ID Part 1 |
| +17 | ID Part 2 |
| +18 | ID Part 3 |

These registers can be combined to reconstruct the full BLE identifier.

---

# Battery 2 Registers

Battery 2 uses **the same layout**.

| Address | Description |
|-------:|-------------|
| 20–38 | Same structure as Battery 1 |

Example:

| Address | Description |
|-------:|-------------|
| 20 | Battery 2 Voltage |
| 21 | Battery 2 Current |
| 22 | Battery 2 SOC |
| 24–27 | Cell voltages |
| 28 | Cell delta |
| 29 | Battery online |
| 31 | Status bits |
| 32–34 | Capacity & cycles |
| 35 | Overall state |
| 36–38 | Battery ID |

---

# System / Total Registers

| Address | Name | Description | Unit |
|-------:|------|-------------|------|
| 39 | Average Voltage | Mean of both batteries | V ×100 |
| 40 | Total Current | Sum of both battery currents | A ×100 |
| 41 | Average SOC | Mean SOC | % |
| 44 | Total Nominal Capacity | Sum of both batteries | Ah |
| 45 | Total Remaining Capacity | Sum remaining capacity | Ah |

---

# Raspberry Pi System Registers

| Address | Name | Description | Unit |
|-------:|------|-------------|------|
| 100 | CPU Temperature | Raspberry Pi CPU temperature | °C ×10 |
| 101 | Overtemperature | Pi thermal alarm | 0 / 1 |

---

# HMI Design Recommendations

For stable HMI design:

Use **Status Bits (11 / 31)** for:

- Charging allowed
- Discharging allowed
- SOC warnings
- Cell imbalance warnings

Use **Overall State (15 / 35)** for:

- simple alarm texts
- traffic-light indicators
- summary status

Avoid complex calculations on the HMI.

---

# Design Philosophy

The register map follows a simple principle:

| Register | Purpose |
|--------|---------|
| Status Bits | Technical state information |
| Overall State | Human-readable battery condition |

This prevents complex logic inside the HMI and improves reliability.

---

_End of document_
