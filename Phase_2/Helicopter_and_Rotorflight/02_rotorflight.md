# Rotorflight

Notes on the Rotorflight firmware ecosystem and what it takes to run it on a board of our own design.

## 1. What it is

Rotorflight is an open-source flight control suite built exclusively for single-rotor RC helicopters. It does not support multirotors or planes. Rotorflight 2 is a fork of Betaflight 4.3 (the lineage runs MultiWii → Baseflight → Cleanflight → Betaflight → Rotorflight), with ideas borrowed from HeliFlight3D. Because of that ancestry it runs on more or less any Betaflight-4.3-capable board that has enough free pins.

Everything is GPLv3. Version numbers map like this: RF 2.0 = firmware 4.3.x, RF 2.1 = 4.4.x, RF 2.2 = 4.5.x, and the current release RF 2.3 = firmware 4.6.0 (June 2026). The 4.6.0 release added a reworked rates system, a redesigned governor with proper nitro/IC support, battery profiles, ESC forward programming (AM32, BLHeli_S/Bluejay, ZTW, OMP, XDFly), and native ELRS RPM/temperature telemetry.

The ecosystem around the firmware:

- **Configurator** — cross-platform GUI for setup and flashing
- **Blackbox** — log analysis tool; the whole tuning workflow revolves around it
- **LUA script suites** — full configuration and PID/rates/governor tuning from the transmitter (EdgeTX/OpenTX and FrSky Ethos)
- **rotorflight-targets** — repository of board configs that the Configurator's flasher loads directly
- Docs live at rotorflight.org, source at github.com/rotorflight

## 2. Helicopter-specific features

**Swashplate mixing.** Electronic CCPM for 120°, 135°, and 140° swashplates, plus a fully customizable mixer for anything else. On top of the geometry there are per-axis tools: swash phase angle, collective geometry correction, link trims, and blade pitch limits — a collective limit (15–16° is typical for 3D), a cyclic ring that caps combined roll+pitch cyclic, and a total pitch limit so no stick combination can bind the head mechanically.

**Governor.** Five modes: OFF (throttle passthrough), PASSTHROUGH (no headspeed control, but still handles slow spool-up, autorotation bailout, and fault recovery), STANDARD (PID headspeed control, comparable to an ESC governor), MODE1 (adds collective/cyclic precompensation like the commercial FBL governors), and MODE2 (MODE1 plus battery voltage compensation inside the loop). Gains are P/I/D/F plus precomp weights for collective, cyclic, and yaw. The important timing parameters are spool-up time, tracking time, and the autorotation timeout/bailout window that allows a fast re-spool if the pilot aborts an auto.

**Tail.** Three tail types are supported: variable-pitch (servo), motorized, and bidirectional motorized. For fixed-pitch motorized tails there is TTA (tail torque assist), which briefly spikes main motor RPM to generate reaction torque in the direction the tail motor can't push. Yaw also gets feedforward precompensation from collective and cyclic, with separate CW/CCW stop gains.

**Rescue.** A flip-to-upright mode followed by staged pull-up, climb, and hover collective, with tunable rates and timeouts. Angle/Horizon/Acro-Trainer stabilization modes also exist.

**Piro compensation.** Implemented as error rotation: the PID error and I-term vector are rotated between pitch and roll as the heli pirouettes, so corrections stay earth-referenced.

**Cross-coupling and feedforward.** Cyclic cross-coupling compensation (cancels the roll reaction to elevator inputs), collective-to-pitch feedforward (cancels the nose-pitch reaction when climbing), and a high-speed integral offset for fast forward flight.

**Filtering.** The RPM filter converts live motor RPM through the configured main/tail gear ratios into exact rotor frequencies and places tracking notch filters on the first (double notch), second, and third harmonics of both rotors. FFT-based dynamic notches and dynamic lowpass are inherited from Betaflight. The result of clean gyro data is simply higher usable gains.

## 3. Supported hardware

- **MCUs:** STM32F405, F722, F745, H743 (F411 and G474 are legacy and being phased out).
- **Receivers:** CRSF/ELRS, S.BUS, FBUS, F.Port, SRXL2, IBUS/IBUS2, XBUS, EXBUS, GHOST, CPPM.
- **ESC telemetry:** BLHeli32/KISS, Hobbywing Platinum V4/V5, Scorpion, Kontronik, Castle, OMPHobby, ZTW, APD, YGE, XDFly, Graupner, FLYROTOR — RPM, voltage, current, temperature over a single UART pin.
- **Servos:** per-servo PWM rate and pulse geometry. Cyclic digital servos typically run 333 Hz with 1520 µs centers; narrow-pulse tail servos run 560 Hz with 760 µs centers.
- **RPM input:** bidirectional DSHOT, ESC telemetry, or a dedicated frequency-sensor pin (used on nitro).
- **Loop rates:** gyro at its native rate (typically 8 kHz), PID loop at 1–2 kHz — the docs are explicit that higher PID rates buy nothing on a heli, since the actuators are servos updating at 333/560 Hz.

