# Measurements

## Task 1: candle teardown

Photos in `docs/teardown/` (IMG_1071 base label, IMG_1072 board, IMG_1073 hollow shell with LED wires, IMG_1074 battery bay).

### Unit torn down

North Light LED Pillar Candle, Clas Ohlson article 34-2087, 2xAA (LR06). This is not the JYSK Dante candle from the design brief; the other models in the collection still need a look to see whether they share the same board.

### From the photos

- Slide switch on the base with three positions: ON / OFF / Timer. Its contacts are on the board (silkscreen "OFF", "TIMER").
- Board is small and narrow, mounted in the base next to the battery bay. Silkscreen reads B+ / B- (battery), 3U+ / U+ / U- (LED side), R1, C1, T1, U1 (3-pin SOT-23) and U2 (8-pin SOIC, the controller).
- A small metal cylinder soldered to the board looks like a 32 kHz timing crystal (the 8h timer).
- The LED sits at the wick on two thin wires (one white, one grey/silver) running up through a hollow shell, so there is plenty of free volume above the base for a CC1101 module and an MCU board. The base itself is tight.
- Batteries in this unit are IKEA 1.2 V NiMH, 1900 mAh (2.4 V nominal, about 2.2-2.8 V in use), not alkaline.

### Still to measure

- [ ] LED forward voltage and current at full brightness.
- [ ] Whether the wick LED flickers by itself when powered directly (through a series resistor, from 2xAA). If it does, it is a flicker LED with its own chip, PWM dimming will not work on it, and the design's software flicker is unnecessary; the fix is to replace it with a plain warm-white LED.
- [ ] Off-state current of the original board from the battery contacts.
- [ ] Free volume in the base (mm), and where the MCU board and CC1101 module can sit.
