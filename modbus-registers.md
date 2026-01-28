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

---

_End of document_
