# desk_hub firmware

RMK firmware for the desk_hub macro pad: an ESP32-S3-N16R8 driving a 6×4 matrix
(19 keys) with a single 19-LED WS2812 chain, configurable from Vial over USB.

- `keyboard.toml` — the board: matrix pins, physical layout, keymap.
- `rgb.toml` — the lighting: chain wiring, LED order, effects, defaults.
- `vial.json` — what Vial shows, including `"lighting": "vialrgb"` for the
  Lighting panel.
- `Cargo.toml` — replaces the template's, so the firmware builds against the
  RMK fork that has the per-key RGB engine.

## Build

Pushing to `master` runs `.github/workflows/build.yml`, which generates the
project with `rmkit`, copies `rgb.toml` into it, and builds the merged image.
Download the `desk_hub-firmware` artifact.

The workflow is our own rather than the upstream reusable one for a single
reason: `rmkit` copies `keyboard.toml`, `vial.json`, `Cargo.toml` and `memory.x`
into the generated project, but not `rgb.toml`.

## Flash

```powershell
espflash.exe write-bin 0x0 desk_hub.bin
# or
espflash.exe flash desk_hub.elf
```

## Change the lighting

Everything lives in `rgb.toml`; see the comments in the file and the RMK
documentation page `docs/configuration/rgb.md` in the fork. Nothing in the
firmware source needs editing, and a key RMK cannot honour fails the build
instead of doing nothing.

## Known state

- **The LED order in `rgb.toml` is a placeholder.** `[[rgb_matrix.layout]]`
  lists the LEDs in the order `keyboard.toml`'s `[layout].map` happens to write
  the keys, which is probably not the order they are soldered in. Effects that
  depend on position (gradients, pinwheels, beacons) will run in the wrong
  direction until the real order is filled in. To read it: open Vial's Lighting
  panel, select **Direct Control**, and paint one LED at a time.
- `[rgb_matrix].animations` turns on every effect the fork implements — all of
  QMK's RGB Matrix effects now, including the reactive/splash family, the typing
  heatmap and digital rain. The reactive ones answer key presses by default;
  `react_on_keyup = true` makes them answer releases instead.
- The chain is powered from 5 V; the firmware caps brightness at 128 of 255 and
  the default brightness is 64. Nineteen LEDs at full white would be about
  1.14 A.
