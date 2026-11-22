# IR Transmitter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a hidden USB-C ESP32 IR transmitter on a customizable 3D-printed tray that is glued inside a hollow object with only the IR LED outside, and expose recorded IR commands in Home Assistant as one device per appliance.

**Architecture:** A transmit-only ESPHome device (the copycat `esp32-2` pattern) drives a transistor-stage IR LED. The tray is one CadQuery model: a fixed board module on a plate whose outline is a parameter (rectangular glue plate, or an exact cavity outline), with one small config file per object and a separate clip-in LED bezel. Home Assistant devices and entities are created from copycat's remotes and commands.

**Tech Stack:** ESPHome, ESP32-S3/C3 USB-C board, CadQuery (Python, run with pytest), Home Assistant.

**Spec:** `docs/design.md` in the experiment root. Paths below are relative to this experiment's directory in the `rn-ax/diy` repo; commit steps follow that repo's branch-and-PR workflow (one pull request per experiment).

## Global Constraints

- One transmitter per room, transmit-only; recording stays in copycat (`monocoque/apps/copycat`). Anything the experiment needs from copycat is copied into this directory.
- Power is USB-C, no battery; the cable leaves through a 3-4 mm notch or hole.
- Phase 1 is a working breadboard with receiver and transmitter before any CAD; the board is a small USB-C ESP32 (S3 or C3 SuperMini class), chosen by the Task 1 tests.
- Fan codes are 136-168 symbols; the board must transmit them reliably.
- Only the IR LED (in its bezel) and the USB-C cable are visible outside the object.
- The board module is identical for every object; only the plate outline is per object.
- The tray leaves room for one more small module (future 433/868 MHz radio).
- Printable parts are CadQuery scripts with named dimensions at the top; exported STEP and STL are committed beside the script; tests check dimensions on the exported geometry.
- No personal information in committed files; photos have metadata stripped.

## Review Focus

- A cavity outline that is self-intersecting, open or too small for the board module must fail loudly, not export a broken plate (Task 4).
- A glue plate smaller than the board module plus a glue margin must be rejected (Task 4).
- A long fan code (168 symbols) must be sent whole, not truncated by the RMT buffer (Task 1).
- A receiver that sees noise or a repeat burst must not publish an empty or partial code as a recording (Task 1).
- A command whose name has spaces or non-ASCII characters (e.g. "Ström på") must still become a valid Home Assistant entity (Task 6).
- The LED hole in the wall can be off-centre or on a curved surface; the bezel must still seat and hold the LED aimed outward (Task 5).

## File Structure

```
firmware/
  ir-breadboard.yaml           breadboard config: receiver + transmitter (phase 1)
  ir-transmitter.yaml          transmit-only config for the hidden device, one copy per room (task 3)
  transmit.py                  copy of copycat's transmit script (API client)
  tests/test_transmit.py       integration test, needs a device (marked)
hardware/tray/
  tray.py                      board module + plate builder (CadQuery)
  bezel.py                     clip-in LED bezel
  objects/red_box.py           glue-plate config
  objects/statue_base.py       cavity-fit config
  export.py                    writes STEP/STL next to the scripts
  tests/test_tray.py, tests/test_bezel.py
homeassistant/
  generate.py                  copycat remotes -> ESPHome/HA entity definitions
  tests/test_generate.py
docs/
  design.md, measurements.md, superpowers/plans/
```

## Phase 1: Breadboard (Tasks 1-2)

Both the receiver and the transmitter work on a breadboard before any CAD. Phase 1 is done when a code captured from a real remote is replayed by the transmitter and the device responds.

### Task 1: Breadboard receiver and transmitter

**Files:**
- Create: `firmware/ir-breadboard.yaml`, `firmware/transmit.py`, `firmware/tests/test_breadboard.py`, `firmware/tests/fixtures/fan_168_symbols.txt`
- Create: `docs/measurements.md`

**Interfaces:**
- Produces: `firmware/ir-breadboard.yaml` (receiver publishing each decoded code as a Pronto string, transmitter with the `transmit_pronto` API service), the chosen board and its receiver and transmitter GPIOs, and a captured real-remote Pronto code, all recorded in `docs/measurements.md` and used by Tasks 2-3 and 6.

