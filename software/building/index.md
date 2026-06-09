# Building from Source

The firmware is built with [PlatformIO](https://platformio.org/) on top of ESP-IDF. No manual IDF installation is needed — PlatformIO manages the toolchain.

## Prerequisites

- **Python 3.8+** (required by PlatformIO and esptool)
- **PlatformIO Core** (CLI) or the **PlatformIO IDE** extension for VS Code

Install the CLI:
```bash
pip install platformio
```

Or install the [PlatformIO IDE extension](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) in VS Code.

## Clone the Repository

```bash
git clone https://github.com/LyndLabs/Edge-AI-Badge.git
cd Edge-AI-Badge/firmware
```

## Build Environments

The firmware supports several build targets defined in `platformio.ini`:

| Environment | Board | Use |
|-------------|-------|-----|
| `edge_badge_alpha_esp32s3` | Edge AI Badge (alpha) | Production build for the alpha badge |
| `edge_badge_alpha_esp32s3_debug` | Edge AI Badge (alpha) | Debug build with serial logging enabled |
| `replay_echo_esp32s3` | Replay Echo | Build for the Echo development board |
| `replay_echo_esp32s3_debug` | Replay Echo | Debug build for Echo |

For the Edge AI Foundation Badge hardware, use the `edge_badge_alpha_esp32s3` environments.

## Build

```bash
# Production build
pio run -e edge_badge_alpha_esp32s3

# Debug build (enables esp_log output over USB serial)
pio run -e edge_badge_alpha_esp32s3_debug
```

PlatformIO will download the correct ESP-IDF version and Xtensa toolchain on the first run. This takes a few minutes initially.

## Flash

Hold **BTN_L** while connecting USB-C to enter bootloader mode, then:

```bash
pio run -e edge_badge_alpha_esp32s3 --target upload
```

## Monitor Serial Output (Debug Builds)

```bash
pio device monitor --baud 115200
```

In debug builds, `ESP_LOGI` / `ESP_LOGW` / `ESP_LOGE` output appears on the USB CDC serial port. Production builds suppress most log output to reduce overhead.

## Adding a New App

Apps live in `src/apps/`. Each app is an `app_t` struct registered with the app manager at boot. See the [Architecture](../architecture/) page for the app/scene model, and use `src/apps/counter.c` as a minimal starting template.

After creating your app files, register the app in `src/gui/gui.c`:
```c
#include "apps/my_thing.h"
// ...
(void)am_register(&app_my_thing);
```

Build both the production and debug environments before submitting a PR to verify nothing breaks across targets.
