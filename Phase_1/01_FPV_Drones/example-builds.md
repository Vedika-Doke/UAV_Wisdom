# Example FPV Builds

Three reference builds chosen to **span the design space**: a beginner-friendly default, an aggressive racer, and a long-range cruiser. For each: the bill of materials with rationale, expected performance, and the firmware stack.

Use these as anchors when reading any other build guide — once you understand *why* each component was chosen, swapping things becomes intuitive.

---

## Build 1 — "The default" 5" Freestyle

The single most common FPV configuration in 2026. If you're learning, build this.

| Slot | Part | Spec | Why |
|------|------|------|-----|
| Frame | iFlight Nazgul Evoque F5D | 5" true-X, 30.5×30.5, 213 mm WB | Cheap, well-supported, replacement arms easy |
| FC | SpeedyBee F405 V4 | STM32 F405, ICM-42688-P, baro, 30.5×30.5 | Solid F4 board, Betaflight default |
| ESC | SpeedyBee BLS 50A 4-in-1 | 50 A cont, BLHeli_S, 30.5×30.5 stack | Matches the FC, BLS = Bluejay compatible |
| Motors | EMAX ECO II 2207 1900KV ×4 | 2207, 1900KV (6S) | Affordable, well-balanced |
| Props | HQProp 5.1×3.1×3 ×4 | 5.1" tri-blade, 3.1" pitch | Freestyle sweet spot |
| RX | RadioMaster RP1 ELRS 2.4G | CRSF, single-antenna | Cheap, low-latency |
| VTX | Foxeer Reaper Nano 5.8G | Analog, 25–600 mW switchable | Classic analog |
| Camera | Foxeer Razer Mini | 1200 TVL CMOS, 16:9 | Good colour & low-light |
| Battery | CNHL Black 6S 1300 mAh 100C | XT60 | Default 5" freestyle pack |
| Goggles | Skyzone Cobra X V4 | Analog, OLED | Analog match |

**Expected AUW:** ~620 g
**TWR:** ~6.5:1
**Flight time:** 4–5 min mixed freestyle, 6+ min cruising
**Total cost:** ~$450 (drone) + ~$80 (radio) + ~$200 (analog goggles) = **~$730**

**Firmware:** Betaflight 4.5 with **Bluejay** on the ESC (BLHeli_S board → unlocks bidirectional DShot → RPM filter → smoother flight + cooler motors).

---

## Build 2 — Aggressive 5" Racer / Punchy Freestyle

Same class, optimised for peak performance instead of cost.

| Slot | Part | Why different from default |
|------|------|----------------------------|
| Frame | Armattan Marmotte 5" | Lifetime warranty, premium carbon |
| FC | Holybro Kakute H7 | STM32 H7 → 8 kHz PID loop, 2 IMUs |
| ESC | T-Motor F55A Pro II | 55 A BLHeli_32, top-tier MOSFETs |
| Motors | T-Motor F40 Pro V 1950KV ×4 | Premium balanced motors |
| Props | Gemfan 51466 tri-blade | Stiffer, more snap |
| RX | TBS Crossfire Nano w/ diversity | 900 MHz, penetration through forests |
| VTX | TBS Unify Pro32 Nano + foldable Lollipop | Premium analog |
| Battery | Tattu R-line v5 6S 1300 mAh 150C | Lowest IR available |

**Expected AUW:** ~580 g
**TWR:** ~8:1
**Total cost:** ~$700 + radio + goggles

Race-tuned: higher P/D gains, lower D filter cutoff, accepts more vibration in exchange for sharper response.

---

## Build 3 — 7" Long-Range Cruiser

Different category entirely. Quiet, slow, *long*.

| Slot | Part | Spec | Why |
|------|------|------|-----|
| Frame | iFlight Chimera7 Pro | 7" dead-cat, 295 mm WB | Tilted FC, GPS pad |
| FC | Holybro Kakute H7 Mini | H7, dual IMU, baro | Long-range needs GPS hold |
| ESC | Holybro Tekko32 F4 65A | 65 A BLHeli_32 + onboard F4 telem | Headroom for big props |
| Motors | T-Motor F90 1300KV ×4 | 2806.5 stator, low KV for 6S | Efficient cruise |
| Props | HQProp 7×3.5×3 | 7" tri-blade | Pushes air efficiently |
| GPS | Holybro M10 GPS | u-blox M10, baro, mag | Required for RTH |
| RX | RadioMaster RP3 ELRS 915 MHz | Diversity, long-range mode | Range > latency |
| VTX | DJI O3 Air Unit | Digital, 1080p, ~10 km | Quality video at distance |
| Battery | 6S2P Molicel P45B (Li-Ion) | ~600 g, 9000 mAh | 25 min cruise |

**Expected AUW:** ~1200 g
**TWR:** ~3.5:1 (cruise-tuned, not snappy)
**Flight time:** 20–30 min
**Range:** 15–25 km round trip
**Total cost:** ~$1100 drone-only

**Firmware:** **INAV** or **iNav-derived "BetaFlight Long Range"** with GPS Rescue. Some long-range pilots use **ArduPilot Copter** on the Kakute for autonomy.

---

## How to read these

For every drone you see online, mentally fill in this table:
- Frame class & wheelbase
- Motor stator + KV
- Prop size + blade count
- Battery (chemistry × S × mAh)
- FC firmware
- Video link type

Once you can do that quickly, you can predict the drone's flight time, agility, and use case before reading any review.

---

## How these relate to Phase 2

Phase 2's **System 1 (Commercial FC)** maps closest to **Build 3** territory — long endurance, GPS, smart battery, autonomy.
Phase 2's **System 2 (AIO FC+ESC)** maps closest to **Build 1** territory — lightweight, integrated, FPV-style modular.

So when designing, validate against these reference profiles: would a Build-3-style cruiser fit cleanly onto System 1? Would Build 1's swap to "System 2" actually save weight? Real builds keep the design honest.

## Sources
1. Oscar Liang — *5-Inch FPV Drone Build* — https://oscarliang.com/build-fpv-drone-5-inch/
2. Joshua Bardwell — recent 5" build videos (YouTube)
3. iFlight Chimera7 Pro product page — https://shop.iflight.com/
4. Holybro Kakute H7 — https://holybro.com/products/kakute-h7
