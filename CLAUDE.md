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
- Holding the base-layer Lower thumb also holds RGUI (Cmd/Win). Any Lower key
  that should type without Cmd must be an `nc_*` mod-morph that masks
  `MOD_RGUI`; a plain `&kp` there gets Cmd added.
- `lower_cmd` only presses and releases `&mo _LOWER` and `RGUI`. Don't add
  delays or extra key taps to it (the user rejected Start-menu "neutralizer"
  taps; a Win tap opening Start is fine). ZMK runs a macro synchronously only up
  to its first delay, and the keys the hold-tap held back are replayed right
  after, so anything slower lands on those keys. A Right Ctrl tap with
  `tap-ms = <10>` once turned fast `?` and Cmd+A into Ctrl+`?` and Ctrl+Cmd+A.
- The LEDs are the gaming indicator (`gaming_on` / `gaming_off`). Don't
  reintroduce RGB toggling there.
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
- `config/west.yml` - ZMK `main` plus `zmk-autoswitch` `main` (casab remote).
- `build.yaml` - build matrix: `nice_nano//zmk` + `reviung41`.
- `.github/workflows/build.yml` - ZMK's shared `build-user-config.yml@main`.

## AutoSwitch module

`zmk-autoswitch` (https://github.com/casab/zmk-autoswitch, same owner) adds a
vendor HID interface so the AutoSwitch KVM app can select Bluetooth profiles.
Changes to its HID interface mean the user must re-pair on every computer after
flashing; say so whenever a change affects it. Its options are documented in the
module's README.

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
