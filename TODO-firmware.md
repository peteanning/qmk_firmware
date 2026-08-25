# TODO: QMK housekeeping and upstream sync

Decision (2026-08-25): staying pinned at `f538af8a29` (Jul 2024) for now. Firmware works;
revisit the upstream sync when there is dedicated time.

## Do soon (protects existing work)

- [ ] Rescue the uncommitted keymap change in the OLD clone
      (`/Users/peter/projects/qmk_firmware`): `keyboards/keebio/iris_ce/keymaps/peteanning/keymap.c`
      has RGB underglow controls (RGB_TOG/RGB_MOD/HUI/SAI/VAI/HUD/SAD/VAD) added to a layer.
      This is what is flashed to the Iris CE but is NOT in git. Copy into this repo, commit, push to fork.
- [ ] Back up the known-good firmware binaries somewhere safe (they only exist in the old clone):
      - `/Users/peter/projects/qmk_firmware/keebio_iris_ce_rev1_peteanning.uf2` (2024-07-07, matches flashed firmware incl. RGB tweaks)
      - `/Users/peter/projects/qmk_firmware/keebio_iris_rev2_peteanning.hex` (2021-06-29)
      - plus `default`/`via` builds for both boards in the same directory
- [ ] Decide fate of the old clone at `/Users/peter/projects/qmk_firmware` — once the keymap
      diff and binaries are rescued, it is redundant with this repo (`~/qmk_firmware`).

## The upstream sync (when there is time)

Fork is upstream QMK as of Jul 2024; upstream has had ~8 quarterly breaking-change cycles since.
Custom changes are additive (keymap files only) so git conflicts ~zero; the work is fixing
compile breaks (e.g. RGB_* underglow keycodes were renamed upstream).

1. `git fetch upstream && git merge upstream/master`
2. `make git-submodule` (submodules move with breaking changes)
3. `qmk compile -kb keebio/iris_ce/rev1 -km peteanning` and fix deprecations
4. `qmk compile -kb keebio/iris/rev2 -km peteanning` and fix deprecations
5. Flash Iris CE (`qmk flash`), test all layers; keep the old .uf2 backup to roll back
6. Push merged master to fork
