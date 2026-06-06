# Changes vs Upstream

This document tracks all modifications made in this experimental fork compared to the [upstream MeshCore](https://github.com/meshcore-dev/MeshCore) repository.

## T-Beam Extended Configuration

**Files:** `variants/lilygo_tbeam_SX1262/platformio.ini`, `variants/lilygo_tbeam_SX1276/platformio.ini`

- `MAX_CONTACTS=160` (vs upstream default 32 in BaseChatMesh.h, 100 in upstream T-Beam)
- `MAX_GROUP_CHANNELS=18` (vs upstream default 8)
- Empirically tested on SX1262: 160/18 stable, 160/20 causes DRAM overflow by 48 bytes

## Observer Builds

**Files:** `.github/workflows/build-companion-firmwares.yml`

- Builds standard firmwares first, then builds observer variants with `-D MESH_PACKET_LOGGING=1`
- Observer outputs get `-obs` filename suffix (e.g., `Tbeam_SX1262_...-obs-merged.bin`)
- Both variants merged into final release artifacts
- Observer variant logs all mesh packets via Serial (type, route, SNR, RSSI, hash)

## CI Improvements

**Files:** `.github/workflows/update-flasher-releases.yml`, `.github/workflows/pr-build-check.yml`

- Auto-update `releases.json` in Webflasher fork on new release
- PR build check: removed path filters, added `workflow_dispatch`, added T-Beam to matrix

## Security Fixes (pending)

Security fixes for findings from the MeshCore security audit are being prepared
for upstream submission. See `~/meshcore/securityaudit/FINDINGS.md` for details.

**⚠️ Experimental build — use at your own risk.**
