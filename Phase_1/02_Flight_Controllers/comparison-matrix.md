# Flight Controller — Comparison Matrix

CSV mirror: [`../matrices/flight_controllers.csv`](../matrices/flight_controllers.csv)

| FC | Class | MCU | IMU(s) | Baro | Mag | Firmware | Weight (g) | Price (USD) | CAN | Redundancy | Target use |
|----|------|-----|--------|------|-----|----------|-----------:|------------:|:---:|------------|------------|
| Holybro Pixhawk 6X | Commercial | STM32H753 | 3× | 2× | ext | PX4 / Ardupilot | ~35 | ~280 | ✅ | Triple IMU, isolated rails | Industrial / autonomous |
| Holybro Pixhawk 6C | Commercial | STM32H743 | 2× | 1× | ext | PX4 / Ardupilot | ~35 | ~180 | ✅ | Dual IMU | Mid-tier commercial |
| CUAV V6X | Commercial | STM32H753 | 3× | 2× | ext | PX4 / Ardupilot | | | ✅ | Triple IMU | Industrial |
| Cube Orange+ | Commercial | STM32H757 | 3× | 2× | int | Ardupilot / PX4 | ~73 | ~400 | ✅ | Triple IMU + isolation | Industrial / heavy |
| Matek H743-WING | Mid | STM32H743 | 1× | 1× | ext | INAV / Ardupilot | ~10 | ~80 | — | None | Fixed-wing / VTOL hobby |
| SpeedyBee F405 V4 | FPV | STM32F405 | 1× | ✅ | — | Betaflight | ~9 | ~40 | — | None | FPV freestyle |
| Holybro Kakute H7 | FPV | STM32H743 | 1× | ✅ | — | Betaflight / INAV | ~9 | ~55 | — | None | High-end FPV |

> *Verify every cell before relying on it — vendor specs change. Mark unverified rows.*

## Sources
<!-- 1. Holybro product pages — https://holybro.com/ -->
<!-- 2. CUAV — https://www.cuav.net/ -->
<!-- 3. ProfiCNC / CubePilot — https://www.cubepilot.com/ -->
