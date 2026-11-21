# LED Candles on Home Assistant Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild battery LED block candles around a tiny sleeping MCU and a CC1101 wake-on-radio receiver so Home Assistant can switch, dim and read back each candle.

**Architecture:** Each candle holds an MCU, a CC1101 module and a MOSFET-driven LED. A shared protocol library (portable C, host-tested) defines the packets. The candle firmware is a hardware-independent core (state machine, tested on the host) plus a thin hardware layer. The hub is an ESP32 with a CC1101 running ESPHome, which exposes each candle to Home Assistant as a light and a battery sensor.

**Tech Stack:** C (gcc host tests, MCU toolchain chosen in Task 2), CC1101 radio, ESPHome with an external component, Home Assistant.

**Spec:** `DESIGN.md` in the experiment root. Paths below are relative to this experiment's directory in the `rn-ax/diy` repo, and commit steps follow that repo's branch-and-PR workflow.

## Global Constraints

- All parts from AliExpress; commodity parts with several sellers only.
- Control of the candle is by Home Assistant: on/off and brightness, individually and as a group. The candle's original control chip and 8h timer are removed.
- Candle reports confirmed state and battery voltage; reports on every command, on a manual button press, and about once a day.
- Listening costs a few uA on average; per-candle cost of a few euros plus one shared hub.
- 868 MHz target (433 MHz an acceptable fallback within its own duty-cycle rules); the duty-cycle limit applies, so commands are few and short.
- CC1101 accepts at most 3.6 V; 2xAA is fine directly, 3xAAA needs an LDO or a rebuild as 2xAAA.
- Poll interval about 1 s.
- Failsafe: with no command for 12 h, a lit candle switches itself off.
- A manual button toggles the candle with no radio involved.
- Electronics must fit inside the existing candle body.
- Battery level is reported as measured voltage; the hub converts it to a percentage.
- Out of scope: rechargeable power, the candle's own timer, security beyond a network ID.

## Review Focus

- A neighbour's system on the same band with a different network ID must not switch the candles (Task 3, sync word).
- A corrupted packet must never change state (Task 3, CRC).
- A group command that every candle acknowledges at once must not collide or break the duty-cycle budget (Task 4, staggered acks).
- Brightness 0 and "off" must be the same state, and the 12 h failsafe timer must restart on every command, including a repeat of the current state (Task 4).
- A battery sagging below the CC1101's minimum voltage must switch the candle off and report low battery first, not leave it flickering or hanging (Task 4).

## File Structure

```
firmware/
  common/protocol.h, protocol.c     packet format, encode/decode (shared by candle and hub)
  candle/core.h, core.c             hardware-independent state machine
  candle/hal.h                      hardware interface the core uses
  candle/hal_<mcu>.c                MCU-specific radio, PWM, ADC, button, sleep
  tests/test_protocol.c, test_core.c
hub/
  candle-hub.yaml                   ESPHome config
  components/candle_radio/          external component using protocol.c
docs/measurements.md                teardown and bench results
Makefile                            builds and runs host tests
```

---

### Task 1: Candle teardown

**Files:**
- Create: `docs/measurements.md`

**Interfaces:**
- Produces: LED forward voltage and current, free internal volume and dimensions, battery compartment contacts, switch/button location. Later tasks read these.

- [ ] **Step 1: Open one candle** and photograph the board, LED(s), switch and battery contacts.
- [ ] **Step 2: Measure** LED current at full brightness and the off-state current from the 2xAA contacts with a multimeter in series; record both in `docs/measurements.md`.
- [ ] **Step 3: Record** free internal volume (mm), where a CC1101 module (about 20x16 mm breakout) and a small MCU board could sit, and whether the original LED and resistor are reusable. Verified when `docs/measurements.md` has all five numbers/answers.

### Task 2: Bench measurement of cheap radio modules

**Files:**
- Modify: `docs/measurements.md`

**Interfaces:**
- Produces: chosen CC1101 module listing, chosen MCU part, measured sleep and wake-on-radio average current, go/no-go for the approach.

