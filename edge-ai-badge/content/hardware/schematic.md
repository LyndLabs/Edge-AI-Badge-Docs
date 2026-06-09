---
title: "Schematic"
description: "Schematic and full GPIO pin map for the Edge AI Foundation Badge"
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 110
toc: true
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

{{< callout context="note" >}}These docs are written for developers. New to the badge? Check out our [blog](/blog/) for guides and project walkthroughs to get started.{{< /callout >}}

[![Schematic](/v0-Badge-SCH.png)](/v0-Badge-SCH.png)

## Pin Map

**RTC** column indicates whether a GPIO is [RTC-capable](), which can:
- Wake the ESP32 from deep sleep
- Be driven by the ULP coprocessor
- Keep output state even during deep sleep

Non-RTC GPIOs are not accesible in deep sleep :(

| System    | Net | GPIO | RTC  | Notes |
|-----------|-----|-----:|:----:|-------|
| Display | LCD_SCK | IO47 | ✓ | GC9A01 SPI clock |
| Display | LCD_MOSI | IO48 | ✓ | GC9A01 SPI data |
| Display | LCD_DC | IO9 | ✓ | Data / Command select |
| Display | LCD_BL | IO10 | ✓ | Backlight MOSFET — hold LOW before sleep to keep panel off |
| Display | LCD_CS | — | | Tied to GND to autoselect SPI|
| Display | LCD_RST | IO41 | | Display reset |
| I2C | SDA | IO38 | ✓ | |
| I2C | SCL | IO39 | ✓ | |
| IMU | INT_IMU | IO3 | ✓ | LIS2DH12 INT1 detects badge inversion |
| Buttons | BTN_L | IO0 | ✓ | SW9, firmware upgrade button |
| Buttons | BTN_1 | IO7 | ✓ | SW5 |
| Buttons | BTN_2 | IO17 | ✓ | SW6 |
| Buttons | BTN_3 | IO18 | ✓ | SW7 |
| Buttons | BTN_4 | IO2 | ✓ | SW8 |
| Buttons | BTN_6 | IO1 | ✓ | SW10 |
| Buttons | BTN_7 | IO4 | ✓ | SW11 |
| Buttons | BTN_8 | IO5 | ✓ | SW4 |
| Charger | CHG_CE | IO13 | ✓ | BQ24079 charge enable |
| Charger | CHG_PGOOD | IO14 | ✓ | See truth-table |
| Charger | CHG_STAT | IO21 | ✓ | See truth-table |
| Battery | ADC_BAT | IO8 | ✓ | Battery voltage via resistor divider |
| IR | IR_TX | IO40 | ✓ | VSMY14940 Infrared LED |
| IR | IR_RX | IO6 | ✓ | TSOP75338 Infrared Receiver |
| LEDs | LED_DATA | IO42 | ✓ | WS2812 single-wire DIN |
| Motor | MOTOR | IO11 | ✓ | Vibration motor |
| UART | UART_TX | IO43 | ✓ | TXD0 |
| UART | UART_RX | IO44 | ✓ | RXD0 |
| USB | USB_DN | IO19 | ✓ | USB-C D− |
| USB | USB_DP | IO20 | ✓ | USB-C D+ |
| RTC XTAL | XTAL_P | IO15 | ✓ | 32.768 kHz crystal |
| RTC XTAL | XTAL_N | IO16 | ✓ | 32.768 kHz crystal |

## Interrupts
`coming soon`

## I2C Bus + Expansion
We expose the I2C bus and power rails through pogo-pin contacts on the top + bottom PCB layer. A daughterboard or custom PCB art can be screwed onto the badge and act as an input device without soldering!