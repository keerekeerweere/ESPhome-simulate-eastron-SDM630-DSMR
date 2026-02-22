# Growatt Installation (Legacy / Compatibility)

This page keeps the original Growatt-oriented setup notes for the Shelly-compatible flow.

Use this path if you are integrating with a Growatt inverter that expects an SDM630-compatible meter over RS485/Modbus RTU.

## Firmware Variant

Use the Shelly-compatible config:

- [`config/esphome/fake-eastron.yaml`](./config/esphome/fake-eastron.yaml)

## Installation (Growatt)

1. Connect the ESP32 dev board to the RS485 module.
2. Connect RS485 `A` and `B` to the Growatt SYS COM port pins `5 (A)` and `6 (B)`.
3. Build and flash the firmware based on [`config/esphome/fake-eastron.yaml`](./config/esphome/fake-eastron.yaml).
4. Enable meter-reading in the Growatt inverter (device-specific menu/installer settings).
5. Power-cycle the inverter completely.

## Wiring / Photos

![Growatt SYS COM](images/growatt-syscom.png)
![Wiring Photo 1](images/IMG_9971.JPG)
![Wiring Photo 2](images/IMG_9972.JPG)

## Modbus Address (Growatt Smart Meter Selection)

Growatt expects different smart meters at different slave addresses:

| Meter | Phases | Slave address |
|---------|----------|-----------------|
| Eastron SDM230 | 1 | 1 |
| Eastron SDM630 v2 | 3 | 2 |
| Eastron SDM630 v3 | 3 | 3 |

In most SDM630 emulation cases, start with **SDM630 v2 / slave address `2`** unless your inverter is configured otherwise.

## References

- [`docs/Growatt-Inverter-Modbus-RTU-Protocol-II-V1-24-English-new.pdf`](./docs/Growatt-Inverter-Modbus-RTU-Protocol-II-V1-24-English-new.pdf)
- [`docs/SDM630-Modbus_Protocol.pdf`](./docs/SDM630-Modbus_Protocol.pdf)
