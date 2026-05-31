# Changes vs Upstream

This document tracks all modifications made in this experimental fork compared to the [upstream MeshCore](https://github.com/meshcore-dev/MeshCore) repository.

## ESP32 Blob Storage Optimization

**Files:** `examples/companion_radio/DataStore.cpp`, `examples/companion_radio/DataStore.h`

### Problem
On ESP32, each contact's advert blob was stored as an individual file in `/bl/<hex_key>`. With SPIFFS using 256-byte pages, this wasted significant flash space:
- 160 contacts × 256 bytes = ~41 KB for ~13 KB actual data

### Solution
Replaced per-key files with a single `/adv_blobs` file containing fixed-size `BlobRec` records (matching the NRF52/STM32 approach):
- `BlobRec` = 157 bytes (timestamp 4 + key 7 + len 1 + data 145)
- 160 contacts × 157 bytes = ~25 KB (saves ~30 KB)
- 350 contacts × 157 bytes = ~55 KB (fits in 128 KB SPIFFS partition)

### Platform Guard (CRITICAL)
The blob functions (`getBlobByKey`, `putBlobByKey`, `deleteBlobByKey`) are wrapped in platform-specific guards:
```cpp
#if defined(NRF52_PLATFORM) || defined(STM32_PLATFORM)
  // NRF52/STM32: single /adv_blobs file with fixed-size records
#elif defined(ESP32)
  // ESP32: same approach + migration from old /bl/ files
#else
  // RP2040/other: one file per key in /bl/<hex>
#endif
```

**⚠️ WARNING:** When porting NRF52 changes to ensure all three platform paths have correct functions. Missing guards cause "redefinition" compile errors.

## T-Beam SX1262/SX1276 Enhancements

**Files:** `variants/lilygo_tbeam_SX1262/platformio.ini`, `variants/lilygo_tbeam_SX1276/platformio.ini`

| Parameter | Upstream | Our Fork |
|---|---|---|
| `MAX_GROUP_CHANNELS` | 8 | **40** |
| `MAX_CONTACTS` | 160 | **350** |

Both values are safe for ESP32 (sufficient RAM and SPIFFS flash).

## Observer Builds

**CI:** `.github/workflows/build-companion-firmwares.yml`

Every release builds **two variants** of each firmware:
- **Standard** — normal firmware
- **Observer** — with `-D MESH_PACKET_LOGGING=1` (filename suffix: `-obs`)

Observer builds log all mesh packet details (type, route, SNR, RSSI, hash) via Serial for debugging. File size impact is negligible (~a few hundred bytes).

## CI Improvements

- `.github/workflows/pr-build-check.yml`: Added `workflow_dispatch` trigger
- `.github/workflows/update-flasher-releases.yml`: Auto-updates `releases.json` in the [web flasher fork](https://github.com/HermestoAizales/flasher.meshcore.io) on every GitHub Release

## Known Issues

- NRF52/RP2040/STM32 UF2/Hex conversion fails with local PlatformIO (toolchain compatibility). CI builds work correctly.
- `wio-e5-mini`: Build fails due to missing `SubGhz.h` (upstream STM32WL platform issue)
