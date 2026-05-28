# 05 — AIO FC+ESC vs Separate FC + ESC Stack

A central design question for Phase 2's **System 2** (Lightweight AIO).

## The two designs
| Option | What it is |
|--------|-----------|
| **AIO** | FC and 4-in-1 ESC on a **single PCB** |
| **Stack** | Separate FC board mounted above a separate ESC board, connected via a wire harness |

## Tradeoffs
| Aspect | AIO | Stack |
|--------|-----|-------|
| Weight | **Lower** (one PCB, less wiring) | Higher |
| Size | **Smaller** | Larger |
| Cost (BOM) | Lower in volume | Higher |
| Thermal isolation | Worse — ESC heat near MCU/IMU | **Better** |
| EMI between FC sensors and ESC switching | Worse — needs careful layout | **Better** by physical separation |
| Repairability | Replace whole board | Replace either layer |
| Max current | Limited by PCB area | **Higher** (dedicated heatsinking) |
| Modularity (mix-and-match) | None | **Flexible** |

## When AIO wins
- Sub-3" / sub-250 g builds where weight is everything
- High-volume products with locked component choices
- When IMU is well-isolated (separate ground plane, careful layout)

## When Stack wins
- Higher current builds (≥45 A continuous per motor)
- Industrial/commercial drones requiring service/repair in the field
- When sensor redundancy and clean EMI environment matter (Phase 2's System 1)

## Sources
<!-- 1. -->
