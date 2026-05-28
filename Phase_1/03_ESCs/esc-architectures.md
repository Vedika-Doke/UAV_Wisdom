# ESC Architectures

## How a BLDC ESC works (1-paragraph version)
A 3-phase inverter (6 MOSFETs in two-per-phase half-bridges) commutates a brushless motor based on rotor position. Modern hobby ESCs detect position **sensorlessly** from back-EMF, sequenced by an MCU (EFM8 / ARM Cortex-M0) running BLHeli/AM32. The FC tells the ESC *how fast* via DShot; the ESC's firmware decides *when to switch each MOSFET*.

## 4-in-1 vs individual
| Aspect | 4-in-1 | Individual |
|--------|--------|-----------|
| Weight | Lower | Higher |
| Wiring | Clean (single connector to FC) | Messy |
| Heat dissipation | Concentrated | Distributed |
| Failure mode | Lose one → lose all 4 | Lose one motor only |
| Repairability | Replace whole board | Swap one ESC |
| Typical use | FPV, small UAVs | Heavy-lift, industrial |

## Key spec dimensions
- Continuous current per channel (A) and burst current
- Voltage range (3S–6S, 6S–12S, HV)
- MOSFET Rds(on) & gate-drive — drives losses
- Switching frequency (typ. 24–48 kHz)
- Telemetry (KISS / BLHeli) over UART
- Heatsinking / capacitor selection (low-ESR for back-EMF spikes)

## Sources
<!-- 1. -->
