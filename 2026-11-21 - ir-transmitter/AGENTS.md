# IR transmitter

A hidden, USB-C-powered ESP32 infrared transmitter, one per room, on a 3D-printed tray that is glued inside any hollow object (a box, a statue base) with only the IR LED visible outside. Home Assistant sends recorded IR commands through it, replacing the room's remote controls.

`docs/design.md` holds the design and the open decisions, and `docs/superpowers/plans/` the implementation plan. The parts the user already owns are in the DIY parts inventory (the `diy-parts` skill); check it before suggesting a part to buy.

## Constraints

- The transmitter is transmit-only. Commands are recorded with the existing copycat app (`monocoque/apps/copycat`), which stores each button as a Pronto code and generates the ESPHome config for a transmit-only device.
- One transmitter per room. It must be able to reach every IR device in the room, and a radio transmitter (433/868 MHz) may be added later, so the tray leaves room for one more small module.
- Power is USB-C from a nearby charger, with the cable leaving through a small notch or hole. There is no battery.
- The hollow objects are easy to drill and saw. The tray is glued in place. The board module is the same for every object, and only the plate outline is per object: a rectangle with enough glue area for a flat floor (the box), or an outline that fits the cavity exactly (the statue base).
- Commands appear in Home Assistant as separate devices (TV, fan, AC), each with one entity per command.

## Facts established

- The existing copycat devices are ESP32 DevKit v1 boards with an HX-53 IR LED module on GPIO21, an RMT symbol buffer of 256, and the `transmit_pronto` API service. Fan remote codes are 136-168 symbols long.
- The HX-53 IR LED module on the existing copycat devices is too weak in practice, and the user has bought replacement components for a stronger transmit stage. Phase 1 (a working breadboard with receiver and transmitter) has not been reached yet, so the range of the new stage is unmeasured.
- The two example objects are a hinged presentation box of roughly 15 x 11 x 6 cm and the hollow base of a statue with a seam that can be cut open.
- A DevKit v1 board has a micro-USB port, so a USB-C board is a different board and its RMT capability for long codes has to be verified with a real fan code.
