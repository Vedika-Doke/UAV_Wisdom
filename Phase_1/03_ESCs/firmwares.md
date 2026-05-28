# ESC Firmwares

| Firmware | MCU family | License | Notes |
|----------|-----------|---------|-------|
| BLHeli_S | EFM8 (8-bit) | Closed | Cheap ESCs, DShot up to 600 |
| BLHeli_32 | ARM Cortex-M | Closed (paid) | High-end FPV, DShot 1200, telemetry, RPM filter |
| AM32 | ARM Cortex-M | **Open source** | Drop-in BLHeli_32 replacement, actively developed |
| Bluejay | EFM8 | Open source | BLHeli_S boards + bidirectional DShot |

## Why this matters for design
- AM32 lets us pick our own MCU (e.g., STM32G071) and ship a hackable ESC.
- BLHeli_32 is closed — using it on a custom board has licensing implications.
- Bidirectional DShot enables **RPM filtering** in Betaflight, which dramatically improves flight performance.

## Sources
1. https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware
2. https://github.com/bitdump/BLHeli