- [ ] **Step 1: Order** 5 CC1101 breakouts (868 MHz where listed, check the seller's stated frequency) and 5 candidate MCU boards (CH32V003 and ATtiny-class).
- [ ] **Step 2: Measure** each module's sleep current and its average current in wake-on-radio mode at a 1 s poll interval, from a 3.0 V supply with a uA-capable meter. Record results per module.
- [ ] **Step 3: Measure** the candidate MCU's deep-sleep current with the CC1101 attached and the interrupt pin wired.
- [ ] **Step 4: Decide** MCU and module and write the decision plus the combined standby current in `docs/measurements.md`. Go if combined standby is about 30 uA or less; otherwise stop and report to the user with the numbers.

### Task 3: Protocol library

**Files:**
- Create: `firmware/common/protocol.h`, `firmware/common/protocol.c`, `firmware/tests/test_protocol.c`, `Makefile`

**Interfaces:**
- Produces:
  - `typedef struct { uint8_t dst, src, seq, cmd, arg, batt; } packet_t;` (6 bytes on air)
  - `#define ADDR_GROUP 0x00`, `#define ADDR_HUB 0x01`, candle addresses `0x10`-`0xEF`
  - `#define CMD_SET 0x01` (arg = brightness 0-255, 0 = off), `#define CMD_QUERY 0x02`, `#define CMD_REPORT 0x81` (arg = brightness, batt = voltage in 20 mV steps)
  - `size_t packet_encode(const packet_t *p, uint8_t out[6]);`
  - `bool packet_decode(const uint8_t in[], size_t len, packet_t *p);` (false on wrong length or unknown cmd)
  - `uint16_t network_sync_word(const char *network_name);` hashes a name to the 16-bit CC1101 sync word, so a neighbour's system with another name never matches.

- [ ] **Step 1: Write failing tests** in `firmware/tests/test_protocol.c`: `test_roundtrip` (encode then decode returns the same packet), `test_decode_rejects_wrong_length`, `test_decode_rejects_unknown_cmd`, `test_sync_word_differs_per_network` (two names give two words, same name gives the same word, neither is 0x0000 or 0xFFFF). CRC and address filtering are the CC1101's own hardware features, so a test that a one-byte corruption is rejected belongs in Task 6 on real hardware.
- [ ] **Step 2: Run** `make test` and confirm the tests fail to build or fail.
- [ ] **Step 3: Implement** the functions above in `protocol.c` with the signatures given.
- [ ] **Step 4: Run** `make test` and confirm all pass.
- [ ] **Step 5: Commit.**

### Task 4: Candle core logic

**Files:**
- Create: `firmware/candle/core.h`, `firmware/candle/core.c`, `firmware/candle/hal.h`, `firmware/tests/test_core.c`

**Interfaces:**
- Consumes: `packet_t`, `CMD_*`, `ADDR_*` from Task 3.
- Produces:
  - `hal.h`: `void hal_set_pwm(uint8_t duty); void hal_radio_send(const uint8_t *buf, size_t len); uint8_t hal_read_battery(void); uint32_t hal_millis(void);` (the core calls these; tests supply fakes)
  - `candle_t` with `void candle_init(candle_t *c, uint8_t my_addr);`
  - `void candle_on_packet(candle_t *c, const packet_t *p);`
  - `void candle_on_button(candle_t *c);`
  - `void candle_tick(candle_t *c);` (called on every wake; handles flicker, failsafe, daily report, low-battery cutoff)
  - `bool candle_can_sleep(const candle_t *c);`

- [ ] **Step 1: Write failing tests** in `firmware/tests/test_core.c` using fake HAL functions: `test_set_brightness_drives_pwm`, `test_brightness_zero_is_off`, `test_set_to_group_address_applies`, `test_packet_for_other_address_ignored`, `test_ack_sent_after_set_delayed_by_address` (ack delay is `(my_addr - 0x10) * 20 ms`, so a group command's acknowledgements do not collide), `test_button_toggles_without_radio_send`, `test_failsafe_turns_off_after_12h` and `test_failsafe_restarts_on_repeat_command` (a SET to the current brightness restarts the 12 h timer), `test_daily_report_sent`, `test_low_battery_reports_then_turns_off` (battery below 2.0 V sends one REPORT, sets PWM to 0, and the candle stays off until the battery recovers).
- [ ] **Step 2: Run** `make test` and confirm failure.
- [ ] **Step 3: Implement** `core.c`. Flicker is a small pseudo-random modulation of brightness in the tick; keep it in one function so the effect can be tuned later.
- [ ] **Step 4: Run** `make test` and confirm all pass.
- [ ] **Step 5: Commit.**

### Task 5: Hardware layer and candle firmware for one MCU

**Files:**
- Create: `firmware/candle/hal_<mcu>.c`, `firmware/candle/main.c`, build file for the toolchain chosen in Task 2

**Interfaces:**
- Consumes: `hal.h` and `candle_*` from Task 4, `packet_*` and `network_sync_word` from Task 3.
- Produces: a flashable image that configures the CC1101 (868 MHz, hardware CRC, address filtering on the candle address with broadcast 0x00, sync word from `network_sync_word("<name>")`, wake-on-radio at a 1 s poll interval) and sleeps whenever `candle_can_sleep` is true.

- [ ] **Step 1: Implement** `hal_<mcu>.c` against `hal.h`; the CC1101 register setup lives in one function with the values commented.
- [ ] **Step 2: Implement** `main.c`: init, then loop: wake on the CC1101 interrupt, button or timer, call `candle_on_packet`, `candle_on_button` or `candle_tick`, sleep.
- [ ] **Step 3: Build** with the chosen toolchain; verified when the image builds without warnings and its size fits the MCU.
- [ ] **Step 4: Flash** one bench board and confirm the LED responds to a button press and the board sleeps (current at about the Task 2 figure).
- [ ] **Step 5: Commit.**

### Task 6: Hub, ESPHome component

**Files:**
- Create: `hub/candle-hub.yaml`, `hub/components/candle_radio/` (ESPHome external component)

**Interfaces:**
- Consumes: `packet_t`, `packet_encode`, `packet_decode`, `network_sync_word` from Task 3.
- Produces: per candle, a light entity (on/off/brightness) and a battery sensor in Home Assistant; one light entity for the group address; each command is re-sent up to 3 times until the matching REPORT arrives, and the hub tracks airtime to stay inside the duty-cycle limit.

- [ ] **Step 1: Write** the external component so it sends a preamble long enough to span the candle's 1 s poll interval before each command.
- [ ] **Step 2: Validate** with `esphome config hub/candle-hub.yaml`; expected output is a valid configuration.
- [ ] **Step 3: Compile** with `esphome compile hub/candle-hub.yaml` and flash an ESP32 with a CC1101.
- [ ] **Step 4: Commit.**

### Task 7: One-candle end-to-end on the bench

**Files:**
- Modify: `docs/measurements.md`

**Interfaces:**
- Consumes: the candle image from Task 5 and the hub from Task 6.

- [ ] **Step 1: Print a base and rebuild one candle.** Model a 3D-printed base (PETG) sized to the shell opening from the Task 1 measurements, holding a 2xAA holder, the MCU board, the CC1101 module with its antenna clear of the batteries, the manual button and the LED wires; the original innards are discarded. Write the model as a CadQuery script in `hardware/base/` with its exported STEP and STL committed beside it, and assemble the MCU, CC1101 and MOSFET-driven LED into it.
- [ ] **Step 2: Verify** from Home Assistant: on, off, dim, flicker, and that the light entity shows confirmed state after the REPORT arrives.
- [ ] **Step 3: Verify a corrupted packet and a foreign network:** transmit a packet with a wrong sync word and a packet with a broken CRC from a second radio; the candle must not change state.
- [ ] **Step 4: Measure** average current over 24 h in idle and lit states, latency from Home Assistant to light, and range through two walls; record results in `docs/measurements.md`.
- [ ] **Step 5: Verify the failsafe** by shortening the timer constant in a test build; the lit candle must switch off.

### Task 8: Rollout

**Files:**
- Create: `docs/building.md`, `hub/automations-example.yaml`

- [ ] **Step 1: Document** the build (parts list with AliExpress search terms, wiring, flashing, assigning an address) in `docs/building.md`.
- [ ] **Step 2: Add** an example Home Assistant automation switching the group at dusk and off at 23:00 in `hub/automations-example.yaml`.
- [ ] **Step 3: Build** the remaining 2xAA candles from the parts order, one at a time, verifying each with Task 7 step 2. 3xAAA candles are decided separately after the first batch (regulator, rebuild as 2xAAA, or skip).
