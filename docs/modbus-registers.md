# Modbus Register Map

This document describes the Modbus RTU register layout used by the
Liontron BLE Modbus Gateway.

All registers are **Holding Registers (Function Code 0x03)**.

---

## Addressing Scheme

- Battery 1 base offset: **0**
- Battery 2 base offset: **20**
- System / Total values: **39**
- Raspberry Pi status: **100**

All values are **unsigned 16-bit**, unless noted otherwise.

---

## Battery Registers (Bat1 / Bat2)

> The same layout applies to both batteries.  
> Bat2 addresses = Bat1 address + 20.

### Electrical Values

| Address | Name | Unit | Description |
|-------:|------|------|-------------|
| +0 | Voltage | V ×100 | Battery voltage |
| +1 | Current | A ×100 (signed) | Charge (+) / Discharge (–) |
| +2 | State of Charge | % | Calculated from Ah |
| +4 | Cell 1 Voltage | V ×1000 | |
| +5 | Cell 2 Voltage | V ×1000 | |
| +6 | Cell 3 Voltage | V ×1000 | |
| +7 | Cell 4 Voltage | V ×1000 | |
| +8 | Cell Delta | mV | Max – Min cell voltage |
| +9 | Online | 0 / 1 | Timeout-based connection status |

---

### Status Bits Register

**Register: +11**

This register contains multiple status flags combined into one bitmask.

| Bit | Value | Meaning |
|----:|------:|---------|
| 0 | 1 | Battery online |
| 1 | 2 | Charging allowed |
| 2 | 4 | Discharging allowed |
| 3 | 8 | Cell delta warning (>120 mV) |
| 4 | 16 | Cell delta alarm (>200 mV) |
| 5 | 32 | BMS warning bits present |
| 6 | 64 | SOC < 20 % |
| 7 | 128 | SOC < 10 % (critical) |
| 8–15 | — | Reserved |

**Example values:**

- `7` → Online + Charge allowed + Discharge allowed  
- `71` → Online + Charge + Discharge + SOC < 20 %

---

### Capacity & Cycles

| Address | Name | Unit |
|-------:|------|------|
| +12 | Nominal Capacity | Ah |
| +13 | Remaining Capacity | Ah |
| +14 | Cycle Count | — |

---

### Overall Battery State (Simple HMI Logic)

**Register: +15**

This register is designed for **simple text states in HMI panels**.

| Value | Meaning |
|-----:|---------|
| 0 | OK |
| 1 | Warning |
| 2 | Alarm / Critical |

**Logic:**
- `Warning` → Cell delta >120 mV OR SOC <20 %
- `Alarm` → Cell delta >200 mV OR SOC <10 % OR BMS warning OR timeout

---

### Battery Identifier (BLE Name Split)

| Address | Description |
|-------:|-------------|
| +16 | ID Part 1 (first digits) |
| +17 | ID Part 2 (middle digits) |
| +18 | ID Part 3 (last digit) |

Used to reconstruct the numeric BLE name in the HMI.

---

## System / Total Registers

| Address | Name | Unit |
|-------:|------|------|
| 39 | Average Voltage | V ×100 |
| 40 | Total Current | A ×100 |
| 41 | Average SOC | % |
| 44 | Total Nominal Capacity | Ah |
| 45 | Total Remaining Capacity | Ah |

---

## Raspberry Pi Status

| Address | Name | Unit |
|-------:|------|------|
| 100 | CPU Temperature | °C ×10 |
| 101 | Overtemperature | 0 / 1 |

---

## Notes for HMI Configuration (Mochuan)

- Use **Register 11** with bit conditions for detailed texts  
  (e.g. “Charging allowed”, “Discharging not allowed”)
- Use **Register 15** for simple traffic-light or text states  
  (“All OK”, “Warning”, “Alarm”)
- Do **not** mix register 11 and 15 logic in the same text element

---

## Design Rationale

- Register 11 = **raw technical status bits**
- Register 15 = **human-readable simplified state**
- This avoids complex bitmask logic inside the HMI

# Modbus Register Map

This document describes the Modbus RTU register layout exposed by the
**Liontron BLE Modbus Gateway**.

The register map is designed for:
- Industrial HMIs (e.g. Mochuan)
- Simple numeric comparisons
- Minimal logic on the HMI side
- Long-term stability

