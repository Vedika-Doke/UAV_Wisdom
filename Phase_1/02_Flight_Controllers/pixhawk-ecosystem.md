# Pixhawk Ecosystem

## What Pixhawk is
An **open hardware standards initiative** for autonomous systems, originated at ETH Zurich and stewarded by the **Dronecode Foundation**. The same team also created **MAVLink, PX4, and QGroundControl**.

Pixhawk publishes specifications across three areas:
1. **Autopilot** (Flight Management Units)
2. **Payload** (cameras, gimbals, actuators)
3. **Smart Battery** (BMS with safety/interop)

Vendors (Holybro, mRo, CUAV, Auterion, etc.) manufacture boards that **conform to the FMU standard** — so firmware (PX4/ArduPilot) runs across hardware from different makers.

## FMU versions (autopilot standard)
> *Fill in: FMUv2 → FMUv6X. For each — release year, MCU, sensor suite, supported firmwares, vendors shipping it.*

| FMU rev | MCU | RAM/Flash | IMUs | Typical vendor board |
|---------|-----|-----------|------|----------------------|
| FMUv5 | STM32F765 | | | Pixhawk 4 |
| FMUv5X | STM32F765 (modular) | | | Holybro Pixhawk 5X |
| FMUv6C | STM32H743 | | | Holybro Pixhawk 6C |
| FMUv6X | STM32H753 | | triple-redundant | Holybro Pixhawk 6X, CUAV V6X |

## Why it matters for Phase 2
The **General Commercial-Grade FC** (System 1) should align with FMUv6X-class specs — triple-redundant IMUs, isolated power rails, DroneCAN/CAN-FD support.

## Sources
1. https://pixhawk.org/
2. https://pixhawk.org/standards/
<!-- add vendor pages and FMU spec PDFs as you read them -->
