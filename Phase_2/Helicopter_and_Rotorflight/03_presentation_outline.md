# Presentation Outline

Slide plan for the helicopter/Rotorflight talk (~15 slides, 12–15 min). Detailed material lives in [01_helicopter_mechanics.md](./01_helicopter_mechanics.md) and [02_rotorflight.md](./02_rotorflight.md).

## Slide plan

**1 · Title** — Single-Rotor Flight: Helicopter Mechanics, Rotorflight, and our AIO FC+ESC PCB. SURP 2026, IIT Bombay.

**2 · Why a helicopter, not another quad** — three wins and one trade:
- Efficiency: one large rotor, low disc loading, hover power scales with √(disc loading)
- Control: collective pitch at constant RPM = millisecond thrust response, including negative thrust
- Safety: autorotation, a controlled power-off landing no multirotor can do
- The trade: swashplate, servos, gears, vibration — complexity that lands on our PCB

**3 · The rotor disc is a rotating wing** — collective vs cyclic, thrust vector tilts with the disc, tail rotor cancels torque (~5–10% of power). Diagram: level disc in hover vs tilted disc in forward flight.

**4 · The swashplate** — two rings on a spherical bearing; raise = collective, tilt = cyclic. eCCPM: three servos, FC does the mixing (120° standard, 135°/140° variants). Diagram: swashplate cutaway.

**5 · The flight controller IS the flybar** — the key slide. Flybar was a mechanical gyroscope; FBL deletes it; a bare FBL head is unstable; gyro + PID at ~1 kHz replaces it. Punchline: the aircraft's stability lives on the PCB.

**6 · Rotor dynamics the controller must respect** — ~90° phase lag (phase angle parameter), dissymmetry of lift and flapping, ground effect (governor transients), autorotation (ESC bailout requirement).

**7 · What the FBL controller computes** — two chains:
- Attitude: RX → rate PID + FF (gyro 8 kHz, PID 1–2 kHz) → CCPM mixer → 4 servos (333/560 Hz)
- Headspeed: RPM sense → governor PID + precomp → ESC → also feeds RPM notch filters
- Why constant RPM + collective instead of throttle: rotor inertia

**8 · Powertrain numbers** — class table (250/450/550/700: headspeed, ESC current, cells). Design anchor: 450 class, ~2700–3400 RPM, 60–80 A on 6S. Heli ESC feature list = our requirements spec (governor, bailout, 10 A BEC, telemetry).

**9 · Rotorflight** — Betaflight 4.3 fork, helicopter-only, GPLv3. Current: RF 2.3 / fw 4.6.0. Ecosystem: Configurator, Blackbox, LUA radio tuning, targets repo. Runs on STM32 F405/F722/F745/H743.

**10 · Heli-specific machinery** — governor modes (OFF/PASSTHROUGH/STANDARD/MODE1/MODE2), tail types + TTA, rescue mode, piro compensation, cyclic ring, collective-to-pitch FF. PID = P·I·D + FF + boost; FF does most of the flying.

**11 · Vibration and RPM notches** — rotor 1/rev (25–70 Hz) sits inside the control band; broadband lowpass would add fatal latency; governed RPM + known gear ratios → narrow tracked notches on main/tail harmonics. Our integrated ESC supplies the RPM for free. Diagram: gyro spectrum with notch markers.

**12 · Our PCB as a Rotorflight board** — no firmware fork; the board is a plain-text `.config` target (resource mappings, timers, gyro bus); merged PR = flash option in the Configurator. Constraint: servo/motor pins on independent timers, decided during schematic capture.

**13 · Board requirements** — the five headline items:
1. BEC 5–8.4 V, ≥10 A cont / 25 A peak, brown-out isolated (flight-critical)
2. 4 servo outputs + 1 ESC channel + RPM input + 3 UARTs
3. IMU chosen and mounted for vibration
4. ESC stage 60–80 A @ 6S with governor, bailout, bidirectional DShot RPM
5. STM32 H743/F722

**14 · Landscape** — Rotorflight vs VBar / Spirit / iKon (closed hardware) vs ArduPilot TradHeli (autonomy path, heavier). Rotorflight is the only one that lets us build our own board.

**15 · Takeaway** — the aircraft: constant RPM, flies on blade pitch, FC replaces the flybar. The board: servo-centric, big BEC, one high-current ESC stage, RPM loop. Next steps: MCU + gyro selection → pin map → power tree → schematic with the RF target drafted alongside.

## Speaker notes

- Slide 5 is the core argument for the whole project; slow down there.
- On slide 7, contrast with a quad explicitly: a quad modulates four motor RPMs, a heli holds one RPM and moves four servos.
- Slide 11 is the elegant systems point: the governor makes vibration predictable, the ESC measures it, the notches remove it — one board closes the loop.
- If asked about autonomy: GPS in Rotorflight is telemetry/logging only; ArduPilot TradHeli is the migration path if that becomes a goal.
