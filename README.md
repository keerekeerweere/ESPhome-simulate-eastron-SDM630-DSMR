Projektbeschreibung / Project Description
English:
This project emulates an Eastron SDM630 energy meter over RS485/Modbus using ESPHome. The primary goal is to expose DSMR/P1 meter values (via Home Assistant) as if they came from a real SDM630. The Shelly 3EM variant is still kept for compatibility with the original GitHub project.

Key Features:

Emulates the Modbus registers of an SDM630.

Uses real measurement values (primarily DSMR/P1 via Home Assistant; optionally Shelly 3EM for compatibility).

Perfect for integration into home automation or energy monitoring systems designed for the SDM630.
# Original Github-Projekt: https://github.com/hankipanky/esphome-fake-eastron-SDM630
# Shelly-kompatibler Fork / Shelly-compatible fork: https://github.com/Feierprinz/ESPhome-fake-eastron-SDM630-Shelly-3EM
# Original Yaml: https://github.com/hankipanky/esphome-fake-eastron-SDM630/blob/master/fake-eastron.yaml
# Original Modbus - Dateien: https://github.com/hankipanky/esphome-fake-eastron-SDM630/blob/master/esphome/components/modbus_server/modbus_server.h und 
# https://github.com/hankipanky/esphome-fake-eastron-SDM630/blob/master/esphome/components/modbus_server/modbus_server.cpp
# Original Github basiert auf ESPHome 2024.5 und einem Shelly 3EM Pro, dieses Projekt hier auf ESPHome 2025.3.3 und einem Shelly 3EM
# Folgender ESP wird verwendet: https://amzn.to/4coLhZm
# Folgender RS485-Adapter: https://amzn.to/3R5zpBN
##################################
## Goal

This project allows you to provide live meter data from a DSMR/P1 setup (via Home Assistant) to a charger/inverter that expects an [Eastron SDM630](https://www.eastroneurope.com/products/view/sdm630modbus) over Modbus RTU.

The original Shelly-based flow is still included to stay compatible with the upstream project and hardware setup.

## How is this working?

The project uses an ESP32 with an RS485 adapter to emulate an Eastron SDM630 over Modbus RTU for a Technivolt 101 EV charger (Vestel-based behavior is expected). The charger polls the ESP32 as if it were a supported MID meter, while the ESP32 maps external data sources into the SDM630 register layout.

### How exactly?

* On boot, ESPHome starts a Modbus RTU slave (typically SDM630 v2 on address `2`) and exposes SDM630-compatible input registers.
* The data source can be DSMR/P1 values via Home Assistant (primary target) or the Shelly-based upstream-compatible flow.
* ESPHome converts values to IEEE-754 float and writes them to the matching SDM630 input registers (especially per-phase currents).
* The Technivolt charger polls these registers and uses them to determine charging behavior / load management.
* Request logging is enabled in the DSMR config so unexpected polling patterns can be captured and analyzed.

### Config variants

* `config/esphome/fake-eastron.yaml` = Shelly-based variant (kept for upstream compatibility)
* `config/esphome/simulated-eastron-dsmr-ha.yaml` = DSMR/P1 via Home Assistant variant (current primary goal)

## Why?

The goal is to connect to a Technivolt 101 EV charger and influence charging behavior through Modbus RTU readings provided by an ESP32 + RS485 adapter.

This makes it possible to shape the values seen by the charger so charging power follows a desired strategy, instead of being tied only to instantaneous real household usage.

Typical use cases:

* Cheap night tariffs: charge up to a chosen capacity during low-cost periods.
* Dynamic tariffs: charge mainly when energy prices are low.
* Flanders capacity tariff: limit peaks most of the time, but allow higher charging power when conditions are favorable.
* Solar surplus: use available PV power for EV charging to avoid exporting to the grid at poor (sometimes negative) feed-in rates.

In short, this project is a protocol bridge and control point: it presents SDM630-compatible meter values to the charger while allowing those values to be derived from DSMR/Home Assistant logic and tariff/solar-aware strategies.

## Installation

1. Connect ESP32 dev board to RS485 module.
2. Connect the RS485 module `A`/`B` lines to the Technivolt 101 EV charger RS485 `A`/`B` connection points exactly as described in the Technivolt installation documentation.
3. For the load-management simulation use case, follow the Technivolt manual instructions for the relevant wiring/configuration in `docs/MA_EN_TECHNIVOLT_100+101_D100001458632.pdf`, especially section `10.2.3`.
4. Build and flash the DSMR/Home Assistant firmware: [`config/esphome/simulated-eastron-dsmr-ha.yaml`](./config/esphome/simulated-eastron-dsmr-ha.yaml).
5. Update the Home Assistant DSMR `entity_id` values in the YAML to match your installation.
6. Configure the Technivolt charger for external meter/load-management operation according to the manual (section `10.2.3` for this simulation scenario).
7. Power-cycle the charger if required by the charger setup procedure.

For the legacy Growatt/Shelly-compatible setup, see [`installation-growatt.md`](./installation-growatt.md).

### Modbus address

Many chargers/controllers expect specific meter types at specific slave addresses. For SDM-compatible setups, the commonly used addresses are:

| Meter | Phases | Slave address |
|---------|----------|-----------------|
| Eastron SDM230 | 1 | 1 |
| Eastron SDM630 v2 | 3 | 2 |
| Eastron SDM630 v3 | 3 | 3 |

For the Technivolt/Vestel-style simulation case in this repository, start with **SDM630 v2 / slave address `2`** unless your charger configuration/documentation explicitly expects a different address.

Always match the ESPHome `modbus_slave_id` to the charger’s configured meter type/address.

## External documentation & tools

* [Eastron SDM630 Modbus Protocol](docs/SDM630-Modbus_Protocol.pdf)
* [Technivolt 100/101 Installation Manual](docs/MA_EN_TECHNIVOLT_100+101_D100001458632.pdf) (see section `10.2.3` for load-management simulation wiring/config)
* [Shelly Pro 3 EM](https://shelly-api-docs.shelly.cloud/gen2/Devices/Gen2/ShellyPro3EM)
* [Growatt Modbus RTU Protocol](docs/Growatt-Inverter-Modbus-RTU-Protocol-II-V1-24-English-new.pdf) (legacy/compatibility reference)
* [ESP32 NodeMCU pinout](docs/ESP-32_NodeMCU_Developmentboard_Pinout.pdf)
* [IEEE-754 Floating Point Converter](https://www.h-schmidt.net/FloatConverter/IEEE754.html)
* [Online Modbus Parse](https://rapidscada.net/modbus/)
* Python tool to query an Eastron; for testing/verification: [https://github.com/nmakel/sdm_modbus](https://github.com/nmakel/sdm_modbus)

## Some thoughts

* I want to have as little elements involved in this as possible. Other projects use an MQTT broker between smart meter and ESP, which could become unavailable. Therefore I use direct communication between the two.
* Shelly always reports the power factor as a positive value. Eastron reports it as negative when exporting power. Therefore I'm calculating the powerfactor myself, rather than using the values provided by the Shelly.
* I would prefer to use Modbus TCP to query the Shelly. Not sure if ESPhome supports this.
* The inverter is constantly adjusting the charge/discharge rate of the battery, trying to avoid importing power. My observations show that the inverter "overshoots" by ~50W to achieve this. As a result, the installation is constantly exporting a little bit of power to the grid. In order to avoid this, I'm adjusting the reported values for active and aparent power by 20W per phase. This balances the import/export at around -5..5W.
