# Helicopter Mechanics & Rotorflight

Research notes for the single-rotor helicopter platform, feeding into the AIO FC+ESC PCB design.

| # | Topic | Status |
|---|-------|:------:|
| [01](./01_helicopter_mechanics.md) | Helicopter mechanics — rotor disc, swashplate, FBL, rotor dynamics, powertrain | 🟡 |
| [02](./02_rotorflight.md) | Rotorflight — features, supported hardware, custom board targets, PID architecture | 🟡 |
| [03](./03_presentation_outline.md) | Presentation outline | 🟡 |

## What this means for the board

- A flybarless heli cannot fly without electronic stabilization — the flight controller replaces the mechanical flybar.
- The heli flies on blade pitch at constant governed RPM, so the board is servo-centric: 4 servo outputs, a ≥10 A BEC, one high-current ESC stage, and RPM sensing.
- Rotorflight turns a custom board into a plain-text `.config` target on a stock STM32 binary — no firmware fork.
- Rotor vibration (1/rev ≈ 25–70 Hz) sits inside the control band; governed RPM plus known gear ratios make narrow tracked notch filters possible.
