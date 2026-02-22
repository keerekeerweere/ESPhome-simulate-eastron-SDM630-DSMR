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

The project provides an ESPhome component acting as a Modbus RTU slave/server that can be polled by a master (e.g. charger or inverter). It behaves as much as possible like an Eastron SDM630 while mapping real meter data into the SDM630 register layout.

### How exactly?

* On boot, the ESPhome starts a Modbus slave on address 2. This is where Growatt expects to find the vanilla Eastron SDM630 Modbus V2. It registers several input registers that can be queried by a Modbus master.
* On the data side, ESPHome can read values either from Home Assistant (DSMR/P1 entities) or from the Shelly-based upstream flow.
* The values are converted to IEEE-754 float and written to matching SDM630 input registers.
* The charger/inverter queries the Modbus slave several times a second, fetching the input registers in various groups.

### Config variants

* `config/esphome/fake-eastron.yaml` = Shelly-based variant (kept for upstream compatibility)
* `config/esphome/simulated-eastron-dsmr-ha.yaml` = DSMR/P1 via Home Assistant variant (current primary goal)

## Why?

I use a Growatt MIN 4200TL-XH hybrid inverter. The inverter only supports two types of smart meters, which must be connected via RS485:

* Eastron SDM630 Modbus V2 or V3
* CHINT smart meter from Growatt

In my case the inverter is installed in a different room relatively far away from the grid connection. Having a two-wire RS485 connection running through the house is not an option. Also, I have a Shelly Pro 3EM smart meter installed. 

Since the Shelly is LAN-connected, this simple ESPhome project bridges manufacturer, distance, physical layer and protocol.

## Installation

1. Connect ESP32 dev board to RS485 module.
2. Connect RS485 A and B connectors to pins 5 (A) and 6 (B) of the Growatt SYS COM port.
3. Build and flash the firmware based on the chosen ESPHome config:
   * Shelly-compatible: [`config/esphome/fake-eastron.yaml`](./config/esphome/fake-eastron.yaml)
   * DSMR/Home Assistant: [`config/esphome/simulated-eastron-dsmr-ha.yaml`](./config/esphome/simulated-eastron-dsmr-ha.yaml)
4. Enable meter-reading in Growatt **TODO: explain how**
5. Power-cycle the inverter completely.

**TODO: fritzing or image from breadboard**

![Image](images/growatt-syscom.png)
![Image](images/IMG_9971.JPG)
![Image](images/IMG_9972.JPG)

### Modbus address

Growatt expects different smart meters at different slave addresses:

| Meter | Phases | Slave address |
|---------|----------|-----------------|
| Eastron SDM230 | 1 | 1 |
| Eastron SDM630 v2 | 3 | 2 |
| Eastron SDM630 v3 | 3 | 3 |

Eastron SDM630 **v3** is a custom version with firmware influenced by Growatt. My understanding is that it allows a higher rate of request/responses, resulting in finer tracking of power demands.

## External documentation & tools

* [Eastron SDM630 Modbus Protocol](docs/SDM630-Modbus_Protocol.pdf)
* [Shelly Pro 3 EM](https://shelly-api-docs.shelly.cloud/gen2/Devices/Gen2/ShellyPro3EM)
* [Growatt Modbus RTU Protocol](docs/Growatt-Inverter-Modbus-RTU-Protocol-II-V1-24-English-new.pdf)
* [ESP32 NodeMCU pinout](docs/ESP-32_NodeMCU_Developmentboard_Pinout.pdf)
* [IEEE-754 Floating Point Converter](https://www.h-schmidt.net/FloatConverter/IEEE754.html)
* [Online Modbus Parse](https://rapidscada.net/modbus/)
* Python tool to query an Eastron; for testing/verification: [https://github.com/nmakel/sdm_modbus](https://github.com/nmakel/sdm_modbus)

## Some thoughts

* I want to have as little elements involved in this as possible. Other projects use an MQTT broker between smart meter and ESP, which could become unavailable. Therefore I use direct communication between the two.
* Shelly always reports the power factor as a positive value. Eastron reports it as negative when exporting power. Therefore I'm calculating the powerfactor myself, rather than using the values provided by the Shelly.
* I would prefer to use Modbus TCP to query the Shelly. Not sure if ESPhome supports this.
* The inverter is constantly adjusting the charge/discharge rate of the battery, trying to avoid importing power. My observations show that the inverter "overshoots" by ~50W to achieve this. As a result, the installation is constantly exporting a little bit of power to the grid. In order to avoid this, I'm adjusting the reported values for active and aparent power by 20W per phase. This balances the import/export at around -5..5W.
