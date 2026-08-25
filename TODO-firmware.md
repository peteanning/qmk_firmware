# TODO: QMK housekeeping and upstream sync

Decision (2026-08-25): staying pinned at `f538af8a29` (Jul 2024) for now. Firmware works;
revisit the upstream sync when there is dedicated time.

## Do soon (protects existing work)

- [x] Rescue the uncommitted keymap change in the OLD clone — DONE 2026-08-25 (commit `6530c55bb1`,
      pushed to fork): iris_ce keymap RGB underglow controls now in git, matching flashed firmware.
- [x] Back up the known-good firmware binaries — DONE 2026-08-25: committed to `firmware_backups/`
      (force-added past .gitignore) and pushed to the fork. See `firmware_backups/README.md`.
- [x] Old clone at `/Users/peter/projects/qmk_firmware` — DELETED 2026-08-25 after verifying
      (checksummed binary copies; stashes inspected and superseded; no unpushed commits).

## The upstream sync (when there is time)

Fork is upstream QMK as of Jul 2024; upstream has had ~8 quarterly breaking-change cycles since.
Custom changes are additive (keymap files only) so git conflicts ~zero; the work is fixing
compile breaks (e.g. RGB_* underglow keycodes were renamed upstream).

Both keymaps verified compiling on the pinned tree (2026-08-25); rev2 needed its config.h
modernized (`a5df1446be`).

**Flash headroom warning (iris rev2):** the rev2's ATmega32u4 build is already at 91% flash
(26212/28672 bytes, 2460 free). QMK tends to grow between releases, so after the upstream merge
the rev2 may not fit. If it overflows: disable unused features in the keymap's rules.mk
(e.g. `CONSOLE_ENABLE = no`, `MOUSEKEY_ENABLE = no`, trim rgblight animations in config.h) or
add `LTO_ENABLE = yes`, which typically saves a few KB. The RP2040-based Iris CE has no such
constraint.

1. `git fetch upstream && git merge upstream/master`
2. `make git-submodule` (submodules move with breaking changes)
3. `qmk compile -kb keebio/iris_ce/rev1 -km peteanning` and fix deprecations
4. `qmk compile -kb keebio/iris/rev2 -km peteanning` and fix deprecations
5. Flash Iris CE (`qmk flash`), test all layers; keep the old .uf2 backup to roll back
6. Push merged master to fork
