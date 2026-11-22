# Transmit stage on the breadboard

`transmit-stage.schematex` is the source of the breadboard drawing (half-size board, 30 columns). It is rendered with the `schematex` skill.

Two IR LEDs in series, switched by a 2N2222A transistor, so the LEDs carry far more current than a board pin can give.

## Build order

Leave USB unplugged until the last step.

1. Red wire from the board's 5V (VIN) pin to the top plus rail, and a red jumper from the rail at column 3 to hole `3a`.
2. Black wire from a board GND pin to the bottom minus rail.
3. 47 Ω resistor (yellow, violet, black, gold, brown) from `3c` to `6c`.
4. IR LED 1 across the centre gap: long leg (anode) in `6e`, short leg in `6f`.
5. IR LED 2: long leg (anode) in `6h`, short leg in `9h`.
6. 2N2222A with the three legs in `9i`, `10i` and `11i`: collector, base, emitter. Kit parts differ in pin order, so if nothing lights, swap collector and emitter.
7. 1 kΩ resistor (brown, black, black, brown, brown) from `10g` to `14g`.
8. Yellow wire from the transmit pin (GPIO21 in the copycat setup) to `14h`.
9. Black jumper from `11j` to the bottom minus rail.
10. Optional: 100 µF capacitor between the plus and minus rails, long leg on plus.

## What is connected where

| Strip | Connected |
|---|---|
| top column 3 | jumper from the plus rail, 47 Ω |
| top column 6 | 47 Ω, LED 1 anode |
| bottom column 6 | LED 1 cathode, LED 2 anode |
| bottom column 9 | LED 2 cathode, transistor collector |
| bottom column 10 | transistor base, 1 kΩ |
| bottom column 11 | transistor emitter, jumper to the minus rail |
| bottom column 14 | 1 kΩ, yellow wire from the transmit pin |

## Limits of the drawing

Schematex has no transistor and no infrared LED, so the transistor is a 3-pin header and the LEDs are drawn white. The drawing does not show LED polarity, so it is stated above.
