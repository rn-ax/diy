# LED candles on Home Assistant: design

## Intent

The user owns a number of battery LED block candles (for example the North Light LED Pillar Candle from Clas Ohlson and the [JYSK Dante, sand, 8x10 cm](https://jysk.se/inredning/dekoration/led-ljus/led-blockljus-dante-o8xh10cm-sand)) with an on/off/8h-timer switch, powered by 2xAA or 3xAAA. The goal is to switch them from Home Assistant (dusk-to-bedtime schedules per room) using cheap parts, with a battery life close to what the candles have today.

## Goals

- Home Assistant switches each candle on and off, individually and as a group, and dims it when the LED allows.
- Home Assistant sees each candle's confirmed state and battery level.
- The flame flickers as realistically as the original candles do.
- Listening costs well under 100 uA on average, so battery life is set by how long the LED burns.
- Per-candle cost of a few euros, plus one shared transmitter.

## Non-goals

- Keeping the candle's own 8h timer, its original control chip or its original board. Schedules live in Home Assistant.
- A failsafe timer, unsolicited daily reports and staggered group acknowledgements in the candle.
- Mains or rechargeable power, or other light types.
- Security beyond preventing a neighbour's identical system from switching the candles (a network ID in each packet).

## Anti-goals

- Batteries that last weeks instead of months because the listener draws too much.
- A candle that cannot be switched off by hand when Home Assistant is down.

## Constraints

- EU sub-GHz rules: the 868 MHz SRD band is duty-cycle limited, so commands are few and short.
- The CC1101 accepts at most 3.6 V. 2xAA (NiMH 2.2-2.8 V, alkaline 2.0-3.2 V) is fine directly. 3xAAA (up to 4.5 V) needs an LDO or regulator, or a rebuild as 2xAAA.
- The MCU runs down to about 2 V, which rules out parts that need 2.7 V or more (the CH32V003).
- The original plastic base is reused as it is, with its battery bay and exposed contacts. The original board is removed.
- The MCU and CC1101 sit in the hollow shell above the base, on short wires to the battery contacts, with the antenna clear of the cells.
- The original switch window holds a small SPST switch from the user's stock that cuts the battery, so OFF means zero drain.
- Every part is ordered from AliExpress, so the design uses only commodity parts with several sellers (CC1101 breakout modules, small ARM or AVR MCUs, small MOSFETs, ESP32 boards). Cheap CC1101 modules come in 433 and 868 MHz variants and clones exist, so the first order is a handful of modules whose sleep and wake-on-radio current is measured before ordering in volume. Parts that need a specific unusual chip or a hard-to-source antenna are avoided.

## Approaches considered

1. **Sub-GHz radio with wake-on-radio (CC1101) plus a tiny MCU, ESP32 with CC1101 running ESPHome on the Home Assistant side. Chosen.** Cheapest per candle, low standby current, two-way, full control of the LED.
2. **Zigbee sleepy end device.** Appears in Home Assistant with no custom transmitter or protocol, but needs mains routers in range of every candle, vendor-SDK firmware and a module that costs more per candle. It becomes the better choice if routers already cover the rooms.
3. **125 kHz wake-up receiver (AS3933).** Instant wake at about 3 uA, but range is a few metres and it needs antenna coils and a custom transmitter.

## Design

```
Candle:  battery contacts -> slide switch (battery cut) -> MCU (deep sleep) -> MOSFET -> LED
                                                           CC1101 (wake-on-radio) -> interrupt -> MCU
Hub:     Home Assistant <-> ESP32 + CC1101 (ESPHome)
```

- The CC1101 samples the air every ~1 s. It wakes the MCU only on a packet carrying the network ID and the candle's address or the group address.
- Commands are on/off, brightness when the LED can be dimmed, and query. The hub sends a preamble long enough to span the poll interval, and a group address lets one transmission reach every candle.
- A candle replies to a command addressed to it with one acknowledgement carrying state and battery voltage.
- A group command gets no acknowledgements. The hub confirms each candle afterwards with a query at its own pace, within the duty-cycle limit.
- The ESPHome side exposes each candle to Home Assistant as a light entity plus a battery sensor.
- The wick LED is kept and driven through a logic-level MOSFET. If it flickers by itself, it stays on or off. If it does not, the MCU produces the flicker with PWM.

## Decisions I made (not yet confirmed with the user)

- 868 MHz rather than 433 MHz (433 MHz modules are the more common AliExpress listing, so the seller's stated frequency is checked, and 433 MHz stays an acceptable fallback within its own duty-cycle rules).
- Poll interval of about 1 s, trading latency against listening current.
- Battery level is reported as a measured voltage, converted to a percentage on the hub per battery type, since NiMH voltage stays nearly flat until the cell is nearly empty.
- Standby go/no-go limit of about 100 uA, since the lit LED draws far more and sets battery life.
- The PY32F002A as the first MCU to try.

## Left to the builder

- Measuring real average listening current at the chosen poll interval before ordering in volume.
- The wick LED test (does it flicker when powered directly) and its current draw.
- Packet format, retry policy and addressing scheme within the duty-cycle limit.
- Where the MCU and CC1101 sit in the shell, from the inner shell dimensions.
- Whether the 3xAAA candles are rebuilt to 2xAAA, regulated, or skipped.
- Candle count is not yet known.
