# Building Tessera on ESPHome 2026.10

Checked 2026-10-30 against ESPHome 2026.10.1 (the latest patch; 2026.10.0 came out 2026-10-21).

- **The image exists:** `ghcr.io/esphome/esphome:2026.10.1` (and `2026.10.0`) are published; there is no 2026.10.2.
- **Config:** `esphome config` passes on crowpanel7, guition and waveshare7b. Two new warnings, neither blocks:
  the core's `image:` block is deprecated (removed in 2027.1.0; use `image: - platform: file`), and 2026.12.0 stops the
  legacy redaction heuristic for `key`/`transparency_key` fields (ESPHome's own). A stub `api` key of all zeros is now
  refused, and one that is not 32 bytes of base64 is too, as before.
- **Compile:** it does not link as is. `components/smart_display/loop_record.cpp` used `esphome::loop_task_handle`,
  which 2026.10 replaced with `esphome_main_task_handle` (`esphome/core/main_task.h`). Fixed on the `esphome-2026-10`
  branch with a `__has_include`, which keeps the old symbol for 2026.9 and older. With it, waveshare7b and crowpanel7
  compile on 2026.10.1.
- **What the branch changes:** `screen_manager/Dockerfile` FROM, `requirements.txt`, the README line, a CHANGELOG line, and
  that fix. The boards' `min_version` pins stay: they are floors. Nothing in `screen_manager/app` (build cache, release
  assets) hardcodes a version: it reads the running ESPHome's.
- **Why it is not a drop-in:** the fix is in `components/smart_display`, so by `tools/affected_boards.py` it reaches
  every board and needs firmware 0.60.0 (`SCREEN_FIRMWARE_VERSION`, `FIRMWARE_VERSION`, `boards.json`, app version and
  CHANGELOG heading). I did not bump it: that is a release decision.
- **Not done:** the add-on image was not built (no Docker here), so the pip pins in the Dockerfile (aiortc, av,
  pillow-heif, against ESPHome's own cryptography/Pillow) are unchecked on 2026.10.1; `tools/check.sh --firmware
  --sample`, the 2026.6.2 min-version leg, and any flashing were not run; only two boards compiled.
