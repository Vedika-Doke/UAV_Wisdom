# How Single-Rotor Helicopters Work — Technical Briefing
### For the IIT Bombay SURP UAV project: AIO Flight Controller + ESC PCB for an electric single-rotor helicopter

---

## 1. Fundamentals

**The rotor disc is a rotating wing.** Each blade is an airfoil; spinning it creates relative airflow, and blade pitch (angle of attack) determines lift. The swept circle of the blades is treated aerodynamically as a "rotor disc" that produces a thrust vector roughly perpendicular to the disc plane. Tilt the disc → tilt the thrust vector → the helicopter translates ([Pilot Mall — helicopter aerodynamics](https://www.pilotmall.com/blogs/news/how-do-helicopters-fly-the-aerodynamics-explained)).

**Collective vs cyclic pitch:**
- **Collective** changes the pitch of *all* blades *equally, simultaneously* → changes total thrust magnitude (climb/descend, and in RC helis, negative pitch for inverted flight).
- **Cyclic** changes each blade's pitch *sinusoidally once per revolution* (more pitch on one side of the disc, less on the other) → tilts the disc → pitch/roll and forward/sideways translation.

**Why torque needs canceling:** Newton's third law — the airframe applies torque to spin the rotor, so the rotor applies equal and opposite torque to the fuselage, which would spin it in the opposite direction. Anti-torque options ([SKYbrary rotor configurations](https://skybrary.aero/articles/helicopter-rotor-systems-configuration)):

| Option | How | Trade-off |
|---|---|---|
| **Tail rotor** | Side-thrusting propeller on a boom; variable pitch (or variable RPM on micro helis) gives yaw control | Consumes ~5–10 % of power producing no lift; ground hazard; standard on RC helis |
| **NOTAR** | Fan in the tailboom blows low-pressure air out of slots; Coandă effect bends main-rotor downwash around the boom to create side force, plus a direct jet thruster for yaw | Quieter/safer but less efficient than a tail rotor ([NOTAR — Wikipedia](https://en.wikipedia.org/wiki/NOTAR), [Pilots Who Ask Why](https://pilotswhoaskwhy.com/2021/10/17/how-exactly-does-the-notar-system-work-and-what-are-the-coanda-and-magnus-effects/)) |
| **Coaxial** | Two counter-rotating rotors on one shaft; torques cancel; yaw by differential torque | All power goes to lift, compact; complex head, rotor-rotor interference ([Coaxial-rotor aircraft — Wikipedia](https://en.wikipedia.org/wiki/Coaxial-rotor_aircraft)) |
| **Tandem** | Two counter-rotating rotors fore and aft (CH-47 style) | Wide CG range, all power to lift; two big rotors + complex transmission ([Tandem-rotor — Wikipedia](https://en.wikipedia.org/wiki/Tandem-rotor_aircraft)) |

For this project the relevant architecture is **single main rotor + tail rotor**, the dominant RC configuration.

---

## 2. Swashplate Mechanics

**The swashplate** is the rotating-to-non-rotating interface. It has two rings on a spherical bearing around the main shaft: the lower ring is non-rotating and pushed/tilted by servos; the upper ring rotates with the head and drives pitch links to the blade grips. Raising the whole plate = collective; tilting it = cyclic (pitch varies sinusoidally as each blade rides around the tilted plate) ([Swashplate — Wikipedia](https://en.wikipedia.org/wiki/Swashplate_(aeronautics))).

**Mechanical mixing (mCCPM, obsolete):** one servo per function (collective, aileron, elevator) connected through levers/bellcranks that mechanically sum motions. Heavy, sloppy, many linkages.

**Electronic mixing (eCCPM):** 3 (sometimes 4) servos attach *directly* to the swashplate at fixed angular positions; the FBL unit (or radio) does the mixing math in software. Any stick input becomes a coordinated move of all servos ([Cyclic/collective pitch mixing — Wikipedia](https://en.wikipedia.org/wiki/Cyclic/collective_pitch_mixing), [RCHelicopterFun — CCPM](https://www.rchelicopterfun.com/ccpm.html)):
- **120°** — three servos equally spaced; the dominant standard today (Align T-Rex, SAB Goblin, Mikado Logo, etc.).
- **135°/140°** — elevator servo geometry adjusted so both side servos have equal ratio on the elevator axis, removing collective interaction and giving very uniform cyclic response ([BEASTX wiki — CCPM setup](https://wiki.beastx.com/index.php?title=Manuals%3AAR7210FblV5%3ASetupmenu_G%2Fen)).
- **90° (H1/mechanical)** — servos at right angles, or "no-mix" single-servo-per-axis; legacy.
- ArduPilot's traditional-heli support documents the same H3-120/H3-140/H1 options ([ArduPilot swashplate setup](https://ardupilot.org/copter/docs/traditional-helicopter-swashplate-setup.html)).

**Flybar vs flybarless (FBL):** A flybar (Bell-Hiller stabilizer bar) is a weighted paddle bar acting as a mechanical gyroscope: it resists attitude change and mechanically feeds a stabilizing cyclic input into the blade grips. It costs efficiency (paddle drag), adds head complexity, and damps agility. **FBL heads delete the bar entirely — but a bare FBL rotor is dynamically unstable/twitchy, so a 3-axis MEMS gyro + PID loop must replicate the flybar's damping electronically**, sending corrections to the swash servos hundreds/thousands of times per second ([RCHelicopterFun — flybarless](https://www.rchelicopterfun.com/flybarless.html)). Virtually all modern RC helis are FBL. **This is the core justification for the custom PCB: the flight controller *is* the flybar.**

---

## 3. Rotor Dynamics

- **90° phase lag ("gyroscopic precession"):** a cyclic pitch input applied at one point of the rotation produces maximum blade displacement ~90° later in the direction of rotation. Strictly, for a flapping rotor this is a resonance phenomenon (blade flapping at 1/rev is a driven oscillator at resonance → 90° phase shift), not classic rigid-body gyroscopic precession, and the lag is <90° when flapping hinges are offset/stiff. FBL units expose this as a "phase angle" parameter ([RCHelicopterFun — gyroscopic precession](https://www.rchelicopterfun.com/gyroscopic-precession.html), [PPRuNe discussion](https://www.pprune.org/rotorheads/19678-helicopter-dynamics-gyroscopic-precession.html)). The swashplate geometry pre-compensates: control inputs are applied 90° ahead of where the response is wanted.
- **Dissymmetry of lift & blade flapping:** in forward flight the advancing blade sees airspeed + rotational speed, the retreating blade sees the difference → unequal lift. Blades flap up on the advancing side and down on the retreating side, changing their local angle of attack and self-equalizing lift. Without flapping freedom (hinges or flexible blades/dampers), forward flight would roll the aircraft ([Pilot Mall aerodynamics](https://www.pilotmall.com/blogs/news/how-do-helicopters-fly-the-aerodynamics-explained)). Retreating-blade stall ultimately limits forward speed.
- **Ground effect:** within roughly one rotor diameter of the surface, induced downwash is reduced, so the rotor needs less power for the same thrust (hover IGE vs OGE). Matters for takeoff/landing behavior and for governor load transients near the ground.
- **Autorotation:** with power removed, the pilot immediately lowers collective; descent drives air *upward* through the disc. On the inboard "driving region" of the blades the resultant aerodynamic force tilts forward of the rotation axis, keeping the rotor spinning; stored rotor kinetic energy is then traded in a final collective flare for a soft touchdown. Full-size descent rates are typically 1500–2000 fpm; below a minimum rotor RPM the rotor stalls unrecoverably ([SKYbrary — Autorotation](https://skybrary.aero/articles/autorotation), [AOPA — The Art of Autorotation](https://www.aopa.org/news-and-media/all-news/2000/june/flight-training-magazine/the-art-of-autorotation)). RC helis with negative-pitch-capable collectives autorotate routinely — a genuine safety/recovery capability no multirotor has. Note the ESC implication: heli ESCs offer "autorotation bailout" (fast spool-up from idle) so an aborted auto can recover ([Hobbywing Platinum 80A V5](https://www.hobbywing.com/en/news/info/34)).

---

## 4. The FBL Control Loop

A modern FBL controller (BeastX, VBar, Spirit, iKon, or open-source **Rotorflight**) does:

1. **3-axis rate stabilization:** MEMS gyro (often + accelerometer) → PID loops on roll/pitch/yaw; output is *swashplate cyclic commands*, not motor speed changes. Stick inputs command angular rates; the loop makes the unstable FBL head fly like it's on rails ([RC FlightPath — open-source FBL](https://www.rcflightpath.com/articles/fbl-fc/)).
2. **Tail gyro in heading-hold mode:** integrating yaw loop that locks heading against torque transients from collective/power changes (the historical "heading hold gyro," now integrated).
3. **Governor:** closed-loop RPM control holding constant headspeed regardless of load — either in the ESC or in the FBL (FBL governors can feed-forward from collective stick for faster response; BeastX claims ±10 RPM hold). Requires an RPM signal ([ArduPilot internal governor](https://ardupilot.org/copter/docs/traditional-helicopter-internal-rsc-governor.html), [Rotorflight](https://github.com/rotorflight/rotorflight-firmware)).
4. **Why constant RPM + collective, not throttle:** a multirotor changes thrust by changing prop RPM — fine for tiny, low-inertia props. A large heli rotor has huge rotational inertia; RPM changes are far too slow for control. Instead RPM is held constant and **thrust is changed near-instantly by blade pitch**. Bonus: constant RPM keeps rotor vibration frequencies fixed (helps gyro filtering) and keeps stored energy available for autorotation and hard 3D maneuvers.

**Rotorflight** (Betaflight 4.3 fork, exclusively for helicopters, STM32 G4/F4/F7/H7) is the reference open-source implementation and an excellent design benchmark for this board ([rotorflight-firmware](https://github.com/rotorflight/rotorflight-firmware), [DIY board requirements](https://rotorflight.org/docs/next/controllers/betaflight-diy)).

---

## 5. Electric RC Helicopter Powertrain

- **Architecture:** one brushless outrunner/inrunner → pinion → large main gear (typical single-stage reduction ~8:1 to 12:1) → main shaft. Motor Kv and gear ratio set headspeed for a given battery (3S micro up to 12S–14S on 700/800 class).
- **Tail drive:** on most helis ≥380-class, the *same motor* drives the tail via **torque tube** (shaft in the boom; efficient, quieter, constant tail authority in autorotation, but fragile gears in crashes) or **belt** (cheap, crash-tolerant, needs tension maintenance). **Motorized/direct-drive tails** (separate small brushless motor + ESC on the boom) are common only on micro/small helis — fast acceleration and great hold on small craft, but thrust response can't keep up on larger machines, and you lose tail authority in autorotation ([RCHelicopterFun — rudder/tail systems](https://www.rchelicopterfun.com/rc-helicopter-rudder.html), [HeliFreak torque tube vs belt](https://www.helifreak.com/showthread.php?t=529939)).
- **Typical headspeeds (main rotor):** roughly inverse to size — 250-class ~3500–4500 RPM; **450-class ~2700–3400 RPM**; 550-class ~2200–2800; **700-class ~1300–2200 RPM** depending on style (scale/efficiency low end, hard 3D high end) ([Rc-Help 450 headspeed thread](https://www.rc-help.com/threads/headspeed-rpms.18940/), [RcHeliaddict head-speed discussion](https://www.rcheliaddict.co.uk/forum/main-forum/main-discussions/121878-), [Align headspeed tool](https://aligntrexhelis.com/headspeed)). Tail rotors typically spin ~4–6× headspeed.
- **ESC requirements** (benchmark: Hobbywing Platinum series, the de facto heli standard):
  - Current: 60–80 A continuous for 450/500 class, 120–160 A continuous / 200 A burst for 700-class HV ([Platinum 120A V4](https://www.hobbywing.com/en/products/platinum-120a-v471), [Platinum HV 160A](https://hobbyking.com/hobbywing-platinum-hv-160a-esc-heli-and-fixed-wing.html)).
  - **Governor mode** with smooth soft-start/spool-up and **autorotation bailout**.
  - **BEC:** built-in switching BEC, 5–8 V (up to 12 V on V5) adjustable, ~10 A continuous / 20–25 A peak — sized for cyclic servo loads ([Hobbywing Platinum 80A V5](https://www.hobbywing.com/en/news/info/34)).
  - **Telemetry:** RPM, voltage, current, temperature streaming (S.BUS2/VBar/SRXL etc.) and data logging.

---

## 6. Helicopter vs Multirotor

| Aspect | Single-rotor heli | Multirotor |
|---|---|---|
| **Disc loading / hover efficiency** | One large rotor → low disc loading → low induced velocity → less power per kg of thrust. Hover power scales with √(disc loading) | Total disc area split into small fast props → higher disc loading, lower efficiency ([Disk loading — Wikipedia](https://en.wikipedia.org/wiki/Disk_loading), [Krossblade — disc loading & hover efficiency](https://www.krossblade.com/disc-loading-and-hover-efficiency)) |
| **Endurance** | Better for equal weight/battery (drivetrain losses partly offset this, but the aero advantage dominates at size) | Shorter |
| **Thrust control** | Collective pitch @ constant RPM → millisecond thrust response, negative thrust available | RPM modulation, thrust ≥ 0 only (bidirectional 3D quads excepted) |
| **Flight envelope** | Full 3D aerobatics, sustained inverted, **autorotation on power loss** | No autorotation; motor/ESC failure on a quad = crash |
| **Mechanical complexity** | Swashplate, linkages, gears, bearings, tail drive — maintenance-heavy, vibration-rich | Almost no moving parts beyond motors |
| **Electronics complexity** | 3–4 servos + 1–2 ESC channels + RPM loop + heavy gyro filtering | 4 ESC outputs, simpler filtering |
| **Failure modes** | Single motor is a *feature* (autorotation) but linkage/servo failure is critical | Motor-out fatal on quads, tolerable on hex/octo |

---

## 7. Implications for the Custom AIO FC + ESC PCB

This is where the board diverges sharply from a standard multirotor AIO:

1. **Servo power rail (the biggest difference).** 3 swash servos + 1 tail servo, all high-torque digitals that can transiently draw several amps each under 3D load. Budget a **switching BEC at 5–8.4 V, ≥10 A continuous, 20–25 A peak** (matching Hobbywing Platinum practice), with bulk capacitance and brown-out isolation from the MCU's 3V3 rail. Consider a redundant/backup input — a BEC brown-out on a heli is an instant crash.
2. **Outputs: servos, not motors.** Instead of 4× DShot motor pads, you need **4× servo PWM outputs** (typically 50–333 Hz analog or 560/760 µs narrow-pulse for tail servos; Rotorflight supports per-output rates) **+ 1 ESC drive channel** (PWM or DShot to the integrated ESC stage), optionally a second motor output for a motorized tail on small airframes ([Rotorflight DIY board requirements](https://rotorflight.org/docs/next/controllers/betaflight-diy)).
3. **RPM sensing for the governor.** Provide an input for motor eRPM: from the on-board ESC stage this is free (BEMF zero-cross count or DShot bidirectional telemetry); also pin out an external RPM/phase-sensor input. Governor + RPM filtering both depend on it.
4. **Vibration environment.** A heli generates strong periodic vibration at main rotor **1/rev (~25–70 Hz for 1500–4000 RPM), n/rev blade-pass harmonics, tail rotor frequencies, and motor frequency**. These sit right in the control band, so: soft-mount or mechanically isolate the IMU, choose a gyro with good vibration rejection, and rely on **RPM-referenced notch filters** (Rotorflight's RPM filter: notches at main-rotor fundamental + 2nd harmonic, tail fundamental + 2nd, typical Q≈2.5) — which again requires the RPM signal routed to the MCU ([Rotorflight RPM filters](https://rotorflight.org/docs/2.1.0/setup/rpm-filters), [filter tuning](https://rotorflight.org/docs/Tuning/First-Flight-Filter-Tuning)). Because headspeed is governed constant, notches are narrow and effective — a design synergy worth stating.
5. **ESC power stage.** For a 450-class target: ≥60–80 A continuous on 6S; 700-class would need 120–160 A on 12S, which realistically forces a separate power board. Design for: low-inductance FET layout, phase current sensing (governor quality + telemetry), temperature sensing, soft spool-up ramp, **governor and autorotation-bailout logic in ESC firmware**, and voltage/current/RPM telemetry back to the FC.
6. **MCU/firmware path.** STM32F7/H7 (or G4 for compact boards) with enough timers for 4 servos + motor + LED/RPM inputs makes the board directly **Rotorflight-compatible**, which gets a mature heli control stack (rates, governor, tail heading hold, RPM filters, mixer for 120°/135°/140° CCPM) for free ([rotorflight GitHub](https://github.com/rotorflight); see also the Matek G474-HELI as a commercial reference design ([RMRC listing](https://www.readymaderc.com/products/details/85877-matek-rc-helicopter-flybarless-controller-g474-heli))).

---

### One-slide takeaway
A single-rotor heli holds rotor RPM constant and flies entirely on *blade pitch*: collective for thrust, cyclic (via swashplate + 90° phase lag) for attitude, tail pitch for yaw. The FBL controller replaces the mechanical flybar with a gyro PID loop, and the governor closes an RPM loop through the ESC. The AIO PCB therefore isn't "a quad FC with fewer motors" — it's a **servo-centric board**: big BEC, 4 servo outputs, one high-current ESC stage with governor + bailout, RPM sensing, and an IMU/filtering chain engineered around fixed-frequency rotor vibration.
