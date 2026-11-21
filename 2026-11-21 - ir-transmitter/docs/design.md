# IR transmitter: design

Status: draft for approval. Nothing is built.

## Intent

The user wants to stop using remote controls. One ESP32 per room carries an IR LED, and Home Assistant replays recorded commands through it to every device in that room. The transmitter is meant to disappear: it sits inside a hollow object that is already in the room, and only the IR LED and a thin USB-C cable show.

## Goals

- One hidden transmitter per room covers every IR device in the room.
- Each real appliance is its own Home Assistant device with one entity per recorded command.
- A 3D-printed tray fits any hollow object. The tray is customizable: the board pocket and everything on it stay fixed, and only the plate outline changes per object. Making one for a new object means editing a few named variables, or supplying the cavity outline.
- Only the IR LED (in a neat bezel) and the USB-C cable are visible outside the object.
- The same tray later accepts a small radio module.

## Non-goals

- Recording IR codes on the hidden device. Recording stays in copycat on a separate receiver board.
- Battery operation.
- A designed enclosure. The host object is the enclosure.

## Anti-goals

- A tray that only works for one box.
- A transmitter whose range is too weak to reach across a room, however tidy it looks.
- Re-flashing every time a command is added, if the API service can carry it.

## Constraints

- Power is USB-C. The board is therefore a USB-C ESP32 variant, not the micro-USB DevKit v1 of the existing copycat devices.
- Fan codes are 136-168 symbols long, so the chosen chip's RMT must transmit them reliably.
- Hollow objects range from a shallow box (about 6 cm inside) to a statue base. The tray stays flat and small.

## Approaches considered

1. **Flat tray with the IR LED on a short lead (chosen).** The tray holds the board in a pocket and is glued to the inside floor. The LED sits on a 10-15 cm lead and press-fits into a printed bezel glued around a drilled hole at any point on the wall.
2. Rigid tray with a built-in LED arm. Tidier, but the hole has to be at an exact position, so it fits one object.
3. A custom insert per object. Best looking, but every object needs new CAD.

## Design

```
Home Assistant <-wifi-> ESP32 (USB-C) -> driver -> IR LED(s) in a bezel in the wall
                          ^ tray glued inside the object
                          '- USB-C cable leaves through a notch
```

- **Firmware:** the transmit-only ESPHome config that copycat generates (the `esp32-2` pattern: `remote_transmitter` plus a `transmit_pronto` API service), one per room.
- **Home Assistant:** commands become button entities grouped under a device per appliance. How best to create those devices (generated ESPHome entities per command, or Home Assistant's own IR support) is checked against current ESPHome and Home Assistant before it is fixed.
- **Tray:** CadQuery, named dimensions at the top of the script, exported STEP and STL committed beside it, tests on the exported geometry (per the repository's CAD convention). It is a fixed board module (board pocket sized from the real board's measured dimensions, USB-C cable relief, a slot for a future radio module) sitting on a plate whose outline is a parameter. A clip-in LED bezel is a second part.
- **Plate outline:** two modes, both producing the same board module on top.
  - *Glue plate:* a plain rectangle (width, depth, thickness as named variables) sized to give enough glue area, for flat-floored objects like the red box. It is larger than the board module.
  - *Cavity fit:* an outline that matches the cavity exactly, for objects like the statue base. The outline is a list of measured points (or a traced outline from a photo or paper template) in the script's configuration, and the plate is extruded from it. The board module is positioned on the plate by named offsets, and a test checks the module still fits inside the outline.
  - Each object's variant is a small configuration file beside the shared script (for example `objects/red-box.py`, `objects/statue-base.py`), so adding an object adds one file and no new modelling.
- **IR LED:** a transistor-driven LED (or two angled LEDs) rather than the bare GPIO-driven HX-53 module, so range holds across a room. Beam angle and range are measured in a real room.

## Decisions I made (not yet confirmed)

- Option 1, a flat tray with a leaded LED and a bezel.
- A small USB-C board in the ESP32-S3 or ESP32-C3 SuperMini class, since it fits shallow objects. The RMT symbol capacity of these chips differs from the classic ESP32. S3 supports DMA for RMT, so it is the first candidate for long fan codes.
- A transistor-driven LED stage instead of the HX-53 module.
- The cable leaves through a 3-4 mm notch at the back or bottom.

## Left to the builder

- Choose the board after one test: transmit a real 168-symbol fan code from the candidate board and confirm the fan responds.
- Measure the chosen board and design the pocket from the measurements.
- Pick the LED, driver and current. Check range across a room.
- Settle how Home Assistant devices and entities are created from copycat's remotes and commands.
- Decide how the radio-module slot is laid out when a radio module is chosen.
- Measure the statue base cavity and decide how its outline is captured (measured points or a traced template).
- Print and fit the tray into the red box (glue plate) and the statue base (cavity fit) as first checks.
