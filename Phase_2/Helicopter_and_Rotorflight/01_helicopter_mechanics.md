# How Single-Rotor Helicopters Work

Notes on helicopter flight mechanics, written while shifting the project to a single-rotor platform. The last section collects what all of this means for our AIO FC+ESC board.

## 1. Basics

A helicopter rotor is best thought of as a rotating wing. Each blade is an airfoil, and the circle swept by the blades (the "rotor disc") produces a thrust vector roughly perpendicular to the disc plane. Tilting the disc tilts the thrust vector, which is how the helicopter moves around.

There are two ways to change blade pitch, and they do different jobs:

- **Collective** changes the pitch of all blades by the same amount at the same time. This changes total thrust, so it handles climb and descent. RC helis also use negative collective for inverted flight.
- **Cyclic** varies each blade's pitch sinusoidally once per revolution, so one side of the disc produces more lift than the other. This tilts the disc and gives pitch, roll, and translation.

Spinning the rotor also torques the fuselage in the opposite direction (Newton's third law), so something has to cancel that torque. The usual answer is a tail rotor: a small side-thrusting propeller on the boom whose variable pitch doubles as yaw control. It eats roughly 5–10% of total power while producing no lift, which is the price of the configuration. Alternatives exist (NOTAR blows air along the tailboom and uses the Coandă effect, coaxial and tandem layouts cancel torque with a second rotor), but single main rotor plus tail rotor is the standard RC layout and the one we are designing for.

## 2. The swashplate

The swashplate carries control inputs from the non-rotating airframe into the rotating rotor head. It consists of two rings on a spherical bearing around the main shaft. The lower ring does not rotate and is pushed and tilted by the servos; the upper ring rotates with the head and drives pitch links up to the blade grips.

Raising the whole plate changes all blades together, which is collective. Tilting the plate makes each blade's pitch vary as it rides around, which is cyclic. One mechanism, both controls.

Older helis used mechanical mixing (mCCPM), with one servo per function connected through levers that summed the motions. Modern helis use electronic mixing (eCCPM): three servos attach directly to the swashplate at fixed positions and the flight controller does the mixing in software. The common geometry is 120° spacing between servos; the 135°/140° variants adjust the elevator servo position so both side servos see the same ratio, and 90°/H1 is legacy. ArduPilot and Rotorflight both support these natively.

The old stabilization device was the flybar (Bell-Hiller): a weighted paddle bar that acts as a mechanical gyroscope and feeds a stabilizing cyclic input into the head. It works, but costs drag, head complexity, and agility. Modern helis are flybarless (FBL), meaning the bar is simply gone. The catch is that a bare FBL rotor head is dynamically unstable. A 3-axis MEMS gyro and a PID loop have to replicate the flybar's damping in software, correcting the swash servos on the order of a thousand times per second.

This point matters for us more than anything else in these notes: a flybarless helicopter cannot fly at all without electronic stabilization. The flight controller is not an accessory here, it is the flybar.

## 3. Rotor dynamics worth knowing

**Phase lag.** A cyclic input applied at one point in the rotation produces maximum blade displacement about 90° later in the direction of rotation. It is often called gyroscopic precession, though for a flapping rotor it is really a resonance effect (blade flapping at 1/rev is a driven oscillator at resonance). With offset or stiff hinges the lag is somewhat less than 90°. Swashplate geometry pre-compensates for it, and FBL firmware exposes it as a "phase angle" parameter.

**Dissymmetry of lift.** In forward flight the advancing blade sees airspeed plus rotational speed while the retreating blade sees the difference, so lift is unequal across the disc. Blades flap up on the advancing side and down on the retreating side, which changes their local angle of attack and evens the lift back out. Without flapping freedom the aircraft would roll in forward flight. Retreating-blade stall is what ultimately limits forward speed.

**Ground effect.** Within about one rotor diameter of the surface, induced downwash is reduced and the rotor needs less power for the same thrust. In practice this shows up as governor load transients during takeoff and landing.

**Autorotation.** If power is lost, the pilot immediately lowers collective and the descent drives air upward through the disc. On the inboard part of the blades the resultant aerodynamic force tilts forward of the rotation axis and keeps the rotor spinning. The stored rotational energy is then traded in a final collective flare for a soft touchdown. RC helis with negative collective autorotate routinely, and it is a real safety capability that no multirotor has. The ESC-side implication: heli ESCs implement "autorotation bailout," a fast re-spool from idle in case the pilot aborts the auto.

## 4. What the FBL controller actually does

A modern FBL unit (BeastX, VBar, Spirit, iKon, or Rotorflight as the open-source option) runs four jobs:

1. Rate stabilization on roll, pitch, and yaw. Stick inputs command angular rates; the PID loop drives the swashplate servos. On a heli, feedforward does most of the flying because cyclic deflection maps directly to rate, and PID trims the error.
2. Tail heading hold: an integrating yaw loop that locks heading against the torque transients from collective and power changes. This used to be a separate "heading hold gyro" box.
3. A governor that holds rotor RPM constant regardless of load and battery sag, either in the ESC or in the FBL. FBL governors can feed-forward from collective for faster response. Either way it needs an RPM signal.
4. Gyro filtering tuned around the rotor's known vibration frequencies (see section 7).

Why constant RPM at all? A multirotor changes thrust by changing prop RPM, which works because its props are tiny and low inertia. A helicopter rotor has enormous rotational inertia and RPM changes are far too slow to control with. So the rotor is held at constant speed and thrust is changed nearly instantly through blade pitch. Two useful side effects: vibration frequencies stay fixed, which makes filtering easier, and the rotor always carries stored energy for autorotation and aggressive maneuvers.

## 5. Electric powertrain numbers

One motor drives everything: motor pinion into a large main gear (typical reduction around 8–12:1) onto the main shaft. On most helis above roughly the 380 class the same motor also drives the tail through a torque tube or belt. Torque tubes are efficient and keep tail authority in autorotation, but their gears are fragile in crashes; belts are cheap and crash-tolerant but need tension maintenance. Separate motorized tails only really work on micro helis and lose tail authority in autorotation.

Typical figures by size class:

| Class | Headspeed | ESC continuous | Battery |
|-------|-----------|----------------|---------|
| 250 | ~3500–4500 RPM | 20–40 A | 3–4S |
| 450 | ~2700–3400 RPM | 60–80 A | 6S |
| 550 | ~2200–2800 RPM | 100–120 A | 6–12S |
| 700 | ~1300–2200 RPM | 120–160 A | 12–14S |

Tail rotors spin around 4–6× headspeed. Our design anchor is the 450 class: roughly 60–80 A continuous on 6S.

A helicopter ESC is not a quad ESC with bigger FETs. The de facto benchmark is the Hobbywing Platinum series, and its feature list is effectively a requirements spec: governor mode with a slow soft-start spool-up, autorotation bailout, a switching BEC at 5–8 V with about 10 A continuous / 20–25 A peak for servo loads, and telemetry (RPM, voltage, current, temperature).

## 6. Helicopter vs multirotor, briefly

| | Single-rotor heli | Multirotor |
|---|---|---|
| Hover efficiency | One large rotor, low disc loading, less power per kg | Small fast props, higher disc loading |
| Thrust control | Blade pitch at constant RPM, millisecond response, can go negative | RPM modulation, thrust ≥ 0 |
| Power failure | Autorotation | Crash (on a quad) |
| Mechanics | Swashplate, linkages, gears, heavy vibration | Almost no moving parts |
| Electronics | 3–4 servos, RPM loop, serious gyro filtering | 4 ESC outputs, simpler filtering |

The multirotor won the market on mechanical simplicity. The helicopter wins on physics: efficiency, instant bidirectional thrust, and a real engine-out mode. The complexity it keeps is exactly the complexity our PCB has to serve.

## 7. Implications for the AIO FC+ESC board

This is where the board diverges from a standard multirotor AIO.

1. **The BEC is flight-critical.** Three swash servos plus a tail servo, all high-torque digitals, can each draw several amps under load. Budget a switching BEC at 5–8.4 V, at least 10 A continuous with 20–25 A peaks, with bulk capacitance and brown-out isolation from the MCU's 3V3 rail. A BEC sag on a heli is an instant crash.
2. **Servo outputs, not motor pads.** We need four servo PWM outputs (cyclic servos typically run at 333 Hz; tail servos often use 560 Hz narrow-pulse framing) plus one ESC drive channel, and optionally a second motor output for a motorized tail on small airframes.
3. **RPM sensing.** The governor and the RPM filters both depend on live motor RPM. From our own ESC stage this comes free (BEMF zero-crossing count or bidirectional DShot); an external RPM input pin is worth adding anyway.
4. **Vibration is the environment, not an edge case.** Main rotor 1/rev lands at about 25–70 Hz for typical headspeeds, with blade-pass harmonics, tail frequencies, and motor frequency above it, all inside the control band. Broadband low-pass filtering would add too much latency, so the plan is mechanical isolation of the IMU plus RPM-referenced notch filters. Because the governor holds headspeed constant and the gear ratios are known, the notches can be narrow and accurate. Gyro selection matters too: MPU-6000/BMI-class parts tolerate hard mounting, MPU-6500 does not.
5. **ESC stage.** For the 450-class target: 60–80 A continuous on 6S, low-inductance FET layout, phase current and temperature sensing, soft spool-up, governor and bailout logic, and RPM telemetry back to the FC.
6. **MCU.** An STM32 F722 or H743 with enough independent timers for four servos, the motor output, and the RPM input makes the board directly Rotorflight-compatible, which gets us a mature heli control stack without writing one.

In one sentence: this is not a quad FC with fewer motors, it is a servo-centric board with a big BEC, one high-current ESC stage, an RPM loop, and a filtering chain built around known rotor frequencies.

## References

- [Pilot Mall — helicopter aerodynamics explained](https://www.pilotmall.com/blogs/news/how-do-helicopters-fly-the-aerodynamics-explained)
- [SKYbrary — rotor system configurations](https://skybrary.aero/articles/helicopter-rotor-systems-configuration) · [autorotation](https://skybrary.aero/articles/autorotation)
- [Wikipedia — swashplate](https://en.wikipedia.org/wiki/Swashplate_(aeronautics)) · [CCPM](https://en.wikipedia.org/wiki/Cyclic/collective_pitch_mixing) · [disk loading](https://en.wikipedia.org/wiki/Disk_loading) · [NOTAR](https://en.wikipedia.org/wiki/NOTAR)
- [RCHelicopterFun — CCPM](https://www.rchelicopterfun.com/ccpm.html) · [flybarless](https://www.rchelicopterfun.com/flybarless.html) · [gyroscopic precession](https://www.rchelicopterfun.com/gyroscopic-precession.html) · [tail systems](https://www.rchelicopterfun.com/rc-helicopter-rudder.html)
- [ArduPilot — swashplate setup](https://ardupilot.org/copter/docs/traditional-helicopter-swashplate-setup.html) · [internal governor](https://ardupilot.org/copter/docs/traditional-helicopter-internal-rsc-governor.html)
- [Hobbywing Platinum 80A V5](https://www.hobbywing.com/en/news/info/34) · [Platinum 120A V4](https://www.hobbywing.com/en/products/platinum-120a-v471)
- [Rotorflight firmware](https://github.com/rotorflight/rotorflight-firmware) · [RPM filters](https://rotorflight.org/docs/2.1.0/setup/rpm-filters) · [DIY board requirements](https://rotorflight.org/docs/next/controllers/betaflight-diy)
- [Krossblade — disc loading and hover efficiency](https://www.krossblade.com/disc-loading-and-hover-efficiency)
