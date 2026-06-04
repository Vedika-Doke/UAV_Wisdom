# 05 — AIO FC+ESC vs Separate FC + ESC Stack

A central design question for Phase 2's **System 2** (Lightweight AIO). This page presents the tradeoffs in enough depth to defend the decision in the report/PPT.

The choice isn't binary "AIO is better for small, stack is better for big" — it's a function of **5 axes** (weight budget, current budget, thermal headroom, serviceability requirement, design effort). Once you understand the axes, the right answer for a given vehicle falls out.

---

## 1. The two designs

| Option | Physical form |
|--------|---------------|
| **AIO** ("All-In-One") | FC logic + 4-in-1 ESC on a **single PCB**, single mounting plane, single connector to the battery. |
| **Stack** | Two (or three) PCBs sandwiched with M2/M3 standoffs. Typical order: ESC on bottom (near battery wires), FC in middle, VTX/cam-control on top. Connected by a small ribbon/JST connector between layers. |

```
   AIO                          Stack
┌─────────────────┐         ┌─────────────────┐  ← FC (or VTX)
│   FC + ESC      │         └─────────────────┘
│   one PCB       │         ┌─────────────────┐  ← FC
└─────────────────┘         └─────────────────┘
                            ┌─────────────────┐  ← ESC
                            └─────────────────┘
```

---

## 2. The five-axis tradeoff

### Axis 1: Weight
| Component | AIO | Stack |
|-----------|----:|------:|
| One PCB (substrate, copper) | ~3 g | — |
| Two PCBs | — | ~5 g + ~5 g |
| Standoffs (4 × M3 × 25 mm) | — | ~1.5 g |
| Stack interconnect (JST + wires) | — | ~1 g |
| **Total weight overhead** | **~3 g** | **~12 g** |

For a sub-250 g build, saving 9 g is meaningful (4 % of MTOW). For a 1.2 kg cinelifter, 9 g is noise.

### Axis 2: Current handling
| Aspect | AIO | Stack |
|--------|-----|-------|
| PCB area for ESC stage | Smaller (shared with FC) | Dedicated full PCB |
| Heatsinking | Limited to PCB copper | Larger copper pours + airflow gap |
| Maximum continuous current | ~30–40 A per channel | **60+ A per channel** |
| Thermal hot-spot risk | Higher | Lower |

This is the single hardest tradeoff. AIOs are *fine* for sub-3" / 5" cinematic builds at moderate current. For aggressive 5" freestyle at 50+ A burst or any 7"+ build, a stack ESC outperforms.

### Axis 3: EMI between ESC switching and FC sensors
| Aspect | AIO | Stack |
|--------|-----|-------|
| ESC PWM noise into IMU | **Worst case** — IMU is ~10 mm from MOSFETs | **Best case** — IMU is on a separate PCB |
| Magnetometer placement | Useless on AIO (huge magnetic field from VBAT trace) | Almost useless on stack ESC layer too — both need external mag on GPS |
| Signal integrity for ESC telem UART | Short trace (good) | Across the inter-PCB connector (more reflections) |

EMI is **manageable on AIO with careful layout**:
- IMU on its own copper ground island, connected to main GND only at one point
- Star-grounding for analog GND
- IMU on a different layer/side from ESC switching nodes
- 6-layer PCB to give EMI more isolation room (more cost)

### Axis 4: Serviceability / repair
| Aspect | AIO | Stack |
|--------|-----|-------|
| If one MOSFET blows | Replace whole board ($60–80) | Replace ESC only ($40) |
| If FC firmware corrupts | Same | Same — flash via USB |
| If frame crash damages PCB | Whole-board replace | Replace whichever layer broke |
| Field repair (in the truck) | Bolt-swap, ~5 min | Bolt-swap + connect ribbon, ~5 min |
| Spare-parts inventory | One SKU | Two SKUs (more flexibility) |

Service matters more for commercial fleets than for hobby users.

### Axis 5: Modularity / mix-and-match
| Aspect | AIO | Stack |
|--------|-----|-------|
| Upgrade FC without changing ESC | ❌ | ✅ |
| Upgrade ESC without changing FC | ❌ | ✅ |
| Try different ESC firmwares | Same (firmware swap) | Same |
| Bring-your-own VTX/cam-control | ✅ via UART pads | ✅ as another stack layer |

For a **product** (Phase 2 System 2), modularity is *less* important — you're shipping a fixed config. For a **research / dev platform**, modularity is gold.

---

## 3. Decision matrix by use case

