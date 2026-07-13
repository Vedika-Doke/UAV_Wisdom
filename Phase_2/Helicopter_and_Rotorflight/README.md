# Helicopter Mechanics & Rotorflight

Research notes for the single-rotor helicopter platform and the Rotorflight firmware ecosystem, feeding directly into the AIO FC+ESC PCB design.

| # | Topic | Status |
|---|-------|:------:|
| [01](./01_helicopter_mechanics.md) | Helicopter mechanics — rotor disc, swashplate/CCPM, FBL, rotor dynamics, powertrain, PCB implications | 🟡 |
| [02](./02_rotorflight.md) | Rotorflight firmware — features, supported hardware, custom board targets, PID architecture, alternatives | 🟡 |
| [03](./03_presentation_outline.md) | Presentation outline — slide-by-slide content with speaker notes | 🟡 |

## Key takeaways for the board

- A flybarless heli **cannot fly without electronic stabilization** — the flight controller replaces the mechanical flybar.
- The heli flies on **blade pitch at constant governed RPM**, so the board is **servo-centric**: 4 servo outputs, ≥10 A BEC, one high-current ESC stage, RPM sensing.
- Rotorflight makes a custom board a **plain-text `.config` target** (STM32 F405/F722/F745/H743) — no firmware fork needed.
- Vibration at rotor 1/rev (~25–70 Hz) sits in the control band; governed RPM + gear ratios enable **RPM-tracked notch filters**.
