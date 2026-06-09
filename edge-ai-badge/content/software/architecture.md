---
title: "Architecture"
description: "System architecture of the Edge AI Foundation Badge firmware."
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 230
toc: true
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

The firmware runs on ESP-IDF with FreeRTOS and is structured in three layers: a hardware abstraction layer, a framework layer, and the application layer.

## Layers

```
┌─────────────────────────────────────────────────┐
│                  Applications                    │
│  launcher · boop · maze · hello_world · counter  │
│         (src/apps/)                              │
├─────────────────────────────────────────────────┤
│               App Manager Framework              │
│   app_t / scene_t lifecycle · input dispatch     │
│   LVGL task · scene stack · group/focus          │
│         (src/app_manager/)                       │
├─────────────────────────────────────────────────┤
│           Hardware Abstraction Layer             │
│  buttons · IMU · battery · IR · LEDs · motor     │
│  board_imu() · board_battery() · board_ir()      │
│         (src/hardware/ + src/board/)             │
└─────────────────────────────────────────────────┘
              ESP-IDF / FreeRTOS / LVGL
```

## FreeRTOS Tasks

Three long-lived tasks handle the main runtime:

| Task | Core | Role |
|------|------|------|
| `gui_task` | Core 0 | LVGL rendering loop; owns all widget operations |
| `input_task` | Core 1 | Polls buttons at 10 ms cadence; posts events to input queue |
| `boop_task` | Core 1 | IR frame RX/TX; drives the Boop peer exchange FSM |

All LVGL operations — widget creation, timers, animations — run exclusively on the LVGL task. No mutex is needed inside scene hooks because the scene lifecycle is dispatched from within that task.

## App Manager

The app manager (`src/app_manager/`) is the framework that hosts apps. It owns:

- The **app registry** — a flat list of `app_t` structs registered at boot
- The **scene stack** — a push/pop stack of active UI scenes
- The **LVGL keypad group** — focus management for button-driven navigation
- **Lifecycle dispatch** — calling `on_enter`, `on_show`, `on_hide`, `on_exit`, `on_tick`, `on_input` on scene hooks in the correct order

### App and Scene Model

An **app** (`app_t`) is a registered unit with a stable ID, a root scene, and optional lifecycle hooks (`on_install`, `on_launch`, `on_close`). An app is what appears in the launcher.

A **scene** (`scene_t`) is a unit of UI under one full-screen LVGL container. Apps can push sub-scenes onto the stack (e.g., a settings sub-menu inside a game) and pop back. Each scene declares:

```c
static const scene_t my_scene = {
    .on_enter = my_on_enter,   // build widgets, alloc heap
    .on_show  = my_on_show,    // start timers, add to focus group
    .on_hide  = my_on_hide,    // cancel timers
    .on_exit  = my_on_exit,    // free heap (must tolerate partial init)
};
```

The manager allocates scene state in PSRAM (`heap_caps_calloc`) and passes it to every hook via `ctx->scene_state`. State is zero-initialized — NULL pointers are safe to check before freeing in `on_exit`.

### Navigation API

Apps navigate by calling the manager API from inside `on_input`, `on_tick`, or LVGL event callbacks:

```c
am_scene_push(&my_sub_scene);   // push a sub-scene
am_scene_pop();                  // back one level
am_launch(&app_counter);         // switch to another app
am_return_to_launcher();         // back to the launcher
```

Calling navigation APIs from `on_enter`, `on_show`, `on_hide`, or `on_exit` is rejected — use a one-shot `lv_timer` to defer.

## Hardware Abstraction

Board-specific hardware is accessed through cross-board accessor functions:

| Accessor | Returns | Implemented by |
|----------|---------|----------------|
| `board_imu()` | `imu_t *` | Board adapter in `src/board/` |
| `board_battery()` | `battery_t *` | Board adapter |
| `board_ir()` | `badge_ir_t *` | Board adapter |

Apps use these without `#ifdef` guards on board identity. Board-specific details (pin numbers, peripheral instances) live entirely in the per-board adapter files.

## Boop — IR Peer Exchange

Boop is the badge's flagship feature: point two badges at each other to exchange contact cards (name, title, role, avatar) over IR and BLE.

**Transport split:**
- **IR** carries small structured blobs (Bio text fields, ~hundreds of bytes) using a custom NEC-based protocol with CRC32 framing and stop-and-wait delivery
- **BLE** carries large binary blobs (avatar images) over an encrypted GATT connection keyed by an LTK exchanged during the IR session

**Protocol flow:**
1. Initiator opens the Boop app and begins beaconing (~400–700 ms interval)
2. Receiver sees the beacon at the launcher and gets a consent modal (Accept / Deny)
3. On Accept: manifest exchange → LTK exchange → IR data pull → BLE avatar pull → Done
4. Journal records the exchange keyed by peer UID; re-meeting the same peer skips already-held blobs

The Boop FSM (`src/apps/boop/boop_proto.c`) runs on its own task, independent of the LVGL render cadence.

## Display Stack

LVGL 9.x runs as the UI framework on top of the GC9A01 SPI driver (`src/hardware/display.*`). The LVGL port uses:
- A DMA-backed SPI flush callback for the GC9A01
- A double-buffer draw configuration backed by PSRAM
- A 1 ms tick timer feeding `lv_tick_inc`

The round 240×240 canvas is the full usable draw area. Apps that need custom pixel rendering allocate a canvas buffer in PSRAM and use the LVGL 9 layer draw API.

## Storage

| Partition | Mount | Contents |
|-----------|-------|----------|
| `nvs` | (NVS API) | Per-app settings, device config |
| `edge-ai` | USB MSC | Exposed as a USB drive; hosts `settings.txt` |
| `models` | mmap | Raw AI model blobs (1 MB) |
| `vfs` | `/vfs` | FATFS: `boops.pb`, `me.pb`, MicroPython storage |

See [Partition Table & OTA](../partition-table/) for the full layout.
