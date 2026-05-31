# FPV / Hobbyist Flight Controllers

These are the **cheap, light, fast** FCs that run Betaflight and INAV. Different design philosophy from Pixhawk:

| Aspect | Pixhawk (FMU) | FPV FC |
|--------|---------------|--------|
| Standardisation | Rigid open standard | Loose conventions (mounting hole pattern) |
| Sensors | Triple-redundant IMU mandatory | **Single IMU** typical |
| RTOS | NuttX / ChibiOS | **None — bare-metal main loop** |
| Loop rate | 1–2 kHz (PX4 control) | **4–8 kHz PID, 8–32 kHz gyro** |
| Cost | $150–500 | **$30–80** |
| Weight | 35–80 g | **5–15 g** |
| Autonomy | Full GPS missions | Minimal (GPS Rescue only) |
| Vehicles | Multi, fixed-wing, VTOL, rover | Multirotor only (Betaflight); +fixed-wing (INAV) |

Both philosophies are valid — they target different problems. For **Phase 2 System 2 (Lightweight AIO)** the FPV philosophy applies.

---

## 1. MCU families — the single most important spec

Naming convention: `STM32 [family][series][line]` — e.g. STM32F405.

| Family | Cortex | Clock | Flash / RAM | Where you find it |
|--------|--------|------:|-------------|-------------------|
| **F4 (F411)** | M4 @ FPU | 100 MHz | 512 KB / 128 KB | Tinywhoops, micro AIOs |
| **F4 (F405)** | M4 @ FPU | 168 MHz | 1 MB / 192 KB | "Default" budget FC since 2017 |
| **F7 (F722)** | M7 @ FPU | 216 MHz | 512 KB / 256 KB | Mid-tier — handles 8 kHz PID + ArduPilot |
| **F7 (F745)** | M7 | 216 MHz | 1 MB / 320 KB | High-end F7 |
| **H7 (H743)** | M7 + DP-FPU | **480 MHz** | **2 MB / 1 MB** | Current high-end FPV / FMUv6C |
| **H7 (H753)** | M7 + crypto | 480 MHz | 2 MB / 1 MB | FMUv6X commercial |
| AT32F435 | M4 | 288 MHz | varies | Cheap H7-ish alternative (cloned by Artery) |

### Why MCU matters in practice
- **Loop rate ceiling** — Betaflight runs gyro sampling at 8–32 kHz; the FC must keep up.
- **UART count** — every peripheral (RX, VTX, GPS, ESC telemetry) eats a UART. F411 has 3; H743 has 8.
- **Flash size** — Betaflight 4.5+ with all features doesn't fit on F411 anymore; F405 is at its limit.
- **Hardware UART inversion** — F4 lacked it (needed external inverter chip for SBUS); F7/H7 have it native.

> **For Phase 2 AIO design:** target **STM32H743** or **AT32F435** (cheaper). H743 gives plenty of headroom and is futureproof; F405 is sunsetting.

---

## 2. IMU — the second most important spec

The IMU is read thousands of times per second; its noise floor and bandwidth set the achievable PID performance.

| IMU | Vendor | Bus | Max ODR | Status (2026) |
|-----|--------|-----|---------|---------------|
| MPU6000 | InvenSense | SPI | 8 kHz | EOL — avoid for new designs |
| ICM-20689 | InvenSense | SPI | 8 kHz | Legacy |
| ICM-20602 | InvenSense | SPI | 32 kHz | Legacy, was popular |
| BMI270 | Bosch | SPI | 6.4 kHz | Cheap but lower performance; **Betaflight has advised against** |
| **ICM-42688-P** | InvenSense | SPI | **32 kHz** | **The current default — best perf/price** |
| ICM-45686 | InvenSense | SPI | 32 kHz | Pixhawk-class, dual-FIFO, premium |
| BMI088 | Bosch | SPI | 1.6 kHz gyro | Used as redundancy partner in Pixhawk 6X |

**For Phase 2 design:** **ICM-42688-P** on SPI is the default choice. For Pixhawk-grade redundancy, pair it with **ICM-45686 + BMI088**.

---

## 3. Other onboard chips on a typical FPV FC

| Chip | Function | Examples |
|------|----------|----------|
| **Barometer** | Altitude estimate (with EKF/baro hold) | BMP388, DPS310, BMP280 |
| **OSD chip** | Analog video overlay | AT7456E (MAX7456 clone, default in Betaflight) |
| **Blackbox flash** | Log gyro/PID for tuning analysis | W25Q128 (16 MB SPI NOR) typical |
| **PDB / BEC** | 5 V, 9 V regulators | Often onboard a "PDB-style" FC or AIO |
| **SD card slot** | Higher-capacity blackbox (rare on small FCs) | — |

