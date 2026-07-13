# Presentation Content — Helicopters & Rotorflight for the AIO FC+ESC PCB
*Slide-by-slide content + speaker notes. Paste into Gamma / any slide generator. ~15 slides, 12–15 min talk.*

---

## Slide 1 — Title
**Title:** Single-Rotor Flight: Helicopter Mechanics, Rotorflight, and Our AIO FC+ESC PCB
**Subtitle:** SURP 2026 · IIT Bombay · UAV Research Project
**Footer:** Phase 2 — Helicopter platform

**Speaker notes:** The project has shifted to a single-rotor helicopter platform. This talk covers (1) how helicopters actually fly, (2) the Rotorflight firmware ecosystem, and (3) what both demand from the all-in-one flight controller + ESC PCB we are designing.

---

## Slide 2 — Why a Helicopter, Not Another Quad
- **Efficiency:** one large rotor = low disc loading = less power per kg of thrust; hover power scales with √(disc loading) → better endurance
- **Control authority:** collective pitch at constant RPM = millisecond thrust response, including *negative* thrust (inverted flight, full 3D envelope)
- **Safety:** autorotation — a controlled power-off landing; no multirotor can do this
- **The trade:** swashplate, linkages, gears, 3–4 servos, RPM loop, severe vibration — that complexity is exactly what our PCB must serve

**Speaker notes:** Multirotors won because they're mechanically trivial. Helicopters win on physics: aerodynamic efficiency, instant bidirectional thrust, and a real engine-out safety mode. The cost is complexity — which moves onto our board.

---

