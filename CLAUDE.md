# CLAUDE.md

ZMK user config for a Reviung41 (nice!nano, board `nice_nano//zmk`) with a
keymap for a Turkish-Q host layout and the `zmk-autoswitch` module.
`README.md` is the user-facing description; keep it in step with the code.

## Rules

- Don't change firmware behavior (bindings, behaviors, `.conf` flags, `west.yml`
  revisions, `build.yaml`) unless the user asks for that change.
- US-named keycodes are physical positions. `SEMI`, `SQT`, `COMMA`, `DOT`,
  `FSLH`, `BSLH`, `GRAVE`, `NON_US_BSLH`, `LEFT_BRACKET`, `RIGHT_BRACKET` and
  shifted ones like `CARET`, `AMPS`, `PLUS` type Turkish-Q characters on purpose
  (see the table in the keymap header and in `README.md`). Never "fix" them to
  what the US name suggests without the user testing on a Turkish-Q host.
- The `RA(...)` aliases at the top of the keymap are Turkish-Q AltGr symbols.
  Leave them unless the user reports wrong output.
- The keyboard is used on both Windows and macOS, both set to Turkish-Q.
  `GRAVE` = `" é` and `NON_US_BSLH` = `< > |` on both (confirmed by the user).
- The `LG(...)` layers are macOS-oriented (Cmd; Win key on Windows). If adding
  Windows/Linux variants, keep the macOS ones and document the host assumption.
- The base-layer Lower thumb is `lwr_cmd` (hold-tap) whose hold is `lazy_cmd`
  from `zmk-lazy-layer-mod`: the user's rule is that Cmd/Win is held **only when
  Lower is held alone**, never added to a key used during the hold. Lower keys
  that need Cmd send it themselves (`LG(...)`). Other modifiers count as
  "combined" too (`cancel-on-modifiers`): Ctrl/Shift/Alt + Lower must never add
  Cmd. Any other key counts from its press, not from what it later sends, so
  Lower + held Raise (a layer-tap, Util after its tapping term) adds no Cmd. Right-half symbols/arrows are
  `nc_*` mod-morphs masking `MOD_RGUI`, which only matters if Cmd already went
  down after `delay-ms`.
- Don't send extra keys around Cmd/Win (the user rejected Start-menu
  "neutralizer" taps: Ctrl and F24). A Win tap opening Start is fine. Lessons:
  ZMK macros run synchronously only up to their first delay, and a hold-tap
  replays held-back keys right after its hold binding, so a macro hold binding
  with taps once turned fast `?` and Cmd+A into Ctrl+`?` and Ctrl+Cmd+A; and
  Windows opens Start the moment a lone Win is released, even when the release
  shares a report with the next key.
- The gaming LED comes from `zmk-rgb-layer-indicator` (`rgb_layer_indicator`
  node). Don't set RGB from keymap macros: the user's own RGB settings must
  survive gaming.
- Only toggled (sticky) layers get an indicator color, like `_GAMING`. Held
  layers (Lower, Raise, Numpad, Util) don't: holding the key already shows
  they're on, and a color would just flash on every hold. Util must never get
  one anyway, because the RGB controls live there.
- Keep the layer `#define`s in `config/reviung41.keymap` in the same order as
  the layer nodes.
- Every layer has a diagram comment above its `bindings`. Update the diagram in
  the same edit as the bindings. Notation is `tap/hold`; `⌘` means `LG(...)`.
- Make small, reviewable edits. Don't reformat the binding grids.
- Update `README.md` when layers, behaviors, build targets, modules, or Turkish
  assumptions change.
- Never flash firmware. Build, then leave flashing to the user.

## Files

- `config/reviung41.keymap` - aliases, behaviors, layers, bindings.
- `config/reviung41.conf` - RGB underglow (auto-off when idle), deep sleep,
  BT TX power +8 dBm, experimental BLE features (passkey pairing),
  `CONFIG_ZMK_AUTOSWITCH=y`.
- Changes that alter HID descriptors or pairing (NKRO, HID indicators, USB boot,
  passkey/security options, AutoSwitch HID) force the user to re-pair every
  computer. Say so before making them.
- `config/west.yml` - ZMK `main` plus the own modules below and
  `zmk-autoswitch`, all `main` (casab remote).
