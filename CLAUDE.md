# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Building firmware

```bash
qmk compile -kb <keyboard> -km <keymap>    # e.g. qmk compile -kb planck/rev6 -km default
make <keyboard>:<keymap>                   # equivalent make form, e.g. make planck/rev6:default
qmk flash -kb <keyboard> -km <keymap>      # compile + flash to a connected board
qmk clean                                  # remove .build/ artifacts
```

`qmk compile` is directory-aware (fills in keyboard/keymap when run from a keyboard or keymap directory). Add `-j 0` for parallel compilation. The CLI is a wrapper that shells out to `make`, so both forms run the same build.

### Tests

C unit tests (googletest, host-native build):

```bash
make test:all                    # run all tests
make test:<name>                 # substring match, e.g. make test:caps_word
make test:caps_word_auto_shift   # test name = path under tests/ with / -> _
make list-tests                  # list all test names
```

Python CLI tests and lint (runs nose2 then flake8, despite the name):

```bash
qmk pytest
qmk pytest -t qmk.tests.test_cli_commands.test_c2json   # single test
```

### Lint / format

```bash
qmk lint -kb <keyboard> [--strict]   # lint a keyboard's config/layout
qmk format-c [files...]              # clang-format C files (-a for all core files)
qmk format-python                    # yapf on lib/python
qmk format-json <file>               # format info.json/keyboard.json
```

## Architecture

QMK builds keyboard firmware for AVR and ARM microcontrollers. Layers from bottom to top:

- `lib/` — vendored third-party submodules (ChibiOS, LUFA, V-USB, pico-sdk, googletest, printf), plus `lib/python/` which is the QMK CLI itself (not a submodule).
- `platforms/` — hardware abstraction per platform: `avr/`, `chibios/` (ARM), `arm_atsam/`, and `test/` (host build used by unit tests). GPIO, timers, EEPROM, bootloaders.
- `tmk_core/protocol/` — USB/HID protocol stacks (LUFA, ChibiOS, V-USB) and USB descriptors.
- `drivers/` — peripheral drivers (OLED, RGB/LED, sensors, encoders, haptics, etc.), platform-agnostic where possible.
- `quantum/` — the QMK core: matrix scan loop (`keyboard.c`), the action/tapping/layer engine (`action*.c`), keycode processing (`process_keycode/`), and feature implementations (`rgb_matrix/`, `audio/`, `debounce/`, `split_common/`, ...).
- `keyboards/` — per-board definitions, nested up to 5 directory levels (e.g. `keyboards/planck/rev6`).
- `layouts/` — community keymaps shared across boards with the same physical layout.
- `users/` — per-username shared keymap code ("userspace").

### Build system

The top-level `Makefile` is a command-line parser: it splits targets like `planck/rev6:default:flash` on colons, resolves keyboard/keymap globs, then re-invokes make with `builddefs/build_keyboard.mk`, which does the real build. `builddefs/common_features.mk` and `generic_features.mk` translate `<FEATURE>_ENABLE = yes` from rules.mk into source files and `-D` defines. Output goes to `.build/`.

### Data-driven keyboard configuration

Keyboard properties live in JSON, not C, wherever possible. A leaf directory containing `keyboard.json` is a buildable target; parent-level `info.json` files contribute shared metadata. At build time the CLI merges all `info.json`/`keyboard.json` up the tree plus values extracted from legacy `rules.mk`/`config.h`, validates against `data/schemas/keyboard.jsonschema`, and generates headers (`info_config.h`, `default_keyboard.c`, etc.) into `.build/<target>/src/`. The mappings between JSON keys and `#define`s/make variables are in `data/mappings/info_config.hjson` and `info_rules.hjson`. Prefer adding config to `keyboard.json` over `config.h`/`rules.mk` for keyboard-level settings.

Config override order (later wins): keyboard `config.h` from deepest revision dir up to the top keyboard dir, then userspace, then keymap `config.h`. `post_config.h` files are included after all `config.h` files so they can react to final values. Keymap `rules.mk` overrides keyboard `rules.mk`.

### Tests layout

`tests/<name>/` directories are full-firmware integration tests: a `test.mk` (marker, may set feature flags), `config.h`, and `test_*.cpp` files, linked against the mock harness in `tests/test_common/` (fake matrix, mock host driver). Feature-level unit tests live in `quantum/<feature>/tests/` (e.g. debounce, encoder) and compile only the files under test. Both are discovered by `builddefs/testlist.mk`; test names may not contain `-`.

### QMK CLI

`lib/python/qmk/cli/` holds one module per subcommand (MILC framework, registered in `cli/__init__.py`). Code generators are under `cli/generate/`; the data-driven merge logic is in `lib/python/qmk/info.py`. Python tests are in `lib/python/qmk/tests/`.

## Conventions

- C style: 4-space indent, One True Brace Style, always include optional braces, `#pragma once` in headers. `.clang-format` at the repo root enforces most of this; wrap layout macros in `// clang-format off` / `// clang-format on` since clang-format mangles them.
- PR targeting: normal bugfixes and new keyboards target `master`; core changes, keyboard moves/renames, and keyboard refactors (e.g. data-driven conversions) must target `develop` (merged to `master` on a ~3-month breaking-changes cycle).
- Use QMK's abstractions rather than raw platform code in keyboard-level code: e.g. the timer API (`timer_read()`) and GPIO API instead of AVR/ChibiOS-specific calls.
