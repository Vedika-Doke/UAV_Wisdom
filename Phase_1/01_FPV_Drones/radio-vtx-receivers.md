# Radio Link, VTX & Receivers

An FPV drone needs **two independent wireless links**:

1. **Control link (RC)** — pilot's stick inputs → drone. Low bandwidth (a few kbps), must be ultra-reliable, long range.
2. **Video link (VTX)** — drone's camera → pilot's goggles. High bandwidth (Mbps), low latency. Range matters less because *if you can't see, you can't fly anyway*.

A common mistake: thinking these are one system. They're independent — you can run **DJI O4 video + ExpressLRS control**, for example. Beginners often confuse them because "DJI goggles" bundle both.

---

## 1. Control link (RC link)

### Evolution
- **PWM** (one wire per channel) → **PPM** (all channels on one wire) → **SBUS/IBUS** (digital serial) → **CRSF** (Crossfire / ELRS, bidirectional with telemetry)
- Older "FrSky D16" / "ACCESS" systems are legacy; modern builds use **ExpressLRS (ELRS)** or **TBS Crossfire**.

### Comparison: the three serious options

| Aspect | **ExpressLRS (ELRS)** | **TBS Crossfire / Tracer** | DJI (paired with O3/O4 goggles) |
|--------|------------------------|----------------------------|---------------------------------|
| License | **Open source** | Closed (TBS) | Closed (DJI) |
| Band | 2.4 GHz **or** 900 MHz | 900 MHz (Crossfire), 2.4 GHz (Tracer) | 2.4 GHz |
| Max packet rate | 1000 Hz (2.4 GHz) / 200 Hz (900 MHz) | 150 Hz | ~100 Hz |
| Latency | **~3 ms** (1000 Hz mode) | ~10 ms | ~15 ms (racing mode) |
| Range (open field, 25 mW) | 3–15 km depending on mode | ~10 km | ~10 km (FCC) |
| Range (50 Hz, 868/915 MHz, high power) | **30+ km** | 30+ km | — |
| Telemetry back | Yes (CRSF) | Yes (CRSF) | Yes |
| RX cost | **$8–25** | $30–60 | bundled with DJI ecosystem |
| TX cost | $30–200+ | $200+ | bundled |
| Notes | Dominant choice in 2026 — cheap, fast, open. | Mature, stable, "set and forget". | Locked-in but tightly integrated with DJI goggles. |

**Default recommendation (2026):** ExpressLRS 2.4 GHz for racing/freestyle, ELRS 900 MHz for long-range. Crossfire is still excellent but losing market share.

### Protocols on the wire (RX → FC)
The receiver hands data to the FC over one of these serial protocols:
- **CRSF** (Crossfire Serial) — used by both TBS Crossfire and ELRS. 420 kbps, bidirectional, supports OpenTX/EdgeTX **LUA scripts** for live config from the radio.
- **SBUS** (Futaba) — 100 kbps, 16 channels, **inverted UART** (FC must support inversion or have an inverter).
- **IBUS** (FlySky) — non-inverted serial, simpler than SBUS.

Modern FCs almost universally have CRSF support. Just pick a UART, set protocol = CRSF, done.

---

## 2. Video link (VTX)

Two parallel ecosystems: **analog** (cheap, instant, low quality) and **digital** (expensive, ~30 ms latency, HD).

### Analog (5.8 GHz)
- 40 channels across 5 bands (A, B, E, F/Airwave, R/Raceband)
- Power output: 25 mW (legal in most countries unmodified) up to 1–2 W
- Resolution: 480p-ish — looks like an old VHS tape
- Latency: **<10 ms** end-to-end (the lowest of any system)
- Drops out gracefully — signal degrades to static, not a hard cutoff
- Vendors: TBS Unify Pro, Rush Tank, AKK FX2

✅ Cheap, ubiquitous, lowest latency, "race-legal" everywhere
❌ Poor image quality, susceptible to interference, channel coordination needed if flying with others

