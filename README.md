# Dac Man 5x6 ZMK Config

Firmware and layout files for the Dac Man 5x6 split keyboard powered by nice!nano v2 controllers. The build targets both halves and a small `settings_reset` helper image for clearing Bluetooth pairings.

## Repo Layout
- `config/west.yml` — west manifest that pulls ZMK from upstream.
- `config/boards/shields/dac_man_5x6/` — custom shield definition, matrix transform, overlays, and the **active** keymap used for builds.
- `config/dac_man_5x6.conf` — optional global features (RGB underglow, OLED) you can toggle by uncommenting.
- `build.yaml` — build matrix; left, right, and `settings_reset` firmware are produced for `nice_nano_v2`.
- `.github/workflows/build.yml` — delegates CI builds to the upstream ZMK workflow.

## Layers at a Glance
- **Default**: QWERTY with thumb keys for space/enter and momentary access to raise/lower.
- **Raise**: Bluetooth slot selection/clearing, function row, brackets/paren, bootloader and reset on the inner thumb keys.
- **Lower**: Media controls (play/pause, volume), navigation arrows, numpad cluster, and screen lock.
- Each half defines a top button on `gpio0 29` as reset (left overlay) and sets the left half as the BLE central (`CONFIG_ZMK_SPLIT_BLE_ROLE_CENTRAL`).

## Building with GitHub Actions
Pushes, PRs, or manual runs trigger `.github/workflows/build.yml`, which calls the upstream ZMK user-config workflow. Download the artifacts from the workflow run; each target contains a `zmk.uf2` ready to copy to a `nice_nano_v2`.

## Build Locally with West
> Requires Zephyr/ZMK prerequisites installed. The west workspace expects this repo as the `config/` folder.

```bash
# Set up the workspace (run once)
west init -l config
cd config
west update
west zephyr-export

# Build left and right halves
west build -p -b nice_nano_v2 -- -DSHIELD=dac_man_5x6_left
west build -p -b nice_nano_v2 -- -DSHIELD=dac_man_5x6_right

# Optional: build the settings reset image
west build -p -b nice_nano_v2 -- -DSHIELD=settings_reset
```

After each build, flash by copying `build/zephyr/zmk.uf2` to the `NICE!BOOT` drive that appears when the controller is in bootloader mode.

## Customization Tips
- Edit `config/boards/shields/dac_man_5x6/dac_man_5x6.keymap` to change the layout; it is the file picked up by the shield builds in `build.yaml`.
- Uncomment lines in `config/dac_man_5x6.conf` to enable RGB underglow (`CONFIG_ZMK_RGB_UNDERGLOW`) or the OLED display (`CONFIG_ZMK_DISPLAY`) if your hardware supports them.
- Pairing issues? Flash `settings_reset` to clear stored Bluetooth bonds, then re-flash the regular firmware.
