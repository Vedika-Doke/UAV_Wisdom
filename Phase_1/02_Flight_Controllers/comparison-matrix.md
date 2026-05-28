# Flight Controller — Comparison Matrix

The headline table for Phase 1. Specs verified from vendor docs and PX4/ArduPilot product pages as of **May 2026** — re-check before any procurement decision.

CSV mirror: [`../matrices/flight_controllers.csv`](../matrices/flight_controllers.csv)

---

## A. Commercial / Pixhawk-class

Targets: photography/surveillance/load-carrying drones. PX4 or ArduPilot. Industrial-grade redundancy.

| FC | FMU std | MCU | Clock | RAM/Flash | IMUs | Baro | CAN | Ethernet | Weight (g) | Price (USD) | Notable |
|----|---------|-----|------:|-----------|------|------|:---:|:--------:|-----------:|------------:|---------|
| **Holybro Pixhawk 6X** | FMUv6X | STM32H753 | 480 MHz | 1 MB / 2 MB | **3×** (ICM-45686 + ICM-42688P + BMI088) | **2×** (BMP388 + ICM-20948) | ✅ 2× | ✅ | ~35 | ~280 | Modular IMU/FMU/Base, isolated domains |
| **Holybro Pixhawk 6C** | FMUv6C | STM32H743 | 480 MHz | 1 MB / 2 MB | 2× (ICM-42688P + BMI055) | 1× | ✅ 1× | ❌ | ~35 | ~180 | Cost-optimised single-board |
| **Holybro Pixhawk 6X-RT** | FMUv6X-RT | NXP iMX RT1176 | **1 GHz** + 400 MHz M4 | **2 MB / 64 MB** | 3× | 2× | ✅ | ✅ | ~35 | ~350 | Future-proof, big compute headroom |
| **CUAV Pixhawk V6X** | FMUv6X | STM32H753 | 480 MHz | 1 MB / 2 MB | 3× | 2× | ✅ 2× | ✅ | ~45 | ~270 | Strong industrial competitor to Holybro |
| **CubePilot Cube Orange+** | "Cube" std | STM32H757 | 480 MHz | 1 MB / 2 MB | 3× (2 vibe-isolated + 1 ref) | 2× | ✅ 2× | ❌ (via carrier) | ~73 (Cube + std carrier) | ~400 | Vibration-isolated IMUs, ADS-B option |
| Holybro Pix32 v6 | FMUv6X core | STM32H753 | 480 MHz | 1 MB / 2 MB | 3× | 2× | ✅ | ❌ | ~30 | ~200 | Naked FMU module, no carrier |
| mRo Control Zero H7 | (custom) | STM32H743 | 480 MHz | 1 MB / 2 MB | 3× | 1× | ✅ | ❌ | ~6 | ~250 | Tiny, mil-spec |

### Take-aways
- **Default industrial pick:** Holybro Pixhawk 6X — best support, modular, ~$280.
- **Best price/performance:** Holybro Pixhawk 6C — full H7 brain, dual IMU, ~$180.
- **Future-proof:** Pixhawk 6X-RT — iMX RT1176 has *real* compute for ML / vision pre-processing.
- **Most rugged:** Cube Orange+ — historical workhorse, dampened IMUs, vendor-tested in commercial fleets.

---

## B. Mid-tier (INAV / ArduPilot, fixed-wing & VTOL hobby)

| FC | MCU | Clock | IMU | Baro | Weight (g) | Price (USD) | Notable |
|----|-----|------:|-----|------|-----------:|------------:|---------|
| **Matek H743-WING V3** | STM32H743 | 480 MHz | dual ICM-42688P | DPS310 | ~12 | ~80 | INAV / ArduPilot Plane default |
| Matek F405-WING | STM32F405 | 168 MHz | ICM-42688P | DPS310 | ~10 | ~60 | Cheaper fixed-wing |
| SpeedyBee F405 WING APP | STM32F405 | 168 MHz | ICM-42688P | BMP280 | ~10 | ~55 | INAV friendly |
| Matek H743-MINI | STM32H743 | 480 MHz | dual ICM-42688P | DPS310 | ~6 | ~75 | Compact ArduPilot Copter |

