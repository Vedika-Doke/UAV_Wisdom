# AIO Flight Controller — Datasheet Threshold Review & Design Cross-Check

**Date:** 2026-07-16
**Scope:** Every datasheet in `Datasheets/` read at threshold level (absolute maximum ratings, recommended operating conditions, electrical characteristics tables), cross-checked against a pin-by-pin net trace of `schematic_Design/Schematic_diagram.pdf` (11 pages). Every claim below cites the datasheet table/section and the schematic page it comes from.

**Datasheet sources used:**
- MPM3610GQV — MPS datasheet Rev 1.01 (`MPM.pdf`)
- FD6288/FD6288Q — Fortior preliminary datasheet (`GATEDRIVER.pdf`)
- AON7524 — Alpha & Omega Rev 1.0, Mar 2013 (`MOSFETdatasheet.pdf`)
- ICM-42688-P — TDK DS-000347 Rev 1.2 (note: `ICM.pdf` in the folder is only the 1-page product brief PB-000072; the full datasheet was fetched for this review — **recommend replacing `ICM.pdf` with DS-000347**)
- SX1280/SX1281 — Semtech datasheet Rev 2.2, May 2018 (`RFTRANSRECIEVER.pdf`)
- W25Q128JV — Winbond Rev F, Mar 2018 (`FLASHMEMORY.pdf`)
- STM32F411xC/xE — ST DocID026289 Rev 7 (`STM32.pdf`)
- STM32G071x8/xB — ST DS12232 (`ESC_MCU.pdf` = `STM32G071GBU6..pdf`, byte-identical duplicates)

---

## A. CRITICAL — will damage hardware or prevent operation