All values are **preprocessed** by the Raspberry Pi.

---

## General Notes

- Modbus function code: **Holding Registers (4x)**
- Data type: **uint16**
- Signed currents are encoded as **int16 wrapped into uint16**
- Scaling is documented per register
- All addresses below are **zero-based offsets**

---

## Battery 1 Registers (Base Offset: `0`)

| Addr | Name | Description | Unit / Scaling |
|-----:|------|-------------|----------------|
| 0 | Bat1 Voltage | Battery voltage | V × 100 |
| 1 | Bat1 Current | Battery current | A × 100 (signed) |
| 2 | Bat1 SOC | State of charge | % |
| 4 | Cell 1 Voltage | Cell 1 | V × 1000 |
| 5 | Cell 2 Voltage | Cell 2 | V × 1000 |
| 6 | Cell 3 Voltage | Cell 3 | V × 1000 |
| 7 | Cell 4 Voltage | Cell 4 | V × 1000 |
| 8 | Cell Delta | Max cell difference | mV |
| 9 | Battery Online | 1 = online, 0 = timeout | — |
| 11 | **Status Bits** | Bitfield (see below) | — |
| 12 | Nominal Capacity | Nominal capacity | Ah |
| 13 | Remaining Capacity | Remaining capacity | Ah |
| 14 | Cycle Count | Charge cycles | — |
| 15 | **Overall State** | 0=OK, 1=Warn, 2=Alarm | — |
| 16 | ID Part 1 | Battery ID (5-5-1 split) | — |
| 17 | ID Part 2 | Battery ID (5-5-1 split) | — |
| 18 | ID Part 3 | Battery ID (5-5-1 split) | — |

---

## Battery 2 Registers (Base Offset: `20`)

Battery 2 uses **the exact same layout** as Battery 1.

| Addr | Name |
|-----:|------|
| 20 | Voltage |
| 21 | Current |
| 22 | SOC |
| 24–27 | Cell Voltages |
| 28 | Cell Delta |
| 29 | Battery Online |
| 31 | **Status Bits** |
| 32 | Nominal Capacity |
| 33 | Remaining Capacity |
| 34 | Cycle Count |
| 35 | **Overall State** |
| 36–38 | Battery ID Parts |

---

## Overall / System Registers

| Addr | Name | Description |
|-----:|------|-------------|
| 39 | Average Voltage | Mean of both batteries |
| 40 | Total Current | Sum of both currents |
| 41 | Average SOC | Mean SOC |
| 44 | Total Nominal Capacity | Sum Ah |
| 45 | Total Remaining Capacity | Sum Ah |

---

## Raspberry Pi System Registers

| Addr | Name | Description | Scaling |
|-----:|------|-------------|---------|
| 100 | Pi Temperature | CPU temperature | °C × 10 |
| 101 | Pi Temp Alarm | 1 = critical | — |

---

## Status Bitfield (Register 11 / 31)

Each battery provides a **single status word**.

### Bit Assignment

| Bit | Mask | Meaning |
|----:|-----:|---------|
| 0 | 1 | Battery online |
| 1 | 2 | Charging allowed |
| 2 | 4 | Discharging allowed |
| 3 | 8 | Cell delta warning (> 120 mV) |
| 4 | 16 | Cell delta alarm (> 200 mV) |
| 5 | 32 | BMS warning flag |
| 6 | 64 | SOC < 20 % |
| 7 | 128 | SOC < 10 % |
| 8–15 | — | Reserved |

➡️ HMI should use **bit checks**, not numeric comparisons.

---

## Overall State Register (15 / 35)

This register summarizes battery health.

| Value | Meaning |
|------:|--------|
| 0 | OK |
| 1 | Warning |
| 2 | Alarm |

### Logic (simplified)

- **Alarm**:
  - Offline
  - Cell delta > 200 mV
  - SOC < 10 %
  - BMS warning
- **Warning**:
  - Cell delta > 120 mV
  - SOC < 20 %
- **OK**:
  - Everything else

➡️ Ideal for a single “All OK / Warning / Alarm” text on the HMI.

---

## HMI Design Recommendation

- Use **Status Bits** for detailed icons/texts
- Use **Overall State** for summary messages
- Avoid calculations on the HMI
- Prefer comparisons (`==`, `>`, bit tests)

---

_End of document_

