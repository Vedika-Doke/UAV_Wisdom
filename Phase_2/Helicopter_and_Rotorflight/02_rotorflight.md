# Rotorflight Firmware — Technical Briefing
*Prepared for the SURP UAV project designing an AIO (FC + ESC) PCB for a single-rotor helicopter*

---

## 1. What Rotorflight Is

- **Definition:** Rotorflight is an open-source **flight control software suite designed exclusively for single-rotor RC helicopters** — it explicitly does *not* support multirotors or airplanes ([firmware README](https://github.com/rotorflight/rotorflight-firmware/blob/master/README.md)).
- **Origin:** Rotorflight 2 is a **fork of Betaflight 4.3** (lineage: MultiWii → Baseflight → Cleanflight → Betaflight → Rotorflight), also incorporating ideas from **HeliFlight3D**. Because of this it supports "all flight controller boards supported by Betaflight 4.3" that have enough free I/O pins.
- **Versions:** "Rotorflight 2" (RF2) is the current generation and is **not backward compatible with RF1**. Version mapping ([Releases.md](https://github.com/rotorflight/rotorflight-firmware/blob/master/Releases.md)):
  - RF 2.0 → firmware 4.3.x (official release cycle began with snapshots Jan 2024, RC2 Apr 2024)
  - RF 2.1 → firmware 4.4.x
  - RF 2.2 → firmware 4.5.x
  - **RF 2.3 → firmware 4.6.x — current official release: 4.6.0, published June 30, 2026** ([releases](https://github.com/rotorflight/rotorflight-firmware/releases)). 4.6.0 added refactored "Rotorflight Rates," a redesigned governor for nitro/IC engines, battery profiles + SmartFuel charge estimator, ESC forward programming (AM32, BLHeli_S/Bluejay, ZTW, OMP, XDFly), FlySky IBUS2, BMP581 baro and BMI323 gyro drivers, and native ELRS RPM/temperature telemetry.
- **License:** **GPLv3** (firmware, configurator, blackbox, LUA scripts). Docs repo is MIT.
- **Ecosystem** ([github.com/rotorflight](https://github.com/rotorflight)):
  - `rotorflight-firmware` (C, GPL-3.0, ~154★) — the FC firmware
  - `rotorflight-configurator` (JS/NW.js, GPL-3.0) — cross-platform setup/flashing GUI; RF 2.3 Configurator supports firmware 4.6.0
  - `rotorflight-blackbox` (JS, GPL-3.0) — flight log analysis tool
  - `rotorflight-lua-scripts` (EdgeTX/OpenTX) and `rotorflight-lua-ethos-suite` (FrSky Ethos) — full **on-transmitter configuration and tuning** (PIDs, rates, governor) via knobs/switches/LUA pages
  - `rotorflight-targets` — board configs ("custom defaults") loaded live by the Configurator's flasher
  - `rotorflight-presets`, `rotorflight-docs` (Docusaurus site at rotorflight.org)

---

## 2. Helicopter-Specific Features

**Swashplate mixing** ([Mixer docs](https://rotorflight.org/docs/configurator/tabs/mixer), [setup-mixer](https://rotorflight.org/docs/setup/setup-mixer)):
- Electronic CCPM mixing for **120°, 135°, and 140° swashplates** (plus direct/mechanical arrangements via the fully customizable servo/motor mixer). Standard layout: Servo 1 = pitch (centerline), Servo 2 = left, Servo 3 = right, Servo 4 = tail; alternate layouts supported.
- Per-axis geometry tools: **swashplate phase angle**, **collective geometry correction** (equalize +/− collective), swash link trims, and **blade pitch limits**: collective limit (typ. 15–16° for 3D), **cyclic blade pitch limit (~16°) — this is the "cyclic ring"** that constrains combined roll+pitch cyclic, and a **total pitch limit** preventing mechanical binding at any cyclic+collective combination.

**Governor** ([Governor wiki](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/Governor), [Tune-Governor](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Tune-Governor.md)) — maintains constant headspeed independent of load and battery sag:
- **Modes:** **OFF** (throttle passthrough, no functions), **PASSTHROUGH** (no headspeed control but slow spool-up, autorotation bailout, fault recovery), **STANDARD** (PID headspeed control "similar to most ESC governors"), **MODE1** (adds **collective/cyclic precompensation**, like commercial FBL governors), **MODE2** (MODE1 + **battery voltage compensation inside the loop**). 4.6.0 reworked governor support for **nitro/IC engines** (electric and nitro both supported; docs include a dedicated nitro setup page).
- **Gains:** P, I, D, F(feedforward) with **precompensation weights for collective, cyclic, and yaw** (typical 20–100). Key parameters: handover throttle, **spool-up time** (e.g., 10 s to 3000 RPM), tracking time (5–20 s), **autorotation timeout and bailout time** (fast recovery if throttle is restored within the window). Throttle bands: <0% cut, 0–5% idle/autorotation, 5%–handover idle, above handover = governed.

**Tail control:**
- Three tail types: **variable-pitch (servo)**, **motorized tail** (second motor output), and **bidirectional motorized tail** ([Mixer tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Mixer)).
- **TTA — Tail Torque Assist** (a.k.a. TALY) for motorized tails: since a fixed-pitch tail motor has authority in only one direction, TTA **spikes the main motor RPM to generate reaction torque in the other direction**. Tuned in increments of ~10; docs note limits (e.g., OMP M2 can't fully eliminate tail swing even at max gain) ([Motorised-Tail-and-TTA](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Motorised-Tail-and-TTA.md)).
- **Yaw precompensation (feedforward to tail):** separate **collective FF gain**, **cyclic FF gain**, and **collective impulse FF**, plus asymmetric **CW/CCW yaw stop gains** ([Profiles tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Profiles)).

**Rescue mode:** flip-to-upright auto-level, then staged **pull-up / climb / hover collective** (0–100% settings), with flip fail-time cutoff, exit transition time, leveling gain and max rate/acceleration limits. There are also Angle/Horizon/Acro-Trainer stabilization ("6D") modes.

**Piro compensation:** implemented as **Error Rotation** — the PID error/I-term vector is rotated between pitch and roll as the heli pirouettes, so corrections stay earth-referenced during piros ([Profiles tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Profiles)).

**Cross-coupling & feedforward compensations:**
- **Cyclic cross-coupling compensation** — cancels roll reaction to elevator inputs (gain, ratio, cutoff-frequency parameters).
- **Collective-to-pitch feedforward** — pre-compensates the nose-pitch reaction from rotor drag when climbing.
- **High-Speed Integral (HSI/offset)** — extra integral offset compensation for high-airspeed flight (pid_mode 3).

**Filtering** ([RPM-Filters](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/RPM-Filters)):
- **RPM notch filtering**: real-time motor RPM (from bidirectional DSHOT, ESC telemetry, or a dedicated frequency sensor) is converted through configured **main/tail gear ratios** into rotor frequencies; notch filters track **1st (double notch), 2nd, and 3rd harmonics** of main and tail rotors with adjustable Q. Result: much cleaner gyro signal → higher usable gains.
- Plus **FFT-based dynamic notch filters** and dynamic lowpass, inherited/adapted from Betaflight.

---

## 3. Supported Hardware

- **MCUs:** **STM32F405, STM32F722, STM32F745, STM32H743** (STM32F411 and G474 supported legacy, being phased out) ([README](https://github.com/rotorflight/rotorflight-firmware/blob/master/README.md)). Firmware target tree includes MATEKF405/F411/F722/H743, NUCLEO F722/H743, SITL (software-in-the-loop), and **STM32_UNIFIED** targets.
- **Purpose-built Rotorflight FCs documented:** Matek G474-HELI / H743-HLite, FlyWing F405 Heli, FlyDragon, RadioMaster Nexus, FrSky 007, Goosky, plus any Betaflight 4.3 board with ≥3–4 free servo outputs.
- **Receivers:** CRSF (incl. **ELRS**), S.BUS, FBUS, F.Port, SRXL2, IBUS/IBUS2, XBUS, EXBUS, GHOST, CPPM; extensive telemetry back to the radio (native ELRS RPM/temp telemetry since 4.6.0).
- **ESC telemetry protocols** ([ESC-Telemetry](https://rotorflight.org/docs/setup/esc-telemetry)): **BLHeli32/KISS, Hobbywing Platinum V4/V4.1/FlyFun V5 and Platinum V5, Scorpion (UNSC), Kontronik (Kosmik/Kolibri), Castle, OMPHobby, ZTW Skyhawk, APD HV Pro, YGE (OpenYGE), XDFly, Graupner, FLYROTOR** — providing RPM, voltage, current, temperature, mAh over a single UART RX pin. ESC **forward programming** from the Configurator for AM32, BLHeli_S/Bluejay, ZTW, OMP, XDFly.
- **Servos** ([Servos tab](https://www.rotorflight.org/docs/2.0.0/Wiki/Configurator/Servos)): per-servo PWM rate and pulse geometry — **cyclic digital servos typically 333 Hz @ 1520 µs center (±700 µs)**; **narrow-pulse tail servos 560 Hz @ 760 µs center (±350 µs)**; fully adjustable min/max/scale/reverse and servo override for bench setup.
- **RPM input:** bidirectional DSHOT, ESC telemetry RPM, or a dedicated **frequency/RPM sensor pin** (e.g., magnetic/backEMF sensor for nitro).
- **Loop rates:** gyro sampled at native rate (typically 8 kHz); **PID loop recommended 1–2 kHz** (higher gives no benefit for helis and just loads the MCU) ([Configuration tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Configuration)).

---

## 4. Making a Custom Board Target (directly relevant to the AIO PCB)

Rotorflight 2 inherits Betaflight 4.3's **Unified Target** architecture — very good news for a custom PCB, since you likely need **no firmware fork at all** if you use a supported MCU:

1. **Pick a supported MCU:** STM32F405, F722, F745, or H743 (H743 recommended for headroom; F411 is being dropped). The firmware ships one unified binary per MCU family (STM32_UNIFIED targets).
2. **Write a board config** ("custom defaults"): a plain-text `.config` file named `MFGID-BOARDNAME.config` (e.g., `MTKS-MATEKH743.config`) in the [rotorflight-targets](https://github.com/rotorflight/rotorflight-targets) repo's `configs/` folder. It contains CLI directives: `board_name`, `manufacturer_id`, **`resource` mappings** (which MCU pin = MOTOR n, SERVO n, UART n, LED, ADC, beeper…), **`timer` and `dma` assignments**, gyro alignment/bus definitions, and `set` defaults. The **Configurator's flasher loads targets directly from that repo**, so a merged PR makes your board a first-class flash option; locally you can just paste the config into the CLI.
3. **Resource remapping is fully CLI-driven** ([betaflight-diy guide](https://rotorflight.org/docs/controllers/betaflight-diy)): e.g., `resource MOTOR 1 B07`, `resource SERVO 1 A00`, freeing unused motor pads as servo outputs. Key: pins used for servos/motors must sit on **independent timers** (PWM rates differ per output: 333 Hz cyclic, 560 Hz tail, DSHOT/PWM for ESC).
4. **Hardware checklist for an AIO heli FC** (from the DIY-board and controller docs):
   - ≥4 motor/servo capable timer pins (3 cyclic servos + tail servo, plus 1–2 motor outputs for main/tail ESC)
   - **5 V BEC with real current headroom for servos** (cyclic servos are the big load; a small drone-style 2 A BEC only suits micro helis — the DarwinFPV 15 A AIO example is cited as micro-heli-viable)
   - Several UARTs: receiver (CRSF/S.BUS) + ESC telemetry + optional GPS/blackbox (OpenLager)
   - Gyro choice matters: **MPU-6000/BMI-class tolerate hard mounting; MPU-6500 is vibration-sensitive and needs soft-mounting** — on a heli AIO expect severe vibration, so gyro selection + mechanical isolation is a first-order design decision
   - Blackbox flash/SD strongly recommended (tuning workflow depends on it)
   - For the integrated ESC: implementing **bidirectional DSHOT or a telemetry UART with RPM** is essentially mandatory, because the RPM filter and governor are built around live RPM.

---

## 5. PID Architecture for Helis vs. Multirotor Betaflight

([Profiles docs](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Profiles), [Tuning guide](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Tuning-description.md))

- Per-axis **P, I, D + Feedforward + Boost (B-term)**: FF is proportional to stick input — on a heli, cyclic deflection (not motor thrust differential) commands rate, so **FF does most of the flying and PID trims the error**; the tuning target is "I stays near zero through full-stick flips/rolls." Boost is a setpoint-derivative kick for sharper response (typ. 20–50, cutoff 10–20 Hz).
- **Helicopter-specific terms absent in multirotor Betaflight:** Error Rotation (piro comp), ground error decay (no I-windup on skids), I-term relax per axis, HSI offset gains, collective-to-pitch FF, cyclic cross-coupling comp, yaw collective/cyclic FF precomp, CW/CCW yaw stop gains.
- **Rate architecture:** actuators are servos, not ESCs — so gyro runs at ~8 kHz but the **PID loop runs at 1–2 kHz** and outputs at **servo rates (333/560 Hz)**, versus Betaflight's 4–8 kHz PID → multi-kHz DSHOT. Latency budget and filtering philosophy therefore differ: heavy reliance on deterministic **RPM notches** (rotor harmonics are known) rather than aggressive broadband lowpass.
- Big mechanical time-constants (rotor disk) mean D is less dominant and the governor + precomp handle power coupling that multirotors don't have.

---

## 6. Comparison to Alternatives

| System | Type | Notes |
|---|---|---|
| **Rotorflight** | Open source (GPLv3), runs on cheap STM32 FCs | Full FBL + governor + rescue + blackbox + radio-LUA tuning; only OSS option with modern heli feature parity; you can build your own board — the only realistic choice for a custom AIO PCB |
| **Spirit FBL** | Commercial, closed | Popular mid/high-end FBL + governor + rescue; proprietary hardware only |
| **VBar (Mikado VBar Control/NEO)** | Commercial, closed | Premium ecosystem tightly integrated with Mikado radios; industry benchmark feel; expensive, closed |
| **Skookum (SK-540/720)** | Commercial — **defunct** (company closed ~2018) | Historical reference only |
| **iKon / MSH Brain2** | Commercial, closed | Full-featured FBL with app-based setup, rescue, governor |
| **ArduPilot Copter (TradHeli)** | Open source (GPLv3) | The other OSS heli option — strong at **autonomous** flight (GPS missions, autonomous autorotation, waypoints) on Pixhawk-class hardware, but heavier stack, weaker 3D/sport-flying feel and community tuning support than Rotorflight; Rotorflight has no GPS position hold/missions (GPS is telemetry/logging only) |

**Positioning:** Rotorflight is effectively "Betaflight for helicopters" — it brought the open-source, blackbox-driven, community-tuned FPV workflow to single-rotor helis, at 1/3–1/10 the hardware cost of commercial FBL units, and it is the only heli firmware where a university team can define its own board target and integrate an ESC on the same PCB.

**Key sources:** [rotorflight-firmware README](https://github.com/rotorflight/rotorflight-firmware/blob/master/README.md) · [Releases.md](https://github.com/rotorflight/rotorflight-firmware/blob/master/Releases.md) · [GitHub org](https://github.com/rotorflight) · [Governor wiki](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/Governor) · [Profiles tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Profiles) · [Mixer tab](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Mixer) · [RPM filters](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/RPM-Filters) · [ESC telemetry](https://rotorflight.org/docs/setup/esc-telemetry) · [Servos](https://www.rotorflight.org/docs/2.0.0/Wiki/Configurator/Servos) · [DIY boards](https://rotorflight.org/docs/controllers/betaflight-diy) · [rotorflight-targets](https://github.com/rotorflight/rotorflight-targets) · [TTA tuning](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Motorised-Tail-and-TTA.md)
