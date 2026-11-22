# IR control

Infrared control of the home: recording remote-control signals (receiving) and replaying them (transmitting). The transmitting side is a hidden, USB-C-powered ESP32 infrared transmitter, one per room, on a 3D-printed tray that is glued inside any hollow object (a box, a statue base) with only the IR LED visible outside. Home Assistant sends recorded IR commands through it, replacing the room's remote controls.

`docs/design.md` holds the design and the open decisions, and `docs/superpowers/plans/` the implementation plan. The parts the user already owns are in the DIY parts inventory (the `diy-parts` skill); check it before suggesting a part to buy.

## Constraints

- The hidden room transmitter is transmit-only, and the receiving side stays on a separate recorder board. Commands are recorded with the existing copycat app (`monocoque/apps/copycat`), which stores each button as a Pronto code and generates the ESPHome config for a transmit-only device.
- One transmitter per room. It must be able to reach every IR device in the room, and a radio transmitter (433/868 MHz) may be added later, so the tray leaves room for one more small module.
- Power is USB-C from a nearby charger, with the cable leaving through a small notch or hole. There is no battery.
- The hollow objects are easy to drill and saw. The tray is glued in place. The board module is the same for every object, and only the plate outline is per object: a rectangle with enough glue area for a flat floor (the box), or an outline that fits the cavity exactly (the statue base).
- Commands appear in Home Assistant as separate devices (TV, fan, AC), each with one entity per command.

## Facts established

- The existing copycat devices are ESP32 DevKit v1 boards with an HX-53 IR LED module on GPIO21, an RMT symbol buffer of 256, and the `transmit_pronto` API service. Fan remote codes are 136-168 symbols long.
- The transmitters tried so far (the HX-53 IR LED module on the existing copycat devices) were too weak: they very rarely managed to control the ceiling lamp. The user ordered a 940 nm IR LED and photodiode set (3 mm and 5 mm) for a stronger transmit stage. The IR receiver on the copycat recorder already works, and the sensor kit holds a spare 3-pin IR receiver module. A breadboard exists with an ESP32 HW-394 board (WROOM-32D, USB-C), an IR receiver module and two IR LEDs; its circuit and the range of the new stage are not yet documented or measured.
- The two example objects are a hinged presentation box of roughly 15 x 11 x 6 cm and the hollow base of a statue with a seam that can be cut open.
- A DevKit v1 board has a micro-USB port, so a USB-C board is a different board and its RMT capability for long codes has to be verified with a real fan code.
