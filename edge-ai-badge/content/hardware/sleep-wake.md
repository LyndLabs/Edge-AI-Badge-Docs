---
title: "Sleep & Wake"
description: "Deep sleep configuration, wake sources, and GPIO hold design for the Edge AI Foundation Badge."
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: true
weight: 130
toc: true
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

{{< callout context="note" >}}These docs are written for developers. New to the badge? Check out our [blog](/blog/) for guides and project walkthroughs to get started.{{< /callout >}}

The ESP32-S3 divides its GPIOs into two domains with different sleep behavior. Understanding this split is essential for building apps that sleep correctly and wake reliably.

## RTC vs Digital GPIOs

| Domain | GPIOs | Deep sleep behavior |
|--------|-------|---------------------|
| **RTC** | GPIO0–GPIO21 | Stay powered; can be wake sources; state preserved with `gpio_hold_en`; accessible by ULP coprocessor |
| **Digital** | GPIO22–GPIO48 | Power domain is gated off during deep sleep; cannot be wake sources |

On the badge, all peripherals that need to be either a wake source or held in a specific state across sleep are intentionally routed to RTC-capable GPIOs. See the [Pinout](../pinout/) for the full table.

## Wake Sources

| Wake scenario | ESP-IDF call | Why it works |
|---------------|-------------|--------------|
| Any button press | `esp_sleep_enable_ext1_wakeup((1ULL<<BTN_L)\|(1ULL<<BTN_1)\|..., ESP_EXT1_WAKEUP_ANY_LOW)` | All 8 buttons are on RTC GPIOs, idle-high with pull-ups — any press pulls a pin LOW |
| USB cable attach | `esp_sleep_enable_ext0_wakeup(CHG_PGOOD, 0)` | BQ24079 drives `/PGOOD` LOW when VBUS is present; CHG_PGOOD is RTC GPIO14 |
| Motion / shake | Program LIS2DH12 INT1 motion generator, then `esp_sleep_enable_ext0_wakeup(INT_IMU, 0)` | INT_IMU is RTC GPIO3; the IMU stays powered from 3.3 V while the SoC sleeps |
| IR command received | `esp_sleep_enable_ext0_wakeup(IR_RX, 0)` | IR_RX is RTC GPIO6; the TSOP75338 is always powered and demodulates the carrier |
| Charge complete | `esp_sleep_enable_ext0_wakeup(CHG_STAT, 0)` | CHG_STAT is RTC GPIO21 |

## GPIO Hold — Preventing Spurious Outputs

When the SoC enters deep sleep, digital GPIOs lose their driven state. Use `gpio_hold_en` + `gpio_deep_sleep_hold_en` to latch an RTC GPIO's output level across the sleep transition.

**Backlight** — hold the panel off to prevent a brief flash during sleep entry:
```c
gpio_set_level(ALPHA_PIN_LCD_BL, 0);
gpio_hold_en(ALPHA_PIN_LCD_BL);
gpio_deep_sleep_hold_en();
```

**Haptic motor** — hold the gate LOW to prevent a spurious vibration:
```c
gpio_set_level(ALPHA_PIN_MOTOR, 0);
gpio_hold_en(ALPHA_PIN_MOTOR);
```

Both `LCD_BL` (IO10) and `MOTOR` (IO11) are on RTC GPIOs, so the hold takes effect.

## What You Cannot Do

Some desirable sleep behaviors are blocked by GPIO domain constraints:

| Scenario | Why it doesn't work |
|----------|---------------------|
| Wake on UART RX activity | UART_RX is IO44 (digital-only). Route RX through an RTC pin via external logic if needed. |
| Animate WS2812 LEDs from ULP | LED_DATA is IO42 (digital-only); the RMT peripheral is also in the digital domain. Power LEDs off before sleep; re-enable on wake. |
| Wake on i2c activity (SDA/SCL) | SDA/SCL are IO38/IO39 (digital-only). Use the LIS2DH12 INT1 line (IO3) for sensor-driven wake instead. |

## Shutdown Sequence

Before calling `esp_deep_sleep_start()`, the firmware needs to quiesce the three long-lived FreeRTOS tasks and power down peripherals in order. The stop primitives (`input_stop` → `gui_stop` → `lvgl_port_stop`) handle task signaling. After the tasks are stopped:

1. Turn off the LCD backlight and `gpio_hold_en` the pin
2. Drive the motor gate LOW and `gpio_hold_en` the pin
3. Configure the desired wake source(s) via `esp_sleep_enable_*_wakeup`
4. Call `esp_deep_sleep_start()`

A power-coordinator module that owns the full sequencing (battery-low timeout, idle timeout, explicit sleep request) is on the firmware roadmap. The stop primitives are the building blocks.
