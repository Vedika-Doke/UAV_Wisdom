# ESC Firmwares

The ESC's MCU runs firmware that does the commutation, processes DShot commands from the FC, and reports telemetry back. As of 2026, this is a four-horse race — but with a clear winner direction.

---

## 1. The four firmwares

| Firmware | MCU class | License | Active dev? | Status (2026) |
|----------|-----------|---------|:-----------:|---------------|
| **BLHeli_S** | 8-bit EFM8BB1/2 (48 MHz) | Closed (free) | ❌ No new dev | Cheap, ubiquitous on entry ESCs. **Flash to Bluejay.** |
| **BLHeli_32** | 32-bit ARM Cortex-M0/M4 | Closed (paid license) | ❌ **Discontinued 2024** | Was king. License model dead. Migrate to AM32. |
| **AM32** | 32-bit ARM Cortex-M (STM32G071, AT32F421, etc.) | **Open source (MIT)** | ✅ Very active | **The future** — drop-in BLHeli_32 replacement, more flexibility |
| **Bluejay** | 8-bit EFM8BB1/2 (same as BLHeli_S) | **Open source** | ✅ Active | Unlocks BLHeli_S hardware to do bidirectional DShot + RPM telemetry |

### The big 2024 inflection
**BLHeli_32 was discontinued.** The company that owned it stopped issuing new licenses; no further development. Any new ESC design needs to pick another firmware:

