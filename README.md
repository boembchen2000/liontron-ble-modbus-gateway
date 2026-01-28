# liontron-ble-modbus-gateway
Stable BLE-to-Modbus gateway for Liontron LiFePO₄ batteries (ESP32 + Raspberry Pi, JBD BMS).

# Liontron BLE Modbus Gateway

This project provides a stable gateway to read Liontron LiFePO₄ batteries
using an ESP32 via BLE and expose the data as Modbus RTU registers
through a Raspberry Pi.

It was developed for use in a camper / motorhome environment
where reliable wired communication to an HMI is required,
but the battery BMS only offers Bluetooth Low Energy.

## Features

- Reads two Liontron batteries with JBD BMS via BLE
- Robust BLE handling using ESP32 + NimBLE
- JSON data stream over serial
- Raspberry Pi Modbus RTU server
- Designed for HMI panels (e.g. Mochuan)
- Clear status bits and alarm logic

## Hardware Overview

- ESP32 (BLE client)
- Raspberry Pi
- RS485 adapter
- Liontron LiFePO₄ batteries (JBD BMS)
- Modbus RTU HMI

## Project Status

This project is in active use and considered **stable**.
Further extensions are planned.

More detailed documentation will follow.
