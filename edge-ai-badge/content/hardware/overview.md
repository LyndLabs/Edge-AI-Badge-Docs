---
title: "Overview"
description: "Onboard components of the Edge AI Foundation Badge."
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

The Edge AI Badge is a self-contained, wearable compute platform designed for edge AI, wireless communication, and evaluating other embedded systems!

---

## Microcontroller
`ESP32-S3 N16R8`  

The badge is built around the **ESP32-S3**, a ubiquitous $4 Wi-Fi chip that packs machine learning, security peripherals, and [much more]().  We use the `WROOM-N16-R8` variant which has 16MB Flash Storage and 8MB PSRAM - very necessary for wireless comms & edge inference.

The ESP32-S3's native USB port handles both [firmware flashing]() (USB DFU) and USB Mass Storage for easy file access - the badge mounts as a flash drive on your computer!

## Display & LED's
`GC9A01 — 1.28" 240×240 Round SPI LCD`

A round display that lined up well with our logo design motif - in case you didn't notice, the badge is an extension of our logo!

We [implemented some tricks]() to get it working, and wrote a lightweight [display driver]() on top of the [LVGL](https://lvgl.io/) graphics library.

`WS2812B — 10× RGB`
Commonplace RGB LED's that use a only one pin to drive indefinite, individually addressable animations!   

## Accelerometer 
`LIS2DH12 — 3-Axis IMU`

An I2C MEMS accelerometer from STMicroelectronics.  We use it for [UI navigation]() and detecting [badge inversion]() on a user-programmable interrupt.

But you can also use it for [step counting](), [free-fall detection](), or [gesture recognition]()!

## Infrared Transceiver
`TSOP75338 (RX)` · `VSMY14940 (TX)`

Basically turns your badge into a TV Remote - exchange contact info with new friends or use it to [control appliances in your house]()!

## Vibration Motor
`Haptic Motor`

A small vibration motor for tactile feedback.  It buzzes when you receive notification to get your attention without making noise! ...unless you wanted to [hack it into a synth]().

## Battery Management
`BQ24079 — Li-Ion Charger`

Charges & protects the onboard battery over USB-C, and lets us track battery level with some [clever signal processing](). 

## Buttons
`8× Tactile Switches`

The badge is designed for input on the faceplate, but we added 8 physical buttons as a fallback. 


## USB-C

Allows the badge to mount as a flash drive where you can modify files, write Python scripts, and read serial logs during development.