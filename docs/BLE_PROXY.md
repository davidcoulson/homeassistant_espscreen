# Bluetooth proxy

A screen can act as a Bluetooth proxy for Home Assistant, with
[esphome-ble-advert-filter](https://github.com/davidcoulson/esphome-ble-advert-filter) in front of it, so the adverts
nobody uses (a neighbour's phone, Apple's background chatter) are dropped on the screen instead of sent over Wi-Fi.
It is a feature file, `packages/features/ble-proxy.yaml`, that no board includes: a screen asks for it.

## Turn it on

Add this to the screen's **Override YAML** (docs/EASY_SETUP.md) and update the firmware. The `ref` is the branch your
screen's own YAML names under `packages:`:

```yaml
packages:
  ble_proxy:
    url: https://github.com/davidcoulson/homeassistant_espscreen
    ref: ble-plugin
    files: [packages/features/ble-proxy.yaml]
```

What it adds: `esp32_ble_tracker` and an active `bluetooth_proxy`, and `ble_advert_filter` with an RSSI threshold of
-80 dBm and Apple's manufacturer ID (0x004C) blocked; HomeKit accessories still pass (`allow_homekit`). Home Assistant
gets a number, **BLE RSSI Threshold**, that tunes the threshold without a build. Substitutions a screen may override:

| Substitution | Default | |
|---|---|---|
| `BLE_RSSI_THRESHOLD` | `-80` | Starting value of the threshold (dBm). |
| `BLE_ALLOW_HOMEKIT` | `true` | HomeKit accessories are not blocked as Apple noise. |
| `BLE_FILTER_REF` | `v2.0.4` | The component's release. |

iBeacons are dropped with the rest of Apple's traffic. To let yours through, add to the Override YAML
`ble_advert_filter: {allow_ibeacon: [{uuid: "…"}]}`. More options (IRKs, name blocks, sensors): the component's
`docs/options.md`.

## Which boards

- It needs **ESPHome 2026.10.0 or newer**: the component hooks into `bluetooth_proxy` as of 2026.10. The add-on builds
  with 2026.10.1 on the `esphome-2026-10` branch; on an add-on with ESPHome 2026.9 the build
  stops at the file's `min_version`.
- ESP32-S3 boards have the radio. The ESP32-P4 boards (crowpanel7, crowpanel101, Waveshare P4, Guition P4) have Bluetooth
  only through the ESP32-C6 (`esp32_hosted`, hardware/esp32p4-c6.yaml); ESPHome's `bluetooth_proxy` runs over that.
  ESP32-C6 firmware must support Bluetooth over the hosted link.
- The 4 MB boards (the CYDs, Hosyond 4.0) have no room: do not use it there.

## How it was checked

- `esphome config` with ESPHome 2026.10.1 (secrets stubbed) on crowpanel7 (P4) and waveshare7b (S3), with the file
  added through `packages:`; both valid, and the merged YAML shows the three components and the number.
- `esphome compile` 2026.10.1 of the same two, on the `esphome-2026-10` branch (this branch's add-on still ships 2026.9.0,
  which cannot build the plugin): both link. waveshare7b: flash 42.0 %, RAM 57.4 %; crowpanel7: flash 45.6 %, RAM 28.1 %.
- `tools/check_packages.py`, `tools/generate_entries.py --check`, and the unit tests named in the commit.

## Not tested

- Nothing was flashed: no scan, no Home Assistant discovery, no filter counters on real hardware.
- Whether the C6's hosted firmware on the CrowPanel exposes Bluetooth, and how much RAM, PSRAM and Wi-Fi/Bluetooth
  coexistence the proxy costs while the screen runs its UI, camera and audio.
- The remote `packages:` form above from the fork's branch (config was checked with a relative `!include`).
- The proxy's three connection slots are ESPHome's default; not tuned for a panel.
