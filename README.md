# Corne Chocofi with nice!view

ZMK config for a wireless Corne (Chocofi), nice!nano v2 controllers, nice!view displays.

## Build

GitHub Actions builds on every push. Manual run from the Actions tab, or:

```
gh workflow run build.yml -R bliiir/zmk-config-corne-chocofi-with-niceview --ref develop
```

Download the firmware when the run is green:

```
rm -rf ~/Downloads/corne-fw
gh run download -R bliiir/zmk-config-corne-chocofi-with-niceview -D ~/Downloads/corne-fw
ls ~/Downloads/corne-fw/firmware
```

Three files:

- `corne_left nice_view_adapter nice_view-nice_nano__zmk-zmk.uf2`
- `corne_right nice_view_adapter nice_view-nice_nano__zmk-zmk.uf2`
- `settings_reset-nice_nano__zmk-zmk.uf2` - wipes Bluetooth bonds, flash only when pairing is broken

Pushing from this laptop needs the personal SSH key. The default key belongs to the work account and gets "Permission denied":

```
GIT_SSH_COMMAND='ssh -i ~/.ssh/id_ed25519_personal -o IdentitiesOnly=yes' git push
```

## Flash

One half at a time, over USB-C. Plugging in is harmless - it charges and the half works as a wired keyboard.

1. Plug in the half.
2. Double-tap its reset button fast, within about half a second. A drive called NICENANO mounts.
3. Copy the firmware onto the drive. It ejects itself when done. An "ejected improperly" warning or "fcopyfile failed" error is normal.

Left:

```
cp -X ~/Downloads/corne-fw/firmware/corne_left*.uf2 /Volumes/NICENANO/
```

Right:

```
cp -X ~/Downloads/corne-fw/firmware/corne_right*.uf2 /Volumes/NICENANO/
```

Normal update - flash left and right, done. Bluetooth pairing survives.

## Fix broken Bluetooth pairing

Symptom: Corne advertises but never connects, usually after the Mac forgot it. The keyboard still holds the old bond and refuses to pair again.

Try the combo first - press the two innermost keys of the bottom row together (`;` and `,`). That clears the current profile. Then pair from System Settings > Bluetooth.

If that fails, full reset:

1. System Settings > Bluetooth - remove Corne if listed.
2. Left half - double-tap reset, flash `settings_reset`, double-tap again, flash left firmware.
3. Right half - same, with the right firmware.
4. Unplug both. Press reset on both halves at the same time so they find each other.
5. Pair Corne from System Settings > Bluetooth.

Settings reset wipes every profile. Other computers need to pair again.

## Bluetooth combos

Two innermost keys of each row, pressed together:

| row | keys | action |
|---|---|---|
| top | K + J | previous profile |
| home | : + . | next profile |
| bottom | ; + , | clear current profile |

## Troubleshooting

NICENANO drive does not appear:

- Double-tap was too slow. Two separate presses only reboot the keyboard.
- Pressing reset on the half that is not plugged in.
- macOS "Allow accessory to connect?" prompt got dismissed. The bootloader shows up as a new USB device. Set System Settings > Privacy & Security > Allow accessories to connect to "Automatically when unlocked".
- Button dead - bridge RST and GND on the nice!nano twice quickly with tweezers.

`INFO_UF2.TXT` on the drive names the hardware. These boards report `Board-ID: nRF52840-nicenano-v2`.

Build fails with "The selected board is not set up for ZMK" - ZMK main moved to Zephyr 4.1 in Dec 2025. Board is now `nice_nano//zmk`, was `nice_nano_v2`. See https://zmk.dev/blog/2025/12/09/zephyr-4-1

Build fails with "Aborting due to Kconfig warnings" naming `ZMK_USB_LOGGING` - remove `CONFIG_ZMK_USB_LOGGING=y` from `config/corne.conf`.

## Docs

- Flashing - https://zmk.dev/docs/user-setup#flashing-uf2-files
- Flashing problems - https://zmk.dev/docs/troubleshooting/flashing-issues
- Bluetooth behaviors - https://zmk.dev/docs/keymaps/behaviors/bluetooth
