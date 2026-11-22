# LED candles

Battery LED block candles rebuilt so Home Assistant can switch them (and dim them when the LED allows) individually and as a group, and read back their state and battery level.

`DESIGN.md` holds the design and `docs/superpowers/plans/` the implementation plan. `docs/measurements.md` collects teardown and bench measurements, with photos in `docs/teardown/`, and `docs/bom.md` lists the parts to order.

## Constraints

- All parts come from AliExpress, so the design uses only commodity parts with several sellers.
- The original control chip, its 8h timer and its board are discarded. The original plastic base and its battery contacts are reused, and the MCU and CC1101 sit in the hollow shell.
- Average standby current stays well under 100 µA, so battery life is set by how long the LED burns.
- The radio is a CC1101 with wake-on-radio, 868 MHz as the target and 433 MHz as the fallback. EU duty-cycle limits apply, so commands are few and short.
- The CC1101 accepts at most 3.6 V.
- The MCU runs down to about 2 V. The CH32V003 needs 2.7 V and is excluded.

## Facts established

- The candle in the teardown photos is a North Light LED Pillar Candle (Clas Ohlson 34-2087). It is not the JYSK Dante from the original brief, and other models may have different bases.
- The candles run on 1.2 V NiMH cells (2.2-2.8 V in use), so battery percentage is derived from voltage per battery type; NiMH stays nearly flat until close to empty.
- The original candles flicker realistically, and the rebuilt candle has to match that.
- The wick LED has two wires, which fits both a plain LED driven by the board's chip and a self-flickering LED with its own chip. Which one it is stays unmeasured until it is powered directly. A self-flickering LED cannot be dimmed with PWM, and the flicker would then be reproduced in software on a plain warm-white LED, with the original's brightness pattern recorded from the original candle as the reference.
- The original board sits in a narrow pocket beside the battery bay. The pocket holds about the board's own size, so the radio and MCU go up into the shell.
- Measuring current uses the user's UNI-T UT133B multimeter, whose DC current range has a 600.0 µA scale with 0.1 µA resolution.
- The CC1101's average wake-on-radio current depends on the poll interval. A user-reported figure is about 60 µA at a 1 s interval, so the bench measurement is the authority.
