# Frames, Motors & Propellers

The "physical layer" of an FPV drone. Three components that together set the drone's class (how big it is), its agility (thrust-to-weight, response), and its efficiency (flight time). Pick them as a *system* — a 5" motor on a 3" frame is wrong.

---

## 1. Frames

A frame is just the structural skeleton — almost always **carbon fibre plates** with **standoffs** between them. The defining measurement is the **prop size it accepts**, which is set by the **motor-to-motor distance** (the "wheelbase").

### Frame classes
| Class | Wheelbase | Prop size | Typical AUW | Use case |
|-------|-----------|-----------|-------------|----------|
| Tinywhoop | 65–85 mm | 1.6" (40 mm) ducted | 25–40 g | Indoor, beginner |
| Toothpick | 90–130 mm | 2–3" | 60–120 g | Indoor/outdoor micro |
| Cinewhoop | 130–160 mm | 2.5–3" ducted | 150–300 g | Indoor cinematic (ducts protect surroundings) |
| **5" Freestyle** | **210–230 mm** | **5"** | **500–700 g** | **The "default" FPV class — start here** |
| 6"–7" Long-range | 270–315 mm | 6–7" | 700–1000 g | Cruising, mapping, distance |
| 10" cinelifter | 400+ mm | 10" | 2–4 kg | Cinema cameras (BMPCC, Komodo) |

### Frame geometry
- **True-X** — symmetric, equal arm lengths. Standard for freestyle.
- **Stretch-X** — front/back arms longer than left/right. Smoother forward flight (racing).
- **Hybrid-X** / **Dead-cat** — gap at the front for camera FOV (no props in shot).

