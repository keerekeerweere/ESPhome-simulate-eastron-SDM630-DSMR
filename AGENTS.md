# Repository Guidelines

## Project Structure & Module Organization
This repository is an ESPHome-based SDM630 emulator for Shelly 3EM hardware.

- `fake-eastron.yaml`: main ESPHome device configuration (entry point for build/flash).
- `esphome/components/modbus_server/`: custom ESPHome component (`.cpp`, `.h`, `__init__.py`) implementing the Modbus server behavior.
- `docs/`: reference PDFs (protocols, pinouts).
- `images/`: wiring and installation photos.
- `secrets.yaml`: local credentials/secrets used by ESPHome (do not commit real secrets).

## Build, Test, and Development Commands
Use the ESPHome CLI from the repository root.

- `esphome config fake-eastron.yaml`: validate YAML and component configuration.
- `esphome compile fake-eastron.yaml`: build firmware only.
- `esphome run fake-eastron.yaml`: build, upload, and start logs (best default during development).
- `esphome logs fake-eastron.yaml`: monitor runtime logs from the device.

If you change the custom component, run `config` and `compile` before testing on hardware.

## Coding Style & Naming Conventions
- YAML: use 2-space indentation and descriptive `id` names (for example `modbusserver`, `last_valid_response`).
- C++ (`modbus_server.*`): follow existing ESPHome-style class names (`PascalCase`) and method/variable names (`snake_case`), keep includes grouped at the top.
- Python (`__init__.py`): follow ESPHome component patterns and `snake_case` constants/functions.
- Prefer small, targeted changes; keep protocol register mappings and callbacks readable.

## Testing Guidelines
There is no automated test suite in this repository yet. Minimum validation for changes:

- `esphome config fake-eastron.yaml`
- `esphome compile fake-eastron.yaml`
- Hardware smoke test against the inverter or a Modbus polling tool (verify expected slave address and register reads).

Document manual test results in PRs when changing register behavior or timing.

## Commit & Pull Request Guidelines
Git history is mostly short, file-focused messages (for example `Update fake-eastron.yaml`). Keep commits:

- Imperative and concise (`Update modbus_server register callback handling`)
- Scoped to one logical change when possible

PRs should include: purpose, affected files, test/validation steps run, and photos/log snippets if behavior changes on hardware.
