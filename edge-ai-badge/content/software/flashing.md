---
title: "Programming the Badge"
description: "How to flash the Edge AI Foundation Badge using a pre-built image or from source."
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 210
toc: true
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---
If you have an unprogrammed badge, there's only 4 steps to upload our pre-built firmware!
1. [Download the pre-built 16MB firmware file]() on your computer.
2. Put your badge in [programming mode]().
3. Choose your programming method ([Chrome]() / [Command-Line]()).
4. [Reset your badge](), and enjoy!

---
### Enter Programming Mode
1. Press the badge `reset` button once.
2. Hold down `BTN-5` while plugging badge into your computer's USB port.
3. That's it!

<img src="/buttons.png" alt="Chrome flashing tool" />

---
### Programming Methods
>
<details>
<summary><strong>Option 1: Program through Chrome</strong></summary>

[DevKitty Flasher Tool](https://update.devkitty.io/) programs directly from the Chrome browser over WebSerial — no installation needed.

1. Open the [firmware flasher]() in Chrome (or any Chromium-based browser).
2. Click **Connect**, select your badge's serial port, which probably says something like `USB-JTAG`.
3. Keep `Offset` at `0x0`.
4. When flashing completes, reset the badge!

<img src="/flash.png" alt="Chrome flashing tool" />
</details>

<details>
<summary><strong>Option 2: Program through Command-Line</strong></summary>

`esptool` is the standard Espressif command-line flasher. It gives you full control over the flash operation.

**Install:**
```bash
pip install esptool
```

**Flash the full factory image:**
```bash
esptool.py --chip esp32s3 --port /dev/ttyACM0 write_flash 0x0 firmware-factory-16mb.bin
```

Replace `/dev/ttyACM0` with your port (`COM3` on Windows, `/dev/cu.usbmodemXXXX` on macOS). The factory image covers the full 16 MB and is safe to flash on a blank or previously-programmed badge.

</details>
