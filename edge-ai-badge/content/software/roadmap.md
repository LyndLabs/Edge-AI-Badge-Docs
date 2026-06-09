---
title: "Roadmap"
description: "Upcoming features and platform goals for the Edge AI Foundation Badge."
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 250
toc: true
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

The Edge AI Foundation Badge is a platform, not a finished product. Here's what's coming.

## Edge AI & On-Device Inference

The badge has a dedicated 1 MB `models` partition and 8 MB of PSRAM — enough headroom to run real inference at the edge. We're building toward running AI and ML models directly on the ESP32-S3, without cloud connectivity.

Planned work includes integrating a lightweight inference runtime, tooling to load and swap model blobs over USB or OTA, and example applications that demonstrate on-device classification, anomaly detection, and signal processing.

We also plan to host hands-on **IoT and edge AI/ML workshops at future conferences**, where attendees can train, deploy, and run models on their badges in real time.

## CTF Support

The badge is being designed to serve as both a CTF challenge target and a platform for hosting challenges. We're building the infrastructure for the badge to participate in capture-the-flag competitions — whether as a node that contestants attack and defend, or as a tool for wireless/hardware challenge categories.

Details will be announced as the platform matures.

## Secure Boot & Embedded Security

Security is a first-class concern. We're actively adding **secure boot support** and collaborating with embedded security vendors — including **Thistle Technologies** and **Infineon** — to bring proper secure boot, firmware signing, and hardware-backed security to the badge platform.

A secure boot module from this collaboration can be physically connected to the badge's expansion header. This makes the badge a hands-on teaching tool for real-world embedded security practices, not just a demo.

## OTA Updates

Wi-Fi OTA (over-the-air firmware updates) is on the roadmap alongside the existing USB DFU path. The A/B partition layout is already in place — the badge can receive a new firmware image into the inactive slot and roll back automatically if the update fails.

## MicroPython

The `vfs` FATFS partition is named and structured to be compatible with MicroPython's default ESP32 port. A MicroPython build for the badge, with Python bindings for the Boop journal API and hardware peripherals, is a planned milestone.

## Get Involved

If you're interested in integrating your project, research, or curriculum with the Edge AI Foundation Badge platform — whether that's running your models on it, building challenge scenarios for it, or contributing firmware — reach out.

The firmware is open source at [github.com/LyndLabs/Edge-AI-Badge](https://github.com/LyndLabs/Edge-AI-Badge).