### Take-aways
- For **fixed-wing/VTOL prototyping**: Matek H743-WING is the de-facto standard.
- For research projects on a budget: ArduPilot or INAV on a Matek H7 board punches *way* above its price.

---

## C. FPV / Hobbyist (Betaflight, INAV)

| FC | MCU | IMU | Mounting | Weight (g) | Price (USD) | Notable |
|----|-----|-----|----------|-----------:|------------:|---------|
| **SpeedyBee F405 V4** | STM32F405 | ICM-42688P | 30.5 | ~9 | ~40 | Default 5" budget FC |
| SpeedyBee F405 Mini V2 | STM32F405 | ICM-42688P | 20.5 | ~5 | ~38 | Same brain, micro form |
| **Holybro Kakute H7** | STM32H743 | dual ICM-42688P | 30.5 | ~9 | ~55 | High-end 5" |
| Holybro Kakute H7 Mini | STM32H743 | dual ICM-42688P | 20.5 | ~6 | ~50 | High-end micro |
| iFlight BLITZ Mini F7 | STM32F722 | ICM-42688P | 25.5 | ~6 | ~45 | Solid F7 mid-tier |
| HGLRC Zeus F722 | STM32F722 | ICM-42688P | 30.5 | ~9 | ~40 | Popular budget F7 |
| BetaFPV F4 1S 5-in-1 AIO | STM32F411 | ICM-42688P | 14×14 | ~2 | ~35 | Tinywhoop AIO |
| **SpeedyBee F405 Mini AIO 25A** | STM32F405 | ICM-42688P | 20.5 | ~7 | ~70 | **Direct reference for Phase 2 System 2** |

### Take-aways
- **ICM-42688-P** is universal. New designs in 2026 use this, full stop.
- **F405** is sunsetting — fine for budget, not for new designs.
- **H743** gives meaningful headroom — better filtering, future-firmware-proof.
- For **AIO reference**: SpeedyBee Mini AIO 25A and HGLRC Zeus AIO are the boards to dissect.

---

## D. Evaluation rubric for Phase 2 design

When defending design choices in the report/PPT, use these axes:

| Axis | Why it matters | Phase 2 System 1 target | Phase 2 System 2 target |
|------|---------------|-------------------------|-------------------------|
| **MCU class** | Loop rate, headroom | STM32H753 (FMUv6X) | STM32H743 or AT32F435 |
| **IMU strategy** | Reliability vs cost | Triple, 3 vendors | Single ICM-42688-P |
| **Sensor isolation** | Vibration / EMI immunity | Separate domain + dampers | Single PCB but isolated GND pour |
| **Power redundancy** | Single-point-failure tolerance | Dual VBAT diode-OR | Single, with TVS + bulk cap |
| **Bus support** | Smart peripherals | CAN-FD + Ethernet | UART-only (FPV stack) |
| **Mounting** | Vehicle fit | Std Pixhawk carrier | 20×20 |
| **Weight** | Vehicle payload budget | 35–50 g acceptable | **<8 g target** |
| **Cost (BOM)** | Productisation | ~$120 BOM → ~$280 retail | **~$15 BOM → ~$70 retail** |
| **Firmware target** | Ecosystem | PX4 + ArduPilot | Betaflight + Bluejay |

---

## Sources
1. Holybro Pixhawk 6X — https://holybro.com/products/pixhawk-6x and PX4 docs https://docs.px4.io/main/en/flight_controller/pixhawk6x.html
2. Holybro Pixhawk 6C — https://holybro.com/products/pixhawk-6c
3. Holybro Pixhawk 6X-RT announcement — https://dronecode.org/announcing-the-pixhawk-fmuv6x-rt/
4. CUAV V6X — https://docs.px4.io/main/en/flight_controller/cuav_pixhawk_v6x.html
5. CubePilot Cube Orange+ specifications — https://docs.cubepilot.org/user-guides/autopilot/the-cube/introduction/specifications
6. Matek H743-WING — https://www.mateksys.com/?portfolio=h743-wing
7. SpeedyBee F405 Mini AIO 25A — https://www.speedybee.com/
8. Holybro Kakute H7 — https://holybro.com/products/kakute-h7
9. Oscar Liang — *FC Processors Explained* — https://oscarliang.com/f1-f3-f4-flight-controller/
