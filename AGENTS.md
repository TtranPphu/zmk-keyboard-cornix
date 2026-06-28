# AGENTS.md — zmk-keyboard-cornix

## What this is

ZMK firmware module for the **Cornix** split keyboard (nRF52840 / Ebyte E73). Defines 3 boards and 3 shields.

## Boards

| Board | Role |
|---|---|
| `cornix_left` | Left half (central, standalone mode) |
| `cornix_right` | Right half (peripheral, no USB) |
| `cornix_ph_left` | Left half for dongle mode |

## Shields

| Shield | Purpose |
|---|---|
| `cornix_dongle_adapter` | Matrix + BLE for dongle setups |
| `cornix_dongle_eyelash` | Display overlay for dongle |
| `cornix_indicator` | RGB battery/status LEDs (power hungry) |

## Build system

This is a **ZMK module** (`zephyr/module.yml` sets `board_root: .`, `snippet_root: .`). Use `just` as the entry point.

### First time

```
just init
```

Runs `west init -l config && west update --fetch-opt=--filter=blob:none && west zephyr-export`.

### Build

```
just build <expr>
```

`<expr>` is matched against artifact names from `build.yaml` (case-insensitive, regex). Common targets:

| Artifact | Board | Shield | Snippet |
|---|---|---|---|
| `cornix_dongle_nosd` | `nice_nano/nrf52840/zmk` | `cornix_dongle_adapter cornix_dongle_eyelash dongle_display` | `studio-rpc-usb-uart nrf52840-nosd` |
| `cornix_left_default_nosd` | `cornix_left` | — | — |
| `cornix_left_for_dongle_nosd` | `cornix_ph_left` | — | — |
| `cornix_right_nosd` | `cornix_right` | — | — |
| `cornix_reset` | `cornix_right` | `settings_reset` | `studio-rpc-usb-uart nrf52840-nosd` |
| `reset_nicenano_nosd` | `nice_nano/nrf52840/zmk` | `settings_reset` | `studio-rpc-usb-uart nrf52840-nosd` |

Use `just list` to see all available targets.

### Single build (low-level)

```
just _build_single <board> <shield> <snippet> <artifact>
```

### Other commands

| Command | What it does |
|---|---|
| `just draw <keyboard>` | Generate SVG keymap (uses keymap-drawer, needs keymap name like `cornix`) |
| `just test <testpath>` | Run native_posix_64 test with snapshot diff |
| `just clean` | Remove `.build/` and `firmware/` |
| `just clean-all` | Also remove `.west/` and `zmk/` |
| `just update` | `west update` + reconfigure zephyr.base |

## Key quirks

- **All builds use `nrf52840-nosd` snippet** — no SoftDevice flash layout. This is required; the original RMK firmware removed it.
- **Never set `ZEPHYR_BASE`** when using west. Use `Zephyr_DIR` pointing to `$ZEPHYR_BASE/share/zephyr-package/cmake/` instead.
- `ZMK_EXTRA_MODULES` must point to the repo root when building outside the module system.
- Dongle screen not yet working with Zephyr 4.1 (commented out in `build.yaml`).
- RGB not yet supported (TODO, SPI config disabled).
- Two keymap variants: `config/cornix.keymap` (54-key) and `config/cornix42.keymap` (42-key). The board-level `cornix.keymap` under `boards/jzf/cornix/` is a fallback for external users.
- Uses `zmk-helpers` (urob) for homerow mods (`config/cornix.keymap` includes it).
- Uses `zmk-dongle-display` (englmaxi) via west.yml.

## Dev environment (Nix)

```
nix develop
```

Provides: gcc-arm-embedded, cmake, dtc, ninja, just, yq (python-yq), tio, keymap_drawer, Zephyr SDK 0.16 (arm-zephyr-eabi).

## Flashing

Output is `.uf2` files in `firmware/` (or `.bin` for non-UF2 boards). Flash by copying to the board's mounted UF2 drive. Use `reset.uf2` (any `settings_reset` target) to clear bonds/layers before re-pairing.

## Testing

Snapshot-based tests target `native_posix_64`:

```
just test tests/<case>
```

Expects `keycode_events.snapshot` and `events.patterns` in the test config dir. Use `--auto-accept` to update snapshots.

## GitHub Actions

- **Push to `boards/` or `config/`** → builds via `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`.
- **Tag `v*.*`** → builds + creates a GitHub release with a zip of all `.uf2` files.
