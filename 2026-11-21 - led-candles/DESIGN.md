# LED candles on Home Assistant: design

Status: draft for approval. Nothing is built.

## Intent

Samuel owns a number of battery LED block candles ([JYSK Dante, sand, 8x10 cm](https://jysk.se/inredning/dekoration/led-ljus/led-blockljus-dante-o8xh10cm-sand)) with an on/off/8h-timer switch, powered by 2xAA or 3xAAA. The goal is to switch and dim them from Home Assistant (dusk-to-bedtime schedules per room) using cheap parts, with a battery life close to what the candles have today.

## Goals

- Home Assistant controls each candle: on/off and brightness, individually and as a group.
- Home Assistant sees each candle's confirmed state and battery level.
- Listening costs a few uA on average, so battery life is set by how long the LED burns.
- Per-candle cost of a few euros, plus one shared transmitter.
- A candle with a dead radio link still behaves safely (see Failsafe).

## Non-goals

- Keeping the candle's own 8h timer or its original control chip. Schedules live in Home Assistant.
- Mains or rechargeable power, or other light types.
- Security beyond preventing a neighbour's identical system from switching the candles (a network ID in each packet).

## Anti-goals

- Batteries that last weeks instead of months because the listener draws too much.
- A candle that cannot be switched on by hand when Home Assistant is down.

## Constraints

- EU sub-GHz rules (Åland/Finland): the 868 MHz SRD band is duty-cycle limited, so commands are few and short.
- The CC1101 accepts at most 3.6 V. 2xAA (about 2.0-3.2 V) is fine directly. 3xAAA (up to 4.5 V) needs an LDO or regulator, or a rebuild as 2xAAA.
- The original innards need not be kept. The candle shell is reused and the base is a custom 3D-printed part holding the battery holder, MCU board, CC1101 module, manual button and the LED wiring, sized to the shell opening.
- Every part is ordered from AliExpress, so the design uses only commodity parts with several sellers (CC1101 breakout modules, CH32V003/ATtiny-class MCUs, small MOSFETs, ESP32 boards). Cheap CC1101 modules come in 433 and 868 MHz variants and clones exist, so the first order is a handful of modules whose sleep and wake-on-radio current is measured before ordering in volume. Parts that need a specific unusual chip or a hard-to-source antenna are avoided.

## Approaches considered

1. **Sub-GHz radio with wake-on-radio (CC1101) plus a tiny MCU, ESP32 with CC1101 running ESPHome on the Home Assistant side. Chosen.** Cheapest per candle, a few uA average, two-way, full control of the LED.
2. **Zigbee sleepy end device.** Appears in Home Assistant with no custom transmitter, but needs mains routers in range, and the modules cost more per candle.
3. **125 kHz wake-up receiver (AS3933).** Instant wake at about 3 uA, but range is a few metres and it needs antenna coils and a custom transmitter.

## Design

```
Candle:  2xAA -> MCU (deep sleep ~1 uA) -> MOSFET -> LED (PWM)
                 CC1101 (wake-on-radio) -> interrupt -> MCU
Hub:     Home Assistant <-> ESP32 + CC1101 (ESPHome)
```

- The original control chip is removed. The original LED and its resistor are kept at first, driven through a logic-level MOSFET with PWM for dimming and a flicker effect.
- The CC1101 samples the air every ~1 s. It wakes the MCU only on a packet carrying the network ID and the candle's address or the group address.
- Commands are on/off/brightness. The hub sends a preamble long enough to span the poll interval, and a group address lets one transmission reach every candle.
- The candle replies with a short acknowledgement carrying state and battery voltage. It also reports on its own about once a day and on a manual button press.
- A manual button toggles the candle with no radio involved.
- The ESPHome side exposes each candle to Home Assistant as a light entity plus a battery sensor.

## Decisions I made (not yet confirmed with Samuel)

- 868 MHz rather than 433 MHz (433 MHz modules are the more common AliExpress listing, so the seller's stated frequency is checked, and 433 MHz stays an acceptable fallback within its own duty-cycle rules).
- Failsafe: with no command for 12 h, a lit candle switches itself off.
- Poll interval of about 1 s, trading latency against listening current.
- Battery level is reported as a measured voltage, converted to a percentage on the hub.

## Left to the builder

- Teardown of one candle to see the board, LED current and available space.
- Choice of MCU (anything with a sleep current of about 1 uA or less and PWM; CH32V003 and ATtiny-class parts are candidates).
- Measuring real average listening current at the chosen poll interval before ordering in volume.
- Packet format, retry policy and addressing scheme within the duty-cycle limit.
- Whether the 3xAAA candles are rebuilt to 2xAAA, regulated, or skipped.
- Candle count and battery types are not yet known.
