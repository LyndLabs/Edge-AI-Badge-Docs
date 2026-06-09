# Partition Table & OTA

The badge uses a custom 16 MB partition layout with A/B OTA slots, a dedicated AI model storage area, and a FATFS volume for user data.

## Layout

| Name | Type | Offset | Size | Notes |
|------|------|--------|------|-------|
| *(bootloader)* | — | `0x000000` | 36 KB | Fixed by IDF; not in the partition table |
| `nvs` | data/nvs | `0x009000` | 32 KB | Key-value store for app settings and device config |
| `otadata` | data/ota | `0x011000` | 8 KB | Tracks which OTA slot is active |
| `phy_init` | data/phy | `0x013000` | 4 KB | RF calibration data |
| `ota_0` | app/ota_0 | `0x020000` | 4.5 MB | Firmware slot A |
| `ota_1` | app/ota_1 | `0x4A0000` | 4.5 MB | Firmware slot B |
| `edge-ai` | data/fat | `0x920000` | 1 MB | USB Mass Storage volume; hosts `settings.txt` |
| `coredump` | data/coredump | `0xA20000` | 64 KB | Crash dump capture |
| `models` | data/0x40 | `0xA30000` | 1 MB | Raw AI model blobs (mmap'd at runtime) |
| `vfs` | data/fat | `0xB30000` | ~4.81 MB | FATFS: Boop journal, MicroPython storage, sketches |

## OTA Slots

The badge uses A/B OTA — two equal firmware slots (`ota_0` and `ota_1`) at 4.5 MB each. The ESP-IDF bootloader reads `otadata` to determine which slot to boot. On a successful OTA update:

1. New firmware is written to the inactive slot
2. `otadata` is updated to mark the new slot as pending
3. On next boot, the bootloader launches the new slot
4. The app calls `esp_ota_mark_app_valid_cancel_rollback()` after confirming the new firmware works
5. If the app never confirms (e.g., it crashes on boot), the bootloader rolls back to the previous slot

There is no factory partition. When `otadata` is invalid (e.g., on a brand-new badge or after a full wipe), the bootloader defaults to `ota_0`.

OTA delivery (USB DFU and Wi-Fi OTA) is on the firmware roadmap.

## AI Model Storage

The `models` partition (1 MB, custom subtype `0x40`) is reserved for on-device AI model blobs. At runtime it is memory-mapped via:

```c
esp_partition_find_first(ESP_PARTITION_TYPE_DATA, 0x40, "models");
esp_partition_mmap(..., SPI_FLASH_MMAP_DATA, ...);
```

The partition is aligned to a 64 KB boundary so the MMU page mapping lines up cleanly. Models stored here can be read directly by an inference runtime without copying to RAM.

## USB Mass Storage (`edge-ai` partition)

The `edge-ai` partition is a 1 MB FATFS volume exported to a host PC via the TinyUSB MSC class. When you plug in the badge while running normal firmware, your computer sees it as a USB drive. It currently hosts `settings.txt` for user-configurable device settings.

## FATFS (`vfs` partition)

The `vfs` partition (~4.81 MB) is mounted at `/vfs` and shared between:

| File | Size envelope | Contents |
|------|---------------|----------|
| `boops.pb` | up to ~3 MB | Boop peer journal (nanopb-encoded) |
| `me.pb` | ≤ 64 KB | Local outgoing Boop manifest |
| `boops_lastfail.bin` | ≤ 64 KB | IR frame dump on Boop error paths |
| MicroPython storage | remainder | MicroPython file system |

The Boop journal evicts the oldest entry (by last-seen time) when the partition reaches 95% capacity, measured against the partition as a whole — not just the Boop slice — so it yields space to MicroPython before filling up.

## NVS Encryption

NVS encryption (`CONFIG_NVS_ENCRYPTION`) is intentionally disabled in v1. If encryption is needed, add an `nvskey` partition at `0x9000` and shift downstream partitions up by 0x1000.