### Digital
| System | Resolution | Latency | Range (advertised) | Open / closed |
|--------|------------|--------:|-------------------:|---------------|
| **DJI O3 Air Unit** | 1080p/100fps | ~30 ms | 10 km (FCC) | Closed |
| **DJI O4 Air Unit** | 1080p/100fps (racing) / 4K/60 recording | **~15 ms** | **15 km (FCC)** | Closed |
| **Walksnail Avatar HD** | 1080p | ~22 ms | ~5 km | Closed |
| **HDZero** | 720p/60 | **<20 ms** | 1–2 km | Semi-open (race-favoured) |

**HDZero** wins for racing (lowest digital latency, channel-based like analog → multiple pilots can fly simultaneously). **DJI O3/O4** wins for image quality and range. **Walksnail** sits between.

Important: digital systems are typically *not* channel-shared like analog — they use OFDM-style allocation, so flying multiple pilots simultaneously requires their protocols to coordinate (HDZero does this well; DJI O3 supports up to 8 pilots).

### What's on the drone vs in the goggles
| End | Component |
|-----|-----------|
| Drone | **VTX** (video transmitter) + **camera** (for analog: a separate 600 TVL CMOS camera; for digital: integrated module) |
| Pilot | **Goggles** (DJI Goggles 3, Fat Shark Recon HD, Skyzone Cobra, etc.) with built-in receiver |

---

## 3. Receivers (RX)

Two flavours:

| Type | Form factor | Use |
|------|-------------|-----|
| **Diversity RX** | Two antennas | Better link reliability — picks the stronger signal per packet |
| **Single-antenna RX** | One antenna | Lighter, cheaper, for micros |

Common ELRS receivers in 2026:
- **HappyModel EP1 / EP2** — $10–15, single antenna, tiny
- **RadioMaster RP1 / RP3** — $15–25, diversity options
- **BetaFPV ELRS Lite** — sub-1 g for tinywhoops

Connect via 3 wires (5 V, GND, UART RX/TX on a single half-duplex line for CRSF).

---

## 4. What it costs to start

| Item | Reasonable choice | Price (USD) |
|------|-------------------|------------:|
| Radio Tx | RadioMaster Pocket (ELRS, EdgeTX) | $80 |
| RX | HappyModel EP2 | $12 |
| VTX (analog) | TBS Unify Pro32 Nano | $40 |
| Camera (analog) | RunCam Phoenix 2 | $35 |
| **OR** Digital combo | DJI O3 Air Unit | $230 |
| Goggles (analog) | Eachine EV800D | $130 |
| **OR** Goggles (digital) | DJI Goggles 3 | $500 |

Analog stack: ~$300 entry. Digital stack: ~$1000 entry.

---

## 5. What this means for our project

For Phase 2 (designing FCs), the relevant takeaways:

- **Reserve at least 2 UARTs** on the FC for RC (CRSF) and VTX (MSP DisplayPort for OSD on digital VTX).
- Provide a **9 V regulator** if the FC will power DJI/HDZero VTXs (they want 7.4–17.6 V but 9 V is the sweet spot for heat).
- Support **SBUS inversion** in hardware (STM32 H7 UARTs can invert in hardware; F4 generation could not).
- For the **commercial FC** (Phase 2 System 1), use **MAVLink-over-915 MHz SiK telemetry radio** (not CRSF) — that's the GCS link, separate from any RC.

## Sources
1. Oscar Liang — *ExpressLRS vs Crossfire* — https://oscarliang.com/expresslrs/
2. UAVMODEL — *DJI O4 vs O3 Air Unit (2026)* — https://blog.uavmodel.com/dji-o4-vs-o3-air-unit-real-world-latency-image-quality-and-range-comparison-2026/
3. DJI O4 Air Unit Specs — https://www.dji.com/o4-air-unit/specs
4. DJI O3 Air Unit Specs — https://www.dji.com/o3-air-unit/specs
5. 360 Rumors — *ELRS 2.4 vs Crossfire / Tracer: range & penetration* — https://360rumors.com/expresslrs-2-4-vs-crossfire-tbs-tracer-range-penetration/
6. ExpressLRS Docs — https://www.expresslrs.org/
7. TBS Crossfire Manual — https://www.team-blacksheep.com/products/prod:crossfire_micro_txse