- [ ] **Step 1: Document the existing breadboard**: the ESP32 HW-394 board, the IR receiver module (3V3, GND, signal GPIO with pull-up, active-low) and the IR LEDs on a second GPIO with a transistor stage. Record the wiring, resistor values and pins in `docs/measurements.md`, correcting the breadboard where it differs from the intended circuit.
- [ ] **Step 2: Create `firmware/ir-breadboard.yaml`** from copycat's `esp32-1` pattern: `remote_receiver` (inverted, pull-up) publishing a Pronto text sensor, and `remote_transmitter` with `rmt_symbols` sized for 168 symbols, `non_blocking: true` and the `transmit_pronto` API service.
- [ ] **Step 3: Copy `transmit.py`** from `monocoque/apps/copycat/esphome/transmit.py` unchanged.
- [ ] **Step 4: Write the integration tests** (marked `integration`, `DEVICE_HOST` env var): `test_receiver_captures_a_real_remote_button` (press a button on a real remote, the "Last IR Code" text sensor changes to a non-empty Pronto string), and `test_long_pronto_code_is_transmitted_whole` (sends `fixtures/fan_168_symbols.txt` and asserts the service returns without error and the device log reports all symbols sent).
- [ ] **Step 5: Run the tests on an ESP32-S3 board, then an ESP32-C3 board**: `DEVICE_HOST=<host> pytest firmware/tests -m integration -v -s`. Expected: PASS on at least one.
- [ ] **Step 6: Round trip**: capture a button from a real remote with the receiver, replay it with the transmitter and confirm the real device responds. Save the captured code as a fixture.
- [ ] **Step 7: Record in `docs/measurements.md`** which boards passed, the chosen board and the reason.
- [ ] **Step 8: Commit** with `feat: add breadboard IR receiver and transmitter`.

### Task 2: LED stage and range

**Files:**
- Modify: `firmware/ir-breadboard.yaml` (pin, carrier duty)
- Modify: `docs/measurements.md`

**Interfaces:**
- Consumes: the working breadboard and chosen board from Task 1.
- Produces: LED part, driver circuit, lead length and bezel hole diameter recorded in `docs/measurements.md`, used by Task 5.

- [ ] **Step 1: Build the stronger transmit stage from the components the user bought** (list them in `docs/measurements.md` first), and compare it with the HX-53 module as the baseline, recording the circuit and current for each.
- [ ] **Step 2: Measure range** by replaying the Task 1 round-trip code from three positions in a room, through a 5 mm hole in a cardboard wall, and record the furthest reliable distance and angle per variant.
- [ ] **Step 3: Pick the variant** that works from the far side of the room and write the hole diameter and lead length into `docs/measurements.md`.
- [ ] **Step 4: Commit** with `docs: record IR LED stage and range`.

## Phase 2: Hidden transmitter (Tasks 3-7)

The deployed device is transmit-only (`firmware/ir-transmitter.yaml`, the breadboard config without the receiver); the receiver stays on the breadboard for recording.

### Task 3: Board module of the tray

**Files:**
- Create: `hardware/tray/tray.py`, `hardware/tray/tests/test_tray.py`
- Modify: `docs/measurements.md` (board dimensions)