## Slide 3 — The Rotor Disc Is a Rotating Wing
- Each blade is an airfoil; the swept circle acts as one "rotor disc" producing a thrust vector perpendicular to the disc
- **Collective pitch** = all blades equally, at once → total thrust (the heli's "throttle")
- **Cyclic pitch** = sinusoidal variation once per revolution → tilts the disc → tilts thrust → pitch/roll/translation
- **Anti-torque:** Newton's 3rd law spins the fuselage opposite the rotor; the tail rotor (variable pitch, ~4–6× headspeed) cancels it and provides yaw, consuming ~5–10% of power

**Suggested visual:** side-by-side diagram — level disc (hover) vs tilted disc (translation); inset showing low vs high blade pitch.

**Speaker notes:** Everything in helicopter control reduces to manipulating blade pitch: collectively for thrust, cyclically for attitude, and at the tail for yaw. Alternatives to the tail rotor exist (NOTAR, coaxial, tandem) but single main rotor + tail rotor is the standard RC configuration and ours.

---

## Slide 4 — The Swashplate
- The rotating-to-fixed interface: two rings on a spherical bearing around the main shaft
  - **Fixed ring** — pushed/tilted by the servos
  - **Rotating ring** — spins with the head, drives pitch links to blade grips
- Raise the plate = collective · Tilt the plate = cyclic
- **eCCPM (electronic mixing):** 3 servos attach directly to the swashplate; the flight controller does the mixing math. Standard geometries: **120°** (dominant), 135°/140°, legacy 90°/H1
- Rotorflight mixes 120°/135°/140° natively + phase angle + blade pitch limits ("cyclic ring")

**Suggested visual:** swashplate cutaway — servos below, fixed ring, rotating ring, pitch links to blades.

**Speaker notes:** Mechanical mixing is obsolete; today three servos push the plate directly and software coordinates them. This makes the FC responsible for the aircraft's geometry — a firmware feature, not a mechanical one.

---

## Slide 5 — The Key Idea: The Flight Controller IS the Flybar
- **Old:** the Bell-Hiller flybar — a weighted paddle bar acting as a mechanical gyroscope, feeding stabilizing cyclic into the head. Costs: drag, complexity, damped agility
- **Now:** flybarless (FBL) — the bar is deleted; a bare FBL head is dynamically unstable
- A 3-axis MEMS gyro + PID loop replicates the flybar in software, correcting the swash servos ~1000×/second
- **Every modern RC helicopter is FBL**

**Speaker notes:** This is the single most important slide for justifying our board: a flybarless helicopter cannot fly at all without electronic stabilization. Gyro quality, filtering, loop latency and servo drive are flight-critical — the aircraft's stability literally lives on the PCB.

---

## Slide 6 — Rotor Dynamics the Controller Must Respect
1. **~90° phase lag** ("gyroscopic precession"): cyclic input → max blade response ~90° later in rotation; really a resonance of blade flapping at 1/rev; exposed in firmware as a *phase angle* parameter
2. **Dissymmetry of lift & flapping:** advancing blade sees more airspeed than retreating blade; blades flap to self-equalize; retreating-blade stall limits top speed
3. **Ground effect:** less power needed within ~1 rotor diameter of ground → governor load transients on takeoff/landing
4. **Autorotation:** drop collective → upward airflow keeps the rotor driving itself → trade stored rotor energy for a flare and soft touchdown. ESC implication: **autorotation bailout** (fast re-spool if aborted)

**Speaker notes:** These four effects are why heli firmware has parameters quads never need — phase angle, governor precompensation, bailout timers.

---

## Slide 7 — What an FBL Controller Computes (control loop)
**Chain 1 (attitude):** RX (CRSF/ELRS/SBUS) → Rate PID + Feedforward (gyro @ 8 kHz, PID @ 1–2 kHz) → CCPM mixer → 4 servos (cyclic @ 333 Hz, tail @ 560 Hz narrow-pulse)
**Chain 2 (headspeed):** RPM sense (bidirectional DSHOT / ESC telemetry) → Governor PID + collective precomp → ESC/motor → also feeds RPM notch filters

- Tail axis runs heading-hold (integrating yaw loop + torque feedforward)
- **Why constant RPM + collective, not throttle:** rotor inertia is huge; RPM changes are far too slow for control. Hold RPM, fly on pitch. Bonus: fixed vibration frequencies + stored energy for autorotation

**Suggested visual:** two-row block diagram (Napkin AI does this well from the text above).

**Speaker notes:** Contrast with a quad: a quad modulates four motor RPMs; a heli holds one RPM constant and moves four servos. Different actuators, different loop rates, different filtering philosophy.

---

## Slide 8 — Electric Heli Powertrain by the Numbers
| Class | Headspeed | ESC (continuous) | Battery |
|---|---|---|---|
| 250 | ~3500–4500 RPM | 20–40 A | 3–4S |
| **450** | **~2700–3400 RPM** | **60–80 A** | **6S** |
| 550 | ~2200–2800 RPM | 100–120 A | 6–12S |
| 700 | ~1300–2200 RPM | 120–160 A | 12–14S |

- One motor → pinion → main gear (~8–12:1) → main shaft; same motor drives the tail via torque tube or belt (tail ~4–6× headspeed)
- Heli ESC benchmark (Hobbywing Platinum): governor mode, soft spool-up, **autorotation bailout**, **BEC 5–8 V @ ~10 A cont / 25 A peak**, RPM/V/A/temp telemetry

**Speaker notes:** The 450-class row is our design anchor: ~60–80 A continuous on 6S. The Platinum feature list is effectively our ESC-stage requirements spec.

---

## Slide 9 — Rotorflight: Betaflight, for Helicopters
- Open source, **GPLv3**; lineage MultiWii → Baseflight → Cleanflight → **Betaflight 4.3** → Rotorflight 2
- Exclusively single-rotor helicopters (no multirotor/plane support)
- Current: **RF 2.3 / firmware 4.6.0** (June 2026) — new rates system, nitro governor, ESC forward programming (AM32/BLHeli/ZTW/XDFly), native ELRS RPM telemetry
- **Ecosystem:** Configurator (setup/flash GUI) · Blackbox (log analysis) · LUA suites (full tuning from the transmitter, EdgeTX/Ethos) · rotorflight-targets (board configs loaded live by the flasher)
- Runs on STM32 **F405 / F722 / F745 / H743**

**Speaker notes:** Rotorflight brought the open, blackbox-driven, community-tuned FPV workflow to helis at 1/3–1/10 the cost of commercial FBL units. For us the decisive point comes two slides later: it's the only heli firmware where we can define our own board.

---

## Slide 10 — Rotorflight's Heli-Specific Machinery
- **Governor modes:** OFF · PASSTHROUGH (spool-up + bailout only) · STANDARD (PID headspeed) · MODE1 (+ collective/cyclic precompensation) · MODE2 (+ battery voltage compensation)
- **Tail:** variable-pitch servo, motorized, or bidirectional motor; **TTA** (spikes main RPM to yaw a fixed-pitch tail in its dead direction); collective/cyclic feedforward to tail; CW/CCW stop gains
- **Rescue mode:** flip upright → pull-up → climb → hover (staged, tunable)
- **Piro compensation:** PID error vector rotates with the pirouette (stays earth-referenced)
- **Geometry protection:** cyclic ring (~16°), collective limit, total pitch limit (no mechanical binding)
- **PID per axis:** P·I·D + Feedforward + Boost — FF does most of the flying; PID trims error

**Speaker notes:** None of these exist in multirotor Betaflight. Each maps to a physical effect from slides 3–6 — good exam-style mapping if asked.

---

## Slide 11 — Vibration: the Enemy, and the RPM-Notch Answer
- Heli gyro spectrum: main rotor **1/rev ≈ 25–70 Hz**, blade-pass harmonics, tail (~4–6× main), motor frequency — all inside the control band
- Broadband low-pass filtering would add fatal latency
- **The synergy:** headspeed is governed constant + gear ratios are known → Rotorflight converts live motor RPM to exact rotor frequencies and parks narrow notch filters on 1st/2nd/3rd harmonics of main and tail
- Clean gyro → higher usable gains → crisper flight
- Requires live RPM → our integrated ESC provides it for free (bidirectional DSHOT / telemetry)

**Suggested visual:** frequency spectrum with peaks (main 1/rev, 2/rev, tail, motor) and dashed notch markers over them.

**Speaker notes:** This is the elegant systems argument for an AIO board: the governor makes the vibration predictable, the ESC measures the RPM, the notches remove the vibration — one PCB closes the whole loop.

---

## Slide 12 — Making Our PCB a First-Class Rotorflight Board
- **No firmware fork needed** — Unified Target architecture: one stock binary per MCU family
- The board is described by a plain-text **`.config` file** (`IITB-SURPHELI.config`) in the `rotorflight-targets` repo: board name, `resource` pin mappings (SERVO 1–4, MOTOR 1, FREQ, UARTs, LED, ADC), timer/DMA assignments, gyro bus/alignment, defaults
- A merged PR makes the board a flash option **directly in the Configurator**
- **Hard constraint:** servo/motor pins need **independent timers** (cyclic 333 Hz, tail 560 Hz narrow-pulse, ESC DShot all differ) → drives MCU pin selection *during* schematic capture

**Speaker notes:** This de-risks the firmware side of the project almost entirely: hardware effort stays on the PCB, and we inherit a mature, flight-proven control stack.

---

## Slide 13 — Requirements for the AIO FC + ESC PCB
1. **BEC is flight-critical:** 3 swash + 1 tail digital servos, amps each under load → switching BEC **5–8.4 V, ≥10 A cont / 25 A peak**, bulk capacitance, brown-out isolation from the 3V3 rail (BEC sag = crash)
2. **Servo-centric I/O:** 4× servo PWM (per-output rates) + 1 ESC channel (+ optional motorized-tail output), RPM/frequency input, ≥3 UARTs (RX, ESC telemetry, blackbox/GPS)
3. **IMU designed for vibration:** MPU-6000/BMI-class tolerate hard mounting; MPU-6500 needs soft-mount; mechanical isolation + RPM notches
4. **ESC stage (450-class):** 60–80 A cont @ 6S, low-inductance FET layout, phase current + temp sensing, soft spool-up, governor + autorotation-bailout logic, **bidirectional DSHOT / telemetry RPM (mandatory)**
5. **MCU:** STM32 H743 (or F722) — enough independent timers, SPI gyro, blackbox flash → Rotorflight-compatible by construction

**Speaker notes:** Headline: this is *not* a quad FC with fewer motor pads — it's a servo-centric board with a high-current ESC stage and an RPM loop.

---

## Slide 14 — Landscape: Where Rotorflight Sits
| System | Type | Verdict for a custom AIO PCB |
|---|---|---|
| **Rotorflight** | Open source (GPLv3), STM32 | Full FBL + governor + rescue + blackbox; **only option allowing our own board target + integrated ESC** |
| VBar NEO | Commercial, closed | Benchmark feel; locked to Mikado hardware |
| Spirit FBL | Commercial, closed | Popular; proprietary hardware only |
| iKon / Brain2 | Commercial, closed | Full-featured; closed hardware |
| ArduPilot TradHeli | Open source, Pixhawk-class | The autonomy path (GPS missions, auto-autorotation); heavier stack, weaker sport feel; RF has no GPS position modes |
| Skookum | Defunct (~2018) | Historical only |

**Speaker notes:** If autonomy becomes a Phase-3 goal, ArduPilot TradHeli is the migration path; for a custom research FC+ESC today, Rotorflight is the only realistic choice.

---

## Slide 15 — Takeaway & Next Steps
**The aircraft:** holds rotor RPM constant and flies entirely on blade pitch — collective for thrust, cyclic (via swashplate + ~90° phase lag) for attitude, tail pitch for yaw. The FBL controller replaces the flybar; the governor closes the RPM loop.

**The board:** servo-centric — big BEC, 4 servo outputs, one 60–80 A ESC stage with governor + bailout, RPM sensing, vibration-hardened IMU — described to Rotorflight by a single `.config` file.

**Next steps:** MCU + gyro selection → timer/pin map (4 servos + ESC + RPM) → power tree (10 A BEC, 60–80 A stage) → schematic capture with the Rotorflight target drafted alongside.

---

## Key sources (for the references slide)
- Rotorflight: rotorflight.org · github.com/rotorflight (firmware README, Releases.md, targets repo, governor/mixer/RPM-filter/servos docs, DIY-board guide)
- ArduPilot traditional-heli docs (swashplate setup, internal governor)
- SKYbrary (autorotation, rotor configurations) · Wikipedia (swashplate, CCPM, disk loading, NOTAR)
- Hobbywing Platinum V4/V5 specifications · RCHelicopterFun (CCPM, flybarless, gyroscopic precession, tail systems)
- Full cited research briefs: `01_Helicopter_Mechanics_Brief.md`, `02_Rotorflight_Brief.md` (same folder)
