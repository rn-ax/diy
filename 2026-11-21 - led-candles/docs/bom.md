# Parts to order

First order: a bench setup, one rebuilt candle and the hub. Every part is ordered from AliExpress, so search terms are given. Customs and VAT may apply on arrival, so everything goes in one order.

| Part | Qty | Search term | What it does |
|---|---|---|---|
| CC1101 868 MHz module | 6, from two different sellers | "CC1101 868MHz module" | The sub-GHz radio. It sleeps and wakes itself to listen for the network's sync word. Five go to bench and candles, one to the hub. Many listings are 433 MHz, so check the stated frequency. |
| PY32F002A board or chip | 4 | "PY32F002A development board" | The candle's MCU. It runs the core logic, talks to the CC1101 over SPI, drives the LED and reads the battery. It runs down to about 2 V. |
| ST-Link V2 clone | 1 | "ST-Link V2" | Flashes the PY32 over two-wire SWD. |
| ESP32 dev board | 1 | "ESP32-C3 supermini" | The hub. It runs ESPHome and talks to Home Assistant over Wi-Fi. |
| AO3400A MOSFET, SOT-23 | 10 | "AO3400A" | Switches the LED from the MCU pin and allows PWM. It turns on fully at about 2 V on its gate (from general knowledge, so check the datasheet). |
| Warm-white LED, 3 mm | 20 | "3mm warm white LED 2200K" | Test load, and a replacement if the wick LED turns out to flicker by itself. Plain LEDs, since the flicker comes from software. |
| Breadboard, jumpers, headers, resistors | 1 set | "breadboard dupont kit" | Bench wiring, and the 100 ohm resistor for the wick LED test. |

Left off the list:
- A µA meter: the user's multimeter already has a 600.0 µA range.
- A regulator: NiMH and alkaline cells are within the CC1101's 3.6 V limit.
- Battery holders: the original base is reused.
- The battery switch: the user's own stock of switches is used. Any SPST switch small enough for the original switch window works, since it only carries the candle's current.