**Interfaces:**
- Consumes: the chosen board (Task 1). Produces `firmware/ir-transmitter.yaml` from `ir-breadboard.yaml` by dropping the receiver, and the board outline and USB-C port position and height from measuring the board.
- Produces: `build_board_module() -> cq.Workplane` (pocket and walls with the board's footprint at the origin, USB-C cable relief, radio-module slot, bezel-lead channel) and module constants `MODULE_WIDTH`, `MODULE_DEPTH`, `MODULE_HEIGHT`, used by Tasks 4-5.

- [ ] **Step 1: Measure the real board** with calipers (length, width, thickness, USB-C port width, height and offset) and write them as named variables at the top of `tray.py` and in `docs/measurements.md`.
- [ ] **Step 2: Write failing tests** in `test_tray.py`: `test_pocket_holds_board_with_clearance` (exported bounding box of the pocket exceeds the board by the clearance variable on every side), `test_cable_relief_clears_usbc_plug`, `test_radio_slot_exists_and_is_clear_of_the_board`.
- [ ] **Step 3: Run to verify failure**: `pytest hardware/tray/tests/test_tray.py -v` → FAIL (function not defined).
- [ ] **Step 4: Implement `build_board_module()`** so the tests pass.
- [ ] **Step 5: Run to verify pass**, then commit with `feat: add tray board module`.

### Task 4: Plate outlines and per-object configs

**Files:**
- Create: `hardware/tray/objects/red_box.py`, `hardware/tray/objects/statue_base.py`, `hardware/tray/export.py`
- Modify: `hardware/tray/tray.py`, `hardware/tray/tests/test_tray.py`

**Interfaces:**
- Consumes: `build_board_module()` and the `MODULE_*` constants (Task 3).
- Produces: `build_glue_plate(width: float, depth: float, thickness: float, module_offset: tuple[float, float]) -> cq.Workplane`, `build_cavity_plate(outline: list[tuple[float, float]], thickness: float, module_offset: tuple[float, float]) -> cq.Workplane`, each returning the plate with the board module on it. Each object file defines its parameters (`NAME`, and either `GLUE_PLATE = {...}` or `CAVITY_OUTLINE = [...]`). `export.py <object>` writes `<object>.step` and `<object>.stl` beside the script.

- [ ] **Step 1: Write failing tests**: `test_glue_plate_is_larger_than_module_by_glue_margin`, `test_glue_plate_smaller_than_module_is_rejected` (raises `ValueError`), `test_cavity_plate_matches_outline_area`, `test_cavity_plate_contains_module` (module footprint inside outline), `test_self_intersecting_outline_is_rejected`, `test_open_outline_is_rejected` (all `ValueError`).
- [ ] **Step 2: Run to verify failure.**
- [ ] **Step 3: Implement both builders** with validation; the cavity plate is extruded from the outline polygon, and the module is placed at `module_offset`.
- [ ] **Step 4: Write `objects/red_box.py`** from measuring the red box floor (glue plate), and **`objects/statue_base.py`** from measured or traced cavity points; record how the outline was captured in `docs/measurements.md`.
- [ ] **Step 5: Implement `export.py`**, run it for both objects, and commit the STEP and STL beside the scripts.
- [ ] **Step 6: Run all tests** → PASS, then commit with `feat: add glue-plate and cavity-fit outlines`.

### Task 5: LED bezel

**Files:**
- Create: `hardware/tray/bezel.py`, `hardware/tray/tests/test_bezel.py`

**Interfaces:**
- Consumes: LED diameter, hole diameter and wall-thickness range from Task 2.
- Produces: `build_bezel(led_diameter: float, hole_diameter: float, wall_thickness: float) -> cq.Workplane` (press-fit sleeve with a flange to glue around the hole).

- [ ] **Step 1: Write failing tests**: `test_led_bore_matches_led_with_press_fit`, `test_sleeve_fits_the_drilled_hole`, `test_flange_is_wider_than_hole_for_gluing`, `test_bezel_holds_led_on_off_centre_or_angled_hole` (bore axis stays aligned with the sleeve axis).
- [ ] **Step 2: Run to verify failure.**
- [ ] **Step 3: Implement `build_bezel`.**
- [ ] **Step 4: Run tests** → PASS, export, commit with `feat: add clip-in LED bezel`.

### Task 6: Home Assistant devices and entities

**Files:**
- Create: `homeassistant/generate.py`, `homeassistant/tests/test_generate.py`
- Modify: `docs/measurements.md` (which Home Assistant mechanism was used and why)

**Interfaces:**
- Produces: `generate(remotes: list[Remote]) -> str` where `Remote` is `{"name": str, "commands": {str: str}}` (button name → Pronto code); the returned ESPHome YAML has one button entity per command, grouped under one device per remote, with entity ids derived from slugified names.

- [ ] **Step 1: Check current ESPHome and Home Assistant documentation** for how to create a device per appliance with entities per command, pick the mechanism and record it in `docs/measurements.md`.
- [ ] **Step 2: Write failing tests**: `test_one_device_per_remote`, `test_one_button_per_command`, `test_entity_ids_are_valid_for_spaces_and_non_ascii_names`, `test_duplicate_command_names_in_one_remote_are_disambiguated`.
- [ ] **Step 3: Run to verify failure.**
- [ ] **Step 4: Implement `generate`.**
- [ ] **Step 5: Run tests** → PASS, flash one room's device with generated output, and confirm each button transmits in Home Assistant.
- [ ] **Step 6: Commit** with `feat: generate Home Assistant devices from remotes`.

### Task 7: Print, fit and install

**Files:**
- Modify: `docs/measurements.md`, `AGENTS.md`

- [ ] **Step 1: Print the tray for the red box and the statue base** and the bezel; test-fit with the real board.
- [ ] **Step 2: Drill the LED hole, glue the plate and bezel, and run the cable out through the notch.** Check that nothing but the LED and cable is visible.
- [ ] **Step 3: Transmit a command from Home Assistant from across the room** and confirm the device responds.
- [ ] **Step 4: Record fit problems and fixes** in `docs/measurements.md`, and add durable facts to `AGENTS.md`.
- [ ] **Step 5: Commit** with `docs: record fit and install results`.
