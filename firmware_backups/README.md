# Known-good firmware backups

Compiled firmware rescued from the old working clone (`~/projects/qmk_firmware`, deleted 2026-08-25).
These are force-added to git (`.gitignore` normally excludes `*.hex`/`*.uf2`).

| File | Built | Notes |
|---|---|---|
| `keebio_iris_ce_rev1_peteanning.uf2` | 2024-07-07 | **What is flashed to the Iris CE** — built from commit `f538af8a29` plus the RGB underglow edit later committed as `6530c55bb1`. Roll-back image. |
| `keebio_iris_ce_rev1_default.uf2` | 2024-07-07 | Iris CE default keymap |
| `keebio_iris_ce_rev1_via.uf2` | 2024-07-07 | Iris CE VIA keymap |
| `keebio_iris_rev2_peteanning.hex` | 2021-06-29 | Original Iris rev2, peteanning keymap |
| `keebio_iris_rev2_default.hex` | 2021-06-29 | Original Iris rev2, default keymap |

To flash a `.uf2`: put the Iris CE into bootloader mode (QK_BOOT or double-tap reset) and copy the file to the RPI-RP2 drive. To flash a `.hex` to the rev2: `qmk flash` or QMK Toolbox.