| Vehicle class | Weight budget | Current | Service need | **Recommend** |
|---------------|--------------|---------|--------------|--------------|
| Tinywhoop (sub-100 g) | extreme | 5–10 A | none | **AIO** (no choice anyway) |
| 3" cinewhoop (~250 g) | tight | 20–25 A | low | **AIO** |
| 5" freestyle (~600 g) | moderate | 35–50 A | moderate | **Either — AIO if light, stack if punchy** |
| 7" long-range (~1 kg) | loose | 30 A cruise / 60 A burst | moderate | **Stack** (better thermals at length) |
| 10" cinelifter (~2 kg) | none | 80+ A | high | **Individual ESCs** (not even 4-in-1) |
| Industrial multirotor (5–25 kg) | none | 100+ A per motor | **high** | **Individual ESCs, often potted, on each arm** |
| Phase 2 System 2 target (sub-300 g compact) | extreme | 25 A | low | **AIO** ✅ |

---

## 4. Design implications for Phase 2 System 2 (AIO target)

The brief says: *"Low weight, Compact size, Good thermal handling, Low noise, Power efficiency."*

That maps to AIO. Concrete implications:

### MCU + IMU sub-block (the FC half)
- STM32H743 or AT32F435 — small QFN package
- ICM-42688-P on its own SPI bus
- Sensor area on a **separate ground island** in the top-left/top-right corner, away from ESC stage
- Decoupling: 100 nF on every IC pin, 10 µF bulk per chip

### ESC sub-block (the ESC half)
- 4× AT32F421 (one per motor) running AM32
- 24× MOSFETs (6 per motor) — pick **30 V, ≤2 mΩ, DFN-5×6** parts
- Current shunts (1 mΩ) with op-amp amplifiers feeding into the AT32 ADCs
- VBAT input through a **low-ESR 1000 µF / 35 V cap**, footprinted *as close to the MOSFETs as possible*
- 4-layer (or better 6-layer) PCB with thick GND/VBAT planes

### Layout discipline (what kills AIO designs)
1. **Keep IMU SPI traces short** and away from any switching node
2. **Star-ground the IMU** to the digital GND at a single point
3. **Capacitor placement matters more than spec** — caps must be physically near the load
4. **Thermal vias** under MOSFETs to the bottom-side copper pour
5. **Test points** for everything — current monitor outputs, sensor data, ESC PWM, motor outputs. Bring-up will go faster.

### What you save vs a stack
- ~9 g (significant for compact UAVs)
- ~$10 BOM (one less PCB, less assembly)
- Footprint: roughly half the height
- One single connector to the battery instead of an XT60-to-FC harness

### What you give up
- ~30 % less continuous current handling
- Harder to push past 4S without thermal headroom problems
- Field repair requires whole-board swap

---

## 5. Reference AIO designs to dissect

Before designing Phase 2 System 2, **read the schematic of at least one** of these (vendor product pages usually link a PDF):

| Board | Class | Why look at it |
|-------|-------|----------------|
| **SpeedyBee F405 Mini AIO 25A** | 20×20, 4S | Direct reference — proven 7 g design |
| SpeedyBee F405 Mini AIO 40A | 20×20, 6S | Same architecture, 6S current path |
| HGLRC Zeus F722 AIO 60A | 25.5, 6S | Higher current — see thermal layout |
| iFlight BLITZ F7 AIO | 25.5, 6S | Different layout philosophy |
| BetaFPV F4 1S 5-in-1 | 14×14, 1S | Extreme miniaturisation |

When reading: trace the VBAT path from the battery pad → bulk cap → ESC FETs. Trace the IMU SPI path. Note how grounding is done. Note which side of the board the MOSFETs vs MCU live on.

---

## 6. Conclusion (for the report)

> *Phase 2 System 2 will use an AIO FC+ESC architecture because the brief prioritises weight, compactness, and thermal-efficient operation in the sub-300 g compact-UAV class. At this weight/current point, the AIO penalty (~30 % reduced peak current handling, harder field service) is outweighed by the AIO benefits (~12 g saved, ~$10 BOM saved, lower assembly complexity, single-PCB integration). For higher-current applications (5" freestyle race builds, 7"+ long-range, or any payload-carrying vehicle) Phase 2 would specify a stack instead.*

## Sources
1. Oscar Liang — *FPV Stack vs AIO* — https://oscarliang.com/aio-vs-stack-fpv/
2. SpeedyBee AIO product pages — https://www.speedybee.com/
3. HGLRC AIO product pages — https://www.hglrc.com/
4. Joshua Bardwell — *5-inch AIO vs stack comparison videos* — YouTube
5. Phil's Lab — *PCB design for high-current motor drivers* — YouTube — https://www.youtube.com/@PhilsLab