### Mounting holes (the spec that determines what FCs/ESCs fit)
Standardized in millimetres between mounting holes:
- **20 × 20** — micros (toothpick, sub-3")
- **25.5 × 25.5** — mid-size
- **30.5 × 30.5** — 5" freestyle (the most common)

> When picking an FC and ESC, match the frame's mounting standard *first* — everything else is moot if it doesn't bolt down.

---

## 2. Motors

FPV motors are **outrunner brushless DC (BLDC) motors** — the casing spins, the stator is fixed. Three numbers fully describe one:

### Naming: `SSHH KKKKKV`
- **SS** — stator **diameter** in mm
- **HH** — stator **height** in mm
- **KKKK** — **KV rating** (RPM per volt, no load)

Example: **2207 1950KV** = 22 mm stator diameter × 7 mm tall, spins at 1950 RPM per applied volt unloaded.

### Stator size
A bigger stator = more copper + more iron = more torque, more weight, more heat capacity.

| Stator | Class | Notes |
|--------|-------|-------|
| 0802–1103 | Tinywhoop | |
| 1404–1606 | Toothpick | |
| 1804–2004 | 3" cinewhoop | |
| **2207** | **5" freestyle/racing** | **Most popular size — high peak power** |
| **2306** | **5" cinematic/freestyle** | **Smoother torque, better cooling (wider, shallower)** |
| 2806–2812 | 6–7" long-range | |
| 3115+ | Heavy-lift / cinelifter | |

**2207 vs 2306 on a 5"**: 2207 punches harder for racing/extreme acro; 2306 is smoother and more efficient for cinematic and long sessions. Both are *correct* choices — flying style picks the winner.

### KV rating — matching to battery
KV is **RPM/Volt with no load**. With propeller drag, real RPM is lower, but it sets the ballpark.

Rule of thumb: **higher cell count → lower KV needed** for the same prop RPM.

| Battery | Voltage (nominal) | Typical 5" KV |
|---------|-------------------|---------------|
| 4S | 14.8 V | 2400–2750 |
| **6S** | **22.2 V** | **1700–2000** (modern default) |

The industry has largely moved from 4S → 6S because, at the same power, 6S draws *less current*, so wiring/ESC losses drop. The motor sees the same wattage but at higher voltage / lower amps.

### Thrust-to-Weight Ratio (TWR)
TWR = total static thrust / AUW. Required values:

| TWR | What it means |
|-----|---------------|
| < 2:1 | Will barely fly, no responsiveness |
| 2–3:1 | Beginner-friendly, sluggish |
| **4–5:1** | **Comfortable freestyle / camera multirotor** |
| 6–8:1 | Aggressive freestyle, racing, snappy response |
| 10:1+ | Extreme racing |

To pick motors: estimate AUW → multiply by target TWR → divide by 4 → that's the required **max static thrust per motor**, which manufacturers publish in datasheet thrust tables.

### Reading a motor thrust table
Vendors (T-Motor, EMAX, RCINPower, BrotherHobby) publish PDFs with thrust per propeller at various throttle %. Always check:
- **Thrust at 100 %** — peak — defines TWR ceiling.
- **Thrust at 50 %** — *hover thrust* — should be ~25 % of total weight per motor.
- **Current at 100 %** — sets minimum ESC rating.
- **Efficiency (g/W)** at hover throttle — sets flight time.

---

## 3. Propellers

### Naming: `DDPP[B]`
- **DD** — diameter in inches
- **PP** — pitch in inches (theoretical forward travel per revolution through a solid medium — like a screw)
- **B** — blade count (2, 3, 4)

Example: **5043 tri** = 5.0" diameter × 4.3" pitch × 3 blades.

### Diameter vs pitch
| Increase | Effect |
|----------|--------|
| Diameter | More thrust, more torque load on motor, higher inertia |
| Pitch | More thrust per rev (faster forward), more torque load, more current |
| Blade count | More thrust, more drag, more current, **less efficiency**, smoother response |

Higher pitch and more blades = more aggressive = more current = shorter flight time + hotter motors. It's a continuous tradeoff, not a "better/worse".

### Bi-blade vs tri-blade
| Aspect | Bi-blade (2) | Tri-blade (3) | Quad-blade (4) |
|--------|:---:|:---:|:---:|
| Efficiency | **Best** | Medium | Worst |
| Top speed | **Highest** | Medium | Lowest |
| Thrust per RPM | Lowest | Medium | **Highest** |
| Smoothness / "grip" | Loose | **Sweet spot** | Locked-in but draggy |
| Common use | Racing, long-range cruise | **Freestyle (default)** | Cinematic, ducted |

### Material
- **Polycarbonate** — most common, cheap, breaks before damaging motors (a feature)
- **Polycarbonate + glass-fibre / carbon-fibre fill** — stiffer, better response, more $
- **Pure carbon** — dangerous (doesn't break on impact, will hurt someone)

### A reasonable starting combo for a 5" freestyle
- Frame: 5" true-X, 30.5×30.5 mounting (e.g. iFlight Nazgul5, Armattan Marmotte)
- Motors: 4× **2207 1950KV** (6S) — e.g. T-Motor F40 Pro V, EMAX ECO II
- Props: **HQProp 5.1×3.1×3** or **Gemfan 51466** tri-blade
- Expected AUW: ~620 g with 6S 1300 mAh
- Expected TWR: ~6:1

## Sources
1. UAVMODEL — *How to Choose FPV Motors* — https://blog.uavmodel.com/how-to-choose-fpv-motors-stator-size-kv-rating-and-torque-selection-guide/
2. Ligpower — *2207 vs 2306 Motors* — https://www.ligpower.com/blog/2207-motors-vs-2306-motors.html
3. UAVMODEL — *KV and Cell Count Matching (2026)* — https://blog.uavmodel.com/fpv-drone-motor-kv-and-cell-count-matching-voltage-prop-load-and-thrust-optimization-2026-guide/
4. Oscar Liang — *How to Choose Propellers* — https://oscarliang.com/propellers/
5. Unmanned Tech — *Bi-Blade vs Tri-Blade vs Quad-Blade (2026)* — https://www.unmannedtechshop.co.uk/blogs/knowledge-base/propeller-guide-tri-blade-vs-bi-blade-vs-quad-blade-fpv
6. Pyrodrone — *Advanced Propeller Dynamics* — https://pyrodrone.com/blogs/introduction-to-fpv/advanced-propeller-dynamics-blade-pitch-size-and-how-they-affect-your-flight
