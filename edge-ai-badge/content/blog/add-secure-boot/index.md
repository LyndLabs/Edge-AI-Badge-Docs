---
title: "Add Secure Boot to Any Project"
description: "Using OPTIGA-TRUST-M to make any microcontroller project secure for IoT."
summary: "Using OPTIGA-TRUST-M to make any microcontroller project secure for IoT."
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 20
categories: []
tags: []
contributors: []
pinned: false
homepage: false
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

Most hobbyist and prototyping microcontroller projects ship with no firmware integrity guarantees — if an attacker can reflash your device, they own it. Secure boot closes that door by verifying each firmware image against a trusted signature before execution. With Infineon's **OPTIGA™ TRUST M** security chip, you can bolt this capability onto almost any MCU without redesigning your hardware.

## What Is OPTIGA TRUST M?

OPTIGA TRUST M is a hardened security element that handles cryptographic operations — key generation, X.509 certificate storage, ECDSA signing and verification — entirely inside tamper-resistant silicon. Your host MCU communicates with it over I²C, so it drops into any design that has two spare GPIO pins.

Key capabilities relevant to secure boot:

- **Asymmetric key storage** — private keys never leave the chip
- **Certificate provisioning** — stores X.509 certs in protected object slots
- **Cryptographic verification** — ECDSA P-256/P-384 signature checks in hardware

## The Secure Boot Flow

A minimal implementation follows three steps at power-on:

1. **Root of trust** — A small, read-only bootloader (ideally in protected flash or ROM) holds the OPTIGA public key or certificate fingerprint.
2. **Image signing** — During your build pipeline, the firmware binary is hashed and signed with the private key held inside OPTIGA TRUST M.
3. **Verification at boot** — The bootloader reads the signature appended to the image, sends the hash and signature to OPTIGA TRUST M over I²C, and only jumps to the application if verification passes.

```
Build machine                        Device at power-on
─────────────────                    ──────────────────
firmware.bin                         ROM bootloader (trusted)
     │                                      │
     ▼                                      ▼
 hash (SHA-256)  ──sign via OPTIGA──►  appended .sig
                                            │
                                      send hash + sig
                                      to OPTIGA TRUST M
                                            │
                                      ECDSA verify
                                            │
                                   pass ──► jump to app
                                   fail ──► halt / recovery
```

## Wiring It Up

OPTIGA TRUST M speaks I²C at address `0x30` by default. Connect SDA, SCL, and the reset line to your MCU, then pull in Infineon's open-source [optiga-trust-m host library](https://github.com/Infineon/optiga-trust-m).

```c
#include "optiga/optiga_util.h"
#include "optiga/optiga_crypt.h"

optiga_lib_status_t verify_firmware_signature(
    const uint8_t *hash,   uint16_t hash_len,
    const uint8_t *sig,    uint16_t sig_len)
{
    optiga_crypt_t *crypt = optiga_crypt_create(0, NULL, NULL);
    optiga_lib_status_t status;

    public_key_from_host_t pub_key = {
        .public_key       = FIRMWARE_SIGNING_PUBLIC_KEY,
        .length           = sizeof(FIRMWARE_SIGNING_PUBLIC_KEY),
        .key_type         = (uint8_t)OPTIGA_ECC_CURVE_NIST_P_256,
    };

    status = optiga_crypt_ecdsa_verify(
        crypt, hash, hash_len, sig, sig_len,
        OPTIGA_CRYPT_HOST_DATA, &pub_key);

    optiga_crypt_destroy(crypt);
    return status;
}
```

Call this from your bootloader before the jump. If `status != OPTIGA_LIB_SUCCESS`, stall or enter a recovery mode.

## Integrating with Your Build Pipeline

Sign firmware automatically at the end of your CMake or Makefile build:

```bash
# Generate signature using OPTIGA TRUST M via host tool
optiga_sign --key-slot 0xE0F0 \
            --input firmware.bin \
            --output firmware.bin.sig

# Append signature to binary for on-device verification
cat firmware.bin firmware.bin.sig > firmware_signed.bin
```

The private key never leaves the OPTIGA chip — the host tool sends the hash over I²C and retrieves only the signature.

## Why This Matters for IoT

Devices in the field get firmware updates over UART, BLE, or Wi-Fi. Without verified boot, a compromised update server or a man-in-the-middle can push malicious firmware. With OPTIGA TRUST M as your root of trust:

- **Only signed images run** — even physical access to the flash bus can't load unsigned code
- **Keys are hardware-bound** — a compromised build server can't forge signatures without the chip
- **Audit trail** — certificate slots can be rotated and logged for lifecycle management

The Edge AI Foundation Badge integrates OPTIGA TRUST M precisely for this reason: AI models and inference pipelines running at the edge are high-value targets, and the hardware root of trust ensures only authorized firmware reaches production devices.

## Next Steps

- Clone the [optiga-trust-m host library](https://github.com/Infineon/optiga-trust-m) and run the `examples/` for your MCU family
- Review Infineon's [Secure Boot Application Note](https://www.infineon.com/optiga-trust-m) for chip provisioning during manufacturing
- Pair secure boot with encrypted storage (OPTIGA also supports AES key wrapping) to protect model weights and configuration at rest