### A1. SX1280 powered from VBAT → destroyed at power-on
- **Datasheet:** SX1280 Table 3-1 (Absolute Maximum Ratings): VBAT and VBAT_IO abs-max = **3.9 V**. Table 3-2 (Operating Range): **1.8–3.7 V**.
- **Schematic (p.7):** VR_PA, VBAT_IO and VDD_IN of U4 (SX1280IMLTRT) are fed from the **VBAT** port (cross-referenced to the gate-driver sheets' VBAT net).
- **Why it's fatal:** the rest of the design cannot run below 2S (see A3), so VBAT = 7.4–8.4 V — roughly **2× the radio's absolute maximum**. Even on 1S, a fresh cell (4.2 V) exceeds 3.9 V abs-max.
- **Fix:** power the SX1280 from the 3.3 V rail (operating max 3.7 V covers it, but see B1 about the rail actually being 3.45 V).

### A2. Gate drivers unpowered — the `5V` net has no source
- **Schematic (pp.10–11):** FD6288Q VCC (pin 4) on both U6/U7, plus the three BAT54 bootstrap feeds, sit on a net named `5V`. The net's cross-reference list is only `gatedriver1[3B], gatedriver2[2A], gatedriver2[3B]` — it exists on no other sheet. The only regulator in the design is the MPM3610 producing 3.3 V (p.2); USB VBUS is a separate isolated net `USB_5V` (p.6).
- **Consequence 1:** the gate drivers never power up; no motor can spin.
- **Consequence 2 — abs-max violation:** FD6288 Absolute Maximum Ratings: logic inputs HIN/LIN rated **−0.3 V to VCC + 0.3 V**. With VCC = 0 V, the first 3.3 V PWM edge from the G071 exceeds the input abs-max by ~3 V.
- **Consequence 3 — even a real 5 V rail would be marginal:** FD6288 Static Characteristics: **VCC UVLO rising = 4.2 / 4.6 / 5.0 V (min/typ/max)**, falling 3.9/4.3/4.7 V. A worst-case part never comes out of UVLO on a 5.0 V nominal rail. Recommended VCC range is 5.0–20 V.
- **Fix:** feed FD6288 VCC directly from VBAT (2S: 7.4–8.4 V). VCC abs-max is 25 V, recommended max 20 V — safe. The AON7524 gate then sees ~8.4 V, within VGS abs-max ±12 V (AON7524 Absolute Maximum Ratings) and better than the 4.5 V spec point (RDS(on) ≤ 4 mΩ @ VGS = 4.5 V, Electrical Characteristics).

### A3. 1S operation is impossible (design is 2S–4S only)
- **MPM3610** Recommended Operating Conditions: **VIN 4.5–21 V**; Electrical Characteristics: **VIN UVLO rising 3.65/3.9/4.15 V**, hysteresis 650 mV (falling ≈ 3.25 V typ). A 1S pack (3.0–4.2 V) sits below or inside the UVLO band for its entire discharge curve.
- **FD6288** VCC UVLO rising up to 5.0 V (see A2) — 1S can never enable the gate drivers.
- **Upper bound:** MPM3610 VIN operating max 21 V → 4S (16.8 V) fine; 5S (21.0 V) has **zero margin**; 6S violates.
- **Doc conflict:** presentation slide 4 claims "VBAT 3.0–4.2 V → 3.3 V" (1S); slide 26 assumes 2S. Only 2S–4S is electrically valid.

### A4. MPM3610 EN pin tied directly to VIN — abs-max violation on 2S
- **Datasheet:** Absolute Maximum Ratings: "All other pins" (including EN) = **−0.3 to +6 V**. Enable Control section (Fig. 4): EN has an internal ~6.5 V zener + 1 MΩ; for VIN > 6 V, EN must connect through a resistor sized to keep zener current < 100 µA — R ≥ (VIN − 6.5 V)/100 µA.
- **Schematic (p.2):** EN (pin 17) is tied hard to IN (pin 16) = VBAT.
- **Consequence:** at 2S (8.4 V max), EN sees 8.4 V vs 6 V abs-max with no current limit — zener overstress.
- **Fix:** insert ≥ 20 kΩ (2S) between VBAT and EN, or a proper divider. For 4S (16.8 V): ≥ 103 kΩ.

### A5. USB wired to the wrong pins
- **Datasheet:** STM32F411 pin/alternate-function tables — USB OTG FS DM/DP exist **only on PA11/PA12**.
- **Schematic (p.3):** USB_DN → **PA1**, USB_DP → **PA2**; PA11/PA12 are no-connect.
- **Consequence:** USB can never enumerate; Configurator/DFU-over-USB impossible.

### A6. IMU SPI broken: PB12 and PB13 shorted
- **Schematic (p.3, net-trace verified twice):** PB12 and PB13 are joined by a junction; the merged net carries both **IMU_CS** and **IMU_SCLK** labels. Chip-select and SPI clock are one wire.
- **Consequence:** SPI2 to the ICM-42688-P cannot work. (ICM-42688-P DS-000347 Table 4: CS is a level input with VIH ≥ 0.7·VDDIO — it cannot double as a clock.)

### A7. No signal path between the flight controller and the ESCs
- **Schematic (pp.3, 8, 9):** the only nets shared between the F411 and the two G071s are **3.3V and GND**. No throttle, PWM, DShot, UART or telemetry net exists. On the F411 side every candidate output (PA0, PA8–PA12, PA15, PB2–PB4, PB6–PB9) is no-connect; on the G071 side the only connected signal pins are the six PWM outputs to the gate drivers.
- **Consequence:** motors are uncontrollable by design.

### A8. ESC SWD clock miswired — G071s cannot be flashed
- **Datasheet:** STM32G071 pinout: SWCLK = **PA14**.
- **Schematic (pp.8–9, both ESCs):** the SWCLK/SWCLK2 wire pads route to **PC14-OSC32_IN**; PA14-BOOT0 is no-connect.
- **Consequence:** SWD debug/flash cannot connect to either ESC MCU.
- (Note: floating PA14-BOOT0 is *not* itself an error — G071 factory option bytes default nBOOT_SEL = 1, the pin is ignored and a blank chip boots the system bootloader via the empty-check. RM0444 §3.5 / FLASH_OPTR default 0xFFFFFEAA.)

### A9. VCAP capacitor undersized on the F411
- **Datasheet:** STM32F411 Table 16: packages with a **single VCAP pin require 4.7 µF** (ESR < 1 Ω); 2×2.2 µF applies only to dual-VCAP packages. UFQFPN48 has one VCAP pin.
- **Schematic (p.3):** C32 on VCAP1 = **2.2 µF**.
- **Consequence:** core regulator (V12 = 1.26–1.38 V at Scale 1) stability out of spec — risk of erratic operation at 100 MHz.
- **Fix:** change C32 to 4.7 µF.

---

## B. WRONG-VALUE / MARGINAL — works by luck or eats all margin

### B1. The "3.3 V" rail is actually 3.45 V
- **Datasheet:** MPM3610 VFB = **0.798 V** (0.786–0.810 @25 °C; 0.782–0.814 over temp); VOUT = VFB × (1 + R_top/R_bottom). Recommended divider for 3.3 V out is **102 kΩ / 32.4 kΩ** (Table 1) → 3.31 V.
- **Schematic (p.2):** R4 = 100 kΩ (OUT→FB), R5 = 30.1 kΩ (FB→GND) → **VOUT = 3.449 V typ, 3.380–3.519 V worst-case**.
- **Margins consumed:**
  - STM32F411 & G071 operating VDD max = **3.6 V** (F411 Table 14; G071 Table 21) → 81 mV worst-case headroom.
  - ICM-42688-P VDD operating max = **3.6 V**, abs-max 4 V (DS-000347 Tables 3, 8) → 81 mV headroom.
  - W25Q128JV VCC operating = 3.0–3.6 V (Winbond §9.2) → 81 mV headroom.
  - SX1280 (once moved to this rail, see A1): operating max 3.7 V → OK but reduced.
- **Fix:** change R4/R5 to the datasheet-recommended 102 k/32.4 k (or 75 k/24 k for lower VIN designs).

### B2. Crystal load caps slightly off
- **Datasheet:** STM32F411 Table 37: CL range 5–25 pF; crystal CL must equal CL1·CL2/(CL1+CL2) + C_stray. ABLS-8.000MHZ-B2-T is an 18 pF-CL crystal.
- **Schematic (p.3):** C6 = C7 = 18 pF → effective 9 pF + ~5 pF stray ≈ 14 pF vs. required 18 pF — the oscillator will run a few tens of ppm fast and with reduced margin.
- **Fix:** 27–33 pF caps (or a 9–10 pF CL crystal). Not fatal, but cheap to fix.

### B3. ESC MCUs are sensor-blind: no ADC/comparator inputs at all
- **Schematic (pp.8–11):** every G071 ADC-capable pin (PA2–PA7, PB0, PB1) is no-connect. R24–R35 (100 kΩ) are **gate-source pull-down bleeders** (gate→phase for high-side, gate→GND for low-side), not sense dividers — no resistor midpoint routes to any MCU pin. No current-sense shunt exists (low-side sources go straight to GND).
- **Why it matters:** sensorless six-step control (per your ST app-note references, or AM32) requires BEMF/phase-voltage dividers into the comparators/ADC and needs current sense for protection. The G071 has the right peripherals (2.5 Msps ADC, internal comparators, 128 MHz-capable TIM1 — DS12232 Tables 7, 56) but nothing is wired to them.

### B4. IMU interrupt not connected
- **Schematic (p.4):** ICM-42688-P INT1/INT (pin 4) is no-connect; INT2/FSYNC tied to GND.
- **Why it matters:** flight firmware (Betaflight/Rotorflight) synchronizes the PID loop to the gyro data-ready EXTI interrupt. Without INT1 you're reduced to polled sampling with jitter.
- **Fix:** route INT1 to any free EXTI-capable F411 pin (e.g. PA8).

### B5. Two SPI clocks on one shared data pair
- **Schematic (p.3):** FLASH_CLK = PA4 and SX_SCK = PA5, while MISO/MOSI are shared (PA6 = FLASH_IO1 + SX_MISO, PA7 = FLASH_IO0 + SX_MOSI).
- **Why it's fragile:** this is not a standard shared SPI bus (one clock, multiple CS). It can only work if firmware bit-manages both clock pins and never overlaps transactions; standard SPI peripherals/drivers assume one SCK per bus. Also caps quad-SPI flash use to dual-IO forever.
- **Clock ceilings for reference:** SX1280 SPI max **18 MHz** (Table 9-1); ICM-42688-P **24 MHz** (DS-000347); W25Q128JV 133 MHz (50 MHz for 03h read) (Winbond §9.6).

### B6. AON7524 current ratings assume copper you don't have
- **Datasheet:** AON7524 note A: the 25 A (TA=25 °C) rating and RθJA = 60/75 °C/W assume **1 in² of 2 oz copper per device, still air**. Six DFN3×3 FETs on a 25×25 mm board cannot each get 1 in².
- **Why it's OK anyway:** a nano-UAV motor draws single-digit amps; with RDS(on) ≤ 4 mΩ @ 4.5 V (≤ 3.3 mΩ @ 10 V), conduction loss at 5 A is < 100 mW/FET. Fine — just don't trust the headline 28 A.

---

## C. Items verified OK (thresholds line up)

| Interface | Requirement (datasheet) | Design value | Verdict |
|---|---|---|---|
| G071 PWM → FD6288 logic | VIH ≥ 2.7 V (FD6288 static char.; "3.3/5 V compatible") | G071 VOH ≥ VDD−0.4 = 2.9 V @3.3 V (DS12232 Table 52); inputs are 200 kΩ pull-down, ~0 load → VOH ≈ VDD | ✅ (~0.6 V margin) |
| FD6288 dead time | internal 100/200/300 ns + shoot-through protection | complementary HIN/LIN from TIM1 | ✅ |
| FD6288 drive vs FET gate | ±1.5/1.8 A source/sink; 10 Ω external gate R | AON7524 Qg = 16 nC @4.5 V, 37 nC @10 V, Rg = 3 Ω typ | ✅ fast, clean |
| AON7524 VDS | 30 V abs-max, 36 V 100 ns spike | 8.4 V bus (2S) | ✅ 3.5× margin |
| AON7524 VGS | ±12 V abs-max | ≤ 8.4 V (VBAT-fed driver) | ✅ |
| Body diode | VSD ≤ 1 V, trr 16 ns | 6-step commutation | ✅ |
| F411 3.3 V logic ↔ flash/IMU/SX1280 VIH/VIL | all 0.7/0.3·VDD class | same rail | ✅ |
| BOOT0 strap (F411) | VIL ≤ 0.1·VDD+0.1 = 0.43 V | 10 kΩ to GND | ✅ electrically (but see DFU note below) |
| NRST buttons + 10 kΩ pull-ups | internal 30–50 kΩ RPU, ≥300/350 ns pulse | TL3342 buttons | ✅ |
| Reset/SWD pads (F411) | PA13/PA14 + GND + 3V3 | present | ✅ |
| MPM3610 load | 1.2 A continuous, 2.4 A min current limit | digital load ≈ 0.15–0.3 A (F411 ~22 mA max @100 MHz + 2× G071 ~7 mA + IMU 0.9 mA + flash ≤25 mA + SX1280 TX 24 mA) | ✅ large margin |
| USB-C CC pulldowns | 5.1 kΩ Rd (UFP per USB-C spec) | R10/R11 = 5.1 kΩ | ✅ |

**Non-electrical but required for Rotorflight** (from rotorflight-ref-design/FC-Design-Requirements.md): DFU **button** (BOOT0 here is hard-strapped low — unreachable without SWD), two indicator LEDs, barometer (SPL06/DPS310), ≥1 Gbit blackbox flash (W25N01G; W25Q128 is listed "supported but not large enough"), 5 V ≥1 A external rail, ADC sensing of Vx and +5 V, servo headers with the timer rules, serial receiver UART. Plus: F411 is EOL in Rotorflight ("support for lesser MCUs like STM32G474 and STM32F411 is EOL and will be removed soon" — rotorflight-firmware README) and SPI ExpressLRS is compiled out of RF2 (`#undef USE_RX_EXPRESSLRS`, `#undef USE_RX_SX1280` in `src/main/target/STM32_UNIFIED/target.h`).

---

## D. Consolidated fix list for Rev B

1. SX1280 supplies → 3.3 V rail, never VBAT (A1).
2. FD6288 VCC → VBAT; delete the phantom `5V` net or add a real 5 V buck (A2) — a 5 V buck is needed anyway for Rotorflight servos/peripherals.
3. Spec the battery as 2S–4S everywhere; fix slide 4 (A3).
4. ≥20 kΩ resistor between VBAT and MPM3610 EN (A4).
5. USB_DN/DP → PA11/PA12 (A5).
6. Separate IMU_CS (PB12) and IMU_SCLK (PB13) (A6).
7. Add throttle-signal (+ telemetry) nets FC → each G071 (A7).
8. ESC SWCLK → PA14 (A8).
9. VCAP C32 → 4.7 µF (A9).
10. FB divider → 102 k/32.4 k for a true 3.3 V rail (B1).
11. Crystal caps → ~30 pF, or use a 9 pF-CL crystal (B2).
12. Add BEMF dividers + low-side shunts to G071 ADC/COMP pins (B3).
13. IMU INT1 → EXTI pin on the FC (B4).
14. Give the SX1280 (or its replacement serial-ELRS section) its own SPI or a proper single-clock shared bus (B5).
15. Replace `ICM.pdf` product brief with the full DS-000347 datasheet; delete the duplicate `STM32G071GBU6..pdf`.
