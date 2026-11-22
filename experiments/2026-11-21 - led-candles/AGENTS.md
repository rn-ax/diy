# LED candles

Battery LED block candles rebuilt so Home Assistant can switch and dim them individually and as a group, and read back their state and battery level.

`DESIGN.md` holds the approved design and `docs/superpowers/plans/` the implementation plan. `docs/measurements.md` collects teardown and bench measurements, with photos in `docs/teardown/`.

## Constraints

- All parts come from AliExpress, so the design uses only commodity parts with several sellers.
- The original control chip, its 8h timer and the original base are discarded. The candle shell is reused with a 3D-printed base.
- Average standby current is a few µA, so battery life is set by how long the LED burns.
- The radio is a CC1101 with wake-on-radio, 868 MHz as the target and 433 MHz as the fallback. EU duty-cycle limits apply, so commands are few and short.
- The CC1101 accepts at most 3.6 V.

## Facts established

- The candle in the teardown photos is a North Light LED Pillar Candle (Clas Ohlson 34-2087). It is not the JYSK Dante from the original brief, and other models may have different boards.
- The candles run on 1.2 V NiMH cells (2.2-2.8 V in use), so battery percentage is derived from voltage per battery type; NiMH stays nearly flat until close to empty.
- The original candles flicker realistically, and the rebuilt candle has to match that.
- The wick LED has two wires, which fits both a plain LED driven by the board's chip and a self-flickering LED with its own chip. Which one it is stays unmeasured until it is powered directly. A self-flickering LED cannot be dimmed with PWM, and the flicker would then be reproduced in software on a plain warm-white LED, with the original's brightness pattern recorded from the original candle as the reference.

## CAD

The printed base is modeled in CadQuery under `hardware/base/` with the shell diameter and other dimensions as named variables at the top of the script. The exported STEP and STL are committed beside the script, and the dimension tests check the exported geometry.