- **For 32-bit MCUs → AM32** (open source, actively developed, matches/exceeds BLHeli_32 features)
- **For 8-bit MCUs → Bluejay** (unlocks features that BLHeli_S couldn't do)

For Phase 2 ESC design, **AM32 on AT32F421 or STM32G071** is the no-brainer modern choice.

---

## 2. Feature comparison

| Feature | BLHeli_S | Bluejay | BLHeli_32 | AM32 |
|---------|:---:|:---:|:---:|:---:|
| DShot 150/300/600 | ✅ | ✅ | ✅ | ✅ |
| DShot 1200 | ❌ | ❌ | ✅ | ✅ |
| **Bidirectional DShot** (for RPM filter) | ❌ | **✅** | ✅ | ✅ |
| KISS/BLHeli serial telemetry | partial | ✅ | ✅ | ✅ |
| Beacon (lost-model finder) | ✅ | ✅ | ✅ | ✅ |
| Variable PWM frequency | ❌ | **✅** | ✅ | ✅ |
| Stall protection | basic | ✅ | ✅ | ✅ |
| Auto-timing adjustment | ❌ | ✅ | ✅ | ✅ |
| Sinusoidal startup | ❌ | partial | ✅ | ✅ |
| **FOC mode** | ❌ | ❌ | ❌ | 🟡 in development |
| Configurator | BLHeliSuite | Bluejay Configurator / BLHeliSuite | BLHeliSuite32 | **AM32 Configurator (web)** |

---

## 3. Why bidirectional DShot matters

This is the single biggest feature delta between old and new ESC firmwares.

**Forward DShot** — FC sends a 16-bit packet per motor, every PID iteration. ESC just executes.

**Bidirectional DShot** — same outbound packet, but the ESC also *replies* with its current measured RPM (eRPM). The FC now knows the *actual* motor speed each cycle.

Why this is huge:
- **RPM filter** — Betaflight/INAV/iNav can dynamically place notch filters at exactly each motor's frequency, knocking out the dominant vibration source before it reaches the gyro. Result: smoother flight, *cooler motors* (less D-term oscillation = less power into damping).
- **More accurate throttle control** — closed-loop on RPM, not just commanded duty cycle.
- **No extra wires** — same single signal wire, just half-duplex.

Bidirectional DShot is **mandatory for modern FPV tuning**. If an ESC can't do it, it's a 2018-era part.

---

## 4. Flashing — practical workflow

ESCs are flashed over the **same wire that sends DShot commands**, by putting them into a bootloader mode. The FC acts as a USB-to-ESC bridge ("passthrough").

- **BLHeli_S → Bluejay:** use **BLHeliSuite** (Windows) or **EscConfigurator** (Chrome web app, cross-platform). Pick the matching Bluejay file (EFM8BB21 vs EFM8BB51 vs G_H_30 etc. — the layout determines which file).
- **BLHeli_32 → AM32:** **AM32 Configurator** (web) connects via FC passthrough; the MCU hex file is picked per-target.

Bricking risk is low — both Bluejay and AM32 provide rescue paths via SWD (requires physical access).

---

## 5. The MCU shift driving everything

| Era | MCU | Firmware | Cost (ESC, 4-in-1 60A) |
|-----|-----|----------|------------------------|
| 2015–2018 | EFM8BB1/2 (8-bit, 48 MHz) | BLHeli_S | ~$30 |
| 2018–2024 | EFM8 or 32-bit ARM | BLHeli_32 | ~$50–80 |
| **2024+** | AT32F421 / STM32G071 (32-bit ARM @ 64–170 MHz) | **AM32** | ~$40–70 |

The hardware actually got *cheaper* with AM32 — the AT32F421 from Artery is dirt cheap (~$0.50 in volume), making 32-bit firmware affordable on every ESC.

---

## 6. Licensing implications for Phase 2

| Firmware | Can I ship it in a product? |
|----------|-----------------------------|
| BLHeli_S | Yes (free use, but no source modifications) |
| BLHeli_32 | **No** — license model is dead |
| **AM32** | **Yes, MIT license — fully commercial-friendly, source modifications allowed** |
| **Bluejay** | **Yes, open source** |

So for Phase 2 System 2 (AIO FC + ESC), the ESC firmware decision is:
- **8-bit MCU (EFM8BB21) → Bluejay**
- **32-bit MCU (AT32F421 / STM32G071) → AM32** ← **recommended**

AM32 lets you ship a hackable ESC and not worry about license fees. You can also contribute upstream — adding a tuned target for your specific board.

---

## 7. Going further — the FOC frontier

Trapezoidal commutation (what all 4 firmwares above use) is good enough for FPV. The next frontier is **FOC (Field-Oriented Control)**:

| Aspect | Trapezoidal (current) | FOC (future) |
|--------|----------------------|--------------|
| Efficiency | Good | **Better** (5–10 %) |
| Noise | Audible whine + harmonics | **Near silent** |
| Smoothness | Notable steps at low RPM | **Smooth at any RPM** |
| Compute cost | Low (8-bit MCU enough) | High (32-bit, fast PWM, current sense) |
| Current sense | Optional | **Mandatory per phase** |
| Used in | All hobby ESCs | VESC (e-bikes), AM32 (experimental), some industrial servos |

AM32 has experimental FOC modes; **VESC** (the open-source ESC platform from Benjamin Vedder, originally for electric skateboards) is the reference. For Phase 2, trapezoidal AM32 is safe; FOC is a research stretch goal.

---

## Sources
1. UAVMODEL — *BLHeli_32 vs Bluejay vs AM32 ESC Firmware* — https://blog.uavmodel.com/blheli_32-vs-bluejay-vs-am32-esc-firmware-features-performance-and-setup/
2. Oscar Liang — *Identify ESC Firmware* — https://oscarliang.com/identify-esc-firmware/
3. Unmanned Tech — *ESC Firmware Compared (2026)* — https://www.unmannedtechshop.co.uk/blogs/knowledge-base/esc-firmware-blheli-s-vs-am32-vs-blheli-32
4. FpvGuru — *End of BLHeli_32 and the Future of ESC Firmware* — https://fpvguru.in/blogs/the-end-of-blheli_32-and-the-future-of-esc-firmware-in-fpv-drones/
5. ZEXFPV — *AM32 vs BLHeli_32 (2026)* — https://zexfpv.com/blogs/fpv-guides/am32-vs-blheli_32-fpv-esc-firmware-comparison-guide-2026
6. ArduPilot Docs — *BLHeli32, AM32, and BLHeli_S ESCs* — https://ardupilot.org/copter/docs/common-blheli32-passthru.html
7. AM32 GitHub — https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware
8. Bluejay GitHub — https://github.com/bird-sanctuary/bluejay
9. VESC Project — https://vesc-project.com/