- `build.yaml` - build matrix: `nice_nano//zmk` + `reviung41`.
- `.github/workflows/build.yml` - ZMK's shared `build-user-config.yml@main`.

## AutoSwitch module

`zmk-autoswitch` (https://github.com/casab/zmk-autoswitch, same owner) adds a
vendor HID interface so the AutoSwitch KVM app can select Bluetooth profiles.
Changes to its HID interface mean the user must re-pair on every computer after
flashing; say so whenever a change affects it. Its options are documented in the
module's README.

## Own modules

Each feature lives in its own repo under `github.com/casab`, local clones next
to this one in `codejunk/` (the user wants one repo per module, like
`zmk-autoswitch`, never several features combined). They must stay public:
GitHub Actions fetches them through `west.yml` without credentials.

- `zmk-lazy-layer-mod`: `zmk,behavior-lazy-layer-mod` (layer + modifier only
  when held alone; `keycode`, `delay-ms`, `tap-on-release`,
  `cancel-on-modifiers`).
- `zmk-rgb-led-overlay`: `zmk,led-strip-overlay` node that `zmk,underglow`
  points at; sits between ZMK's underglow and the real strip. `disabled-leds`
  (out of the underglow effect only; indicators may still light them),
  `led-x` (positions in mm, for animations), `identify`. C API for indicator
  modules: `set/set_delayed/clear/commit` (static LEDs, fade in/out, diffed so
  repaints don't flicker) and `animate` (sweep/ring/pulse); slots are blended
  in order, higher on top. Powers the LEDs for indicators when the RGB is off.
  While idle it hides persistent slots (gaming) but shows slots marked
  `set_show_when_idle` (BLE, battery) and animations: a KVM switch changes the
  profile without a keypress, while the keyboard is idle.
- `zmk-rgb-layer-indicator`: `zmk,rgb-layer-indicator` node, child per layer
  (`layer`, `color-hsb`, `effect`, optional `leds`). With `leds` it paints those
  LEDs through the overlay (slot 0); without, it takes over the whole underglow
  and restores the user's RGB (also at boot).
- `zmk-rgb-ble-indicator`: `zmk,rgb-ble-indicator` node; `leds` (left to right)
  split into one area per profile in use (paired + active), active area lit
  (slot 3).
- `zmk-rgb-battery-indicator`: `zmk,behavior-rgb-battery-indicator` behavior
  (`&bat_ind` on Util); left-to-right gauge on `leds`, green >= 50%, orange >=
  25%, red below (slot 2).
- `zmk-rgb-media-indicator`: `zmk,rgb-media-indicator` node; watches consumer
  keycodes (no keymap bindings): next/prev sweep, vol/brightness rings,
  play/pause pulses `pulse-leds`, sweeps/rings leave out `skip-leds` (the
  center LED, at the user's request) (slot 4). Teal media, amber brightness. Media
  state (playing, volume level) is unknowable to the keyboard, so animations
  show the key pressed, never a state.

LED indexes follow the strip wiring, not position. The map (from the Reviung41
v1.4 KiCad netlist, gtips/reviung) is drawn in the keymap above `led_overlay`:
left half 0-4, right half 5-9, 10 is the center (disabled, the user dislikes
it).

To build this config against local, unpushed module changes, mount the module
folders into the Docker build and pass `-DZMK_EXTRA_MODULES="<dir1>;<dir2>"`.

## Validation

- Docs or comment-only changes: run `git diff --check`. No build needed.
- Keymap or config changes: build before committing. Either push and check
  `gh run list` / `gh run watch`, or build locally with Docker:

  ```sh
  docker run --rm -v zmk-config-ws:/ws -v "$PWD:/repo:ro" zmkfirmware/zmk-build-arm:stable bash -c '
    cd /ws && rm -rf config && cp -r /repo/config config &&
    { [ -d .west ] || west init -l config; } && west update && west zephyr-export &&
    west build -s zmk/app -d build -b nice_nano//zmk -p -- -DZMK_CONFIG=/ws/config -DSHIELD=reviung41'
  ```

  On Windows Git Bash, prefix with `MSYS_NO_PATHCONV=1` and use a `C:/...` path
  for the repo mount.