Purpose-built Rotorflight FCs already on the market include the Matek G474-HELI and H743-HLite, FlyWing F405 Heli, RadioMaster Nexus, and FrSky 007 — useful as reference designs.

## 4. Running it on our own board

This is the part that makes Rotorflight the right choice for the project. RF2 inherits Betaflight 4.3's unified target architecture, so with a supported MCU there is no firmware fork at all:

1. Pick a supported MCU — H743 has the most headroom; F722 is the budget option.
2. Describe the board in a plain-text config file (`MFGID-BOARDNAME.config`, e.g. `IITB-SURPHELI.config`) containing the board name, `resource` pin mappings (SERVO n, MOTOR n, UARTs, LED, ADC, beeper), timer and DMA assignments, gyro bus and alignment, and default settings.
3. Submit it to the rotorflight-targets repo. The Configurator's flasher loads targets from that repo, so once merged, our board becomes a normal flash option. Before that, the same config can just be pasted into the CLI.

Hardware checklist pulled from the DIY-board docs:

- At least 4 servo/motor-capable pins on independent timers (the outputs run at different rates: 333 Hz cyclic, 560 Hz tail, DSHOT/PWM for the ESC) — this constrains MCU pin selection and should be settled during schematic capture, not after
- A 5 V BEC with real current headroom for cyclic servo loads (drone-style 2 A BECs are only viable on micro helis)
- Several UARTs: receiver, ESC telemetry, optionally GPS or external blackbox
- Gyro choice with vibration in mind: MPU-6000 and BMI-class parts tolerate hard mounting, MPU-6500 needs a soft mount
- Blackbox flash or SD strongly recommended
- For the integrated ESC stage, bidirectional DSHOT or a telemetry UART with RPM is effectively mandatory, since the governor and RPM filters are built around live RPM

## 5. PID architecture vs multirotor Betaflight

Each axis runs P, I, D plus feedforward and a "boost" term (a setpoint-derivative kick). The balance is different from a quad: on a heli, cyclic deflection commands rate almost directly, so feedforward does most of the flying and PID trims the residual error. The stated tuning goal is that the I-term stays near zero through full-stick flips and rolls.

Several terms have no multirotor equivalent: error rotation for pirouettes, ground error decay (no I-windup while sitting on skids), collective-to-pitch feedforward, cyclic cross-coupling compensation, yaw precomp with separate CW/CCW stop gains, and the high-speed integral offset.

The filtering philosophy differs too. A quad leans on aggressive broadband lowpass and multi-kHz loops; a heli has known, governed rotor frequencies, so Rotorflight leans on precise RPM-tracked notches with a modest 1–2 kHz PID loop feeding servo-rate outputs.

## 6. Alternatives

| System | Type | Notes |
|--------|------|-------|
| Rotorflight | Open source (GPLv3), STM32 | Full FBL + governor + rescue + blackbox + radio LUA tuning. The only option where we can define our own board target and integrate the ESC. |
| VBar NEO | Commercial, closed | Industry benchmark feel; tied to Mikado hardware and radios. |
| Spirit FBL | Commercial, closed | Popular mid/high-end unit; proprietary hardware only. |
| iKon / MSH Brain2 | Commercial, closed | Full-featured, app-based setup; closed hardware. |
| ArduPilot TradHeli | Open source, Pixhawk-class | The autonomy route: GPS missions, autonomous autorotation. Heavier stack, weaker sport-flying feel. Rotorflight has no GPS position modes (GPS is telemetry/logging only). |
| Skookum | Defunct (~2018) | Historical reference. |

The short version: Rotorflight brought the open, blackbox-driven Betaflight workflow to helicopters at a fraction of commercial FBL prices, and it is the only heli firmware where a university team can make its own hardware a first-class citizen. If autonomy becomes a later goal, ArduPilot TradHeli is the migration path.

## References

- [rotorflight-firmware README](https://github.com/rotorflight/rotorflight-firmware/blob/master/README.md) · [Releases](https://github.com/rotorflight/rotorflight-firmware/blob/master/Releases.md) · [GitHub org](https://github.com/rotorflight)
- [Governor](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/Governor) · [Profiles](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Profiles) · [Mixer](https://rotorflight.org/docs/2.0.0/Wiki/Configurator/Mixer) · [RPM filters](https://rotorflight.org/docs/2.0.0/Wiki/Tutorial-Setup/RPM-Filters)
- [ESC telemetry](https://rotorflight.org/docs/setup/esc-telemetry) · [Servos](https://www.rotorflight.org/docs/2.0.0/Wiki/Configurator/Servos)
- [DIY / Betaflight boards guide](https://rotorflight.org/docs/controllers/betaflight-diy) · [rotorflight-targets](https://github.com/rotorflight/rotorflight-targets)
- [TTA tuning](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Motorised-Tail-and-TTA.md) · [Governor tuning](https://github.com/rotorflight/rotorflight-docs/blob/main/docs/Tuning/Tune-Governor.md)