---

## 4. Mounting standards

| Pattern (mm) | Class | Frame typical |
|--------------|-------|---------------|
| 16 × 16 | Tinywhoop / sub-2" AIOs | |
| **20 × 20** | Toothpick / 3" / cinewhoop | Most micros |
| 25.5 × 25.5 | Mid-size | |
| **30.5 × 30.5** | **5" freestyle / racing** | **The default** |

For **Phase 2 System 2 (Lightweight AIO)**, you'd pick a mounting standard at the start of layout — likely **20 × 20** for the lightweight target.

---

## 5. Common FCs to study (their datasheets and schematics are educational)

| Board | MCU | IMU | Mounting | Notable |
|-------|-----|-----|----------|---------|
| **SpeedyBee F405 V4** | F405 | ICM-42688-P | 30.5 | Cheap reference design |
| SpeedyBee F405 Mini V2 | F405 | ICM-42688-P | 20.5 | Same chip, smaller form |
| **Holybro Kakute H7** | H743 | dual ICM-42688-P | 30.5 | Top-tier 5", premium build |
| Holybro Kakute H7 Mini | H743 | dual ICM-42688-P | 20.5 | Same brain, smaller |
| iFlight BLITZ Mini F7 | F722 | ICM-42688-P | 25.5 | Solid F7 mid-tier |
| Matek H743-WING | H743 | dual ICM-42688-P | (proprietary) | Fixed-wing / VTOL-focused, INAV-default |
| **AIO: SpeedyBee F405 Mini AIO 25A** | F405 + 25A ESC | ICM-42688-P | 20.5 | **Reference AIO design — exact category of Phase 2 System 2** |
| AIO: HGLRC F722 AIO | F722 + ESC | ICM-42688-P | 20.5 | Another reference AIO |

Open the **product page → "Specifications"** and the **schematic PDF** for any of these — they're treasure troves for understanding what an actual layout looks like.

---

## 6. Comparison axes that matter for FPV FCs

When choosing or designing one:

1. **MCU class** — sets ceiling on features, UART count, loop rate.
2. **IMU model** — sets noise floor / max gyro rate.
3. **Mounting standard** — matches the frame.
4. **UART count and inversion support** — for RX, VTX, ESC telem, GPS.
5. **BEC ratings** — 5 V/3 A and 9 V/2 A typical for VTX power.
6. **Stack vs AIO** — if stack, what ESC current rating is paired.
7. **Blackbox** — onboard flash size or SD slot.
8. **Baro present** — needed for altitude hold.
9. **Solder pad layout** — does it fit through frame holes cleanly.
10. **Firmware target** — is it on Betaflight's "supported" list, or community-only?

---

## 7. How Phase 2 System 2 will sit in this landscape

The brief: **Lightweight AIO FC + ESC** for "compact autonomous UAVs", emphasising low weight, compact size, good thermal handling, low noise, power efficiency.

Reference competition to beat: **SpeedyBee F405 Mini AIO 25A** and **HGLRC F722 AIO**. Both ~5–7 g, 20×20, run Betaflight.

Suggested design starting points:
- MCU: **STM32H743** (or AT32F435 for cost) — gives headroom for INAV/iNav-derived autonomy if needed
- IMU: **ICM-42688-P** on SPI1
- Baro: **DPS310** on I²C
- Onboard ESC: 4-in-1 BLHeli_S board running **Bluejay** (open-source) — 25–35 A continuous
- Mounting: 20×20
- BEC: 5 V/3 A + 9 V/2 A
- Output protocols: DShot300/600 (Bluejay caps at 600)
- Compact 4-layer PCB, IMU on a small isolated copper pour, away from ESC switching

That spec sheet alone is the seed of System 2.

## Sources
1. Oscar Liang — *Flight Controller Explained* — https://oscarliang.com/flight-controller/
2. Oscar Liang — *FC Processors Explained (F4/G4/F7/H7/AT32)* — https://oscarliang.com/f1-f3-f4-flight-controller/
3. Betaflight — *Manufacturer Design Guidelines* — https://betaflight.com/docs/development/manufacturer/manufacturer-design-guidelines
4. Multirotor Guide — *F1, F3, G4, F4, F7, H7 MCUs* — https://www.multirotorguide.com/guide/f1-f3-f4-f7-and-h7-mcu-for-flight-controllers-explained/
5. Zbotic — *Matek FC Buying Guide F405/F722/H743* — https://zbotic.in/matek-flight-controller-buying-guide-f405-vs-f722-vs-h743/
6. SpeedyBee — product pages — https://www.speedybee.com/
7. Holybro Kakute H7 — https://holybro.com/products/kakute-h7
