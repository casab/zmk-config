# Reviung41 ZMK Config

ZMK firmware configuration for a Reviung41 keyboard with a nice!nano controller.
The keymap is built for a computer set to the **Turkish-Q** keyboard layout, and
it includes the [AutoSwitch](https://github.com/casab/zmk-autoswitch) module so
the AutoSwitch KVM app can change the keyboard's Bluetooth profile.

## Files

| File | What it holds |
| --- | --- |
| `config/reviung41.keymap` | Layers, key bindings, custom behaviors, Turkish symbol aliases. Each layer has a diagram comment above its bindings. |
| `config/reviung41.conf` | Feature settings: RGB underglow, Bluetooth transmit power, AutoSwitch. |
| `config/west.yml` | Which code is fetched: ZMK and `zmk-autoswitch`, both from their `main` branches. |
| `build.yaml` | What GitHub Actions builds: board `nice_nano//zmk`, shield `reviung41`. |
| `.github/workflows/build.yml` | Runs ZMK's shared build workflow on push, pull request, or manual run. |

## Build and flash

1. Push a change, or run the workflow by hand from the GitHub **Actions** tab.
2. Open the finished run and download the `firmware` artifact.
3. Double-tap the reset button (or use the **Bootloader** key on the Util layer)
   so the nice!nano shows up as a USB drive.
4. Copy the `.uf2` file onto that drive. The keyboard restarts by itself.

To check that the firmware compiles without pushing, build it locally with
Docker (the first run downloads ZMK and Zephyr and takes a few minutes):

```sh
docker run --rm -v zmk-config-ws:/ws -v "$PWD:/repo:ro" zmkfirmware/zmk-build-arm:stable bash -c '
  cd /ws && rm -rf config && cp -r /repo/config config &&
  { [ -d .west ] || west init -l config; } && west update && west zephyr-export &&
  west build -s zmk/app -d build -b nice_nano//zmk -p -- -DZMK_CONFIG=/ws/config -DSHIELD=reviung41'
```

## Layers

Notation: `tap/hold`. For example `Esc/Npd` types Esc when tapped and opens the
numpad layer while held. The thumb row is, left to right: outer, inner, space,
inner, outer.

| # | Layer | How to reach it | What's on it |
| --- | --- | --- | --- |
| 0 | `_QWERTY` | Default | Letters including Ğ (hold G) and Ü (hold U), Caps Word / Shift, Tab / Ctrl, Backspace / Right Shift. |
| 1 | `_GAMING` | Util layer, space key. Turn off with the right outer thumb. | Plain Shift, Ctrl and Alt with no tap actions, so they react instantly. Toggling it also toggles RGB, which acts as an indicator. |
| 2 | `_LOWER` | Hold the left inner thumb | `LG(...)` shortcuts on the left (Cmd on macOS, Win on Windows), arrows and symbols on the right. |
| 3 | `_CMDSHIFT` | On Lower, hold the left key of the middle row | Cmd+Shift+key for every letter. |
| 4 | `_RAISE` | Hold the right inner thumb | Symbols, Home/End/PgUp/PgDn, Delete. |
| 5 | `_NUMPAD` | Hold the top-left key | Digits, F1–F12, `= - + * / . ,`. |
| 6 | `_UTIL` | Hold both inner thumbs | RGB, Bluetooth profiles 1–5 and clear, media, screen brightness, external power, reset, bootloader. |

### Base layer

```
| Esc/Npd |  Q  |  W  |  E  |  R  |  T  |   |  Y  | U/Ü |  I  |  O  |  P  | Bspc/RSft |
| CpsW/Sft|  A  |  S  |  D  |  F  | G/Ğ |   |  H  |  J  |  K  |  L  |  Ş  |     İ     |
| Tab/Ctl |  Z  |  X  |  C  |  V  |  B  |   |  N  |  M  |  Ö  |  Ç  |  .  |   Enter   |
                     | ⌘/Alt | ,/Lwr | Space | "/Rse | RAlt |
```

- `Esc/Npd` types Tab instead of Esc while a left modifier is held, so
  Cmd+Tab and Shift+Tab work from that key.
- `,/Lwr` types `;` while Right Shift is held.
- `CpsW` is Caps Word: it capitalizes letters until you type a non-letter.
  `_` keeps it going, so `MY_CONSTANT` works. (On Turkish-Q, `_` is the `PLUS`
  keycode, which is why the keymap's continue list names `PLUS`.)

Diagrams for the other layers are in `config/reviung41.keymap`.

## Turkish-Q notes

ZMK sends key positions, and the computer turns them into characters with its
own layout. So with the computer set to Turkish-Q, US-named keycodes type
Turkish characters:

| Keycode | Types |
| --- | --- |
| `SEMI` | ş Ş |
| `SQT` | i İ |
| `COMMA` | ö Ö |
| `DOT` | ç Ç |
| `FSLH` | . : |
| `BSLH` | , ; |
| `LEFT_BRACKET` / `RIGHT_BRACKET` | ğ Ğ / ü Ü |
| `GRAVE` | " é |
| `NON_US_BSLH` | < > \| |

Shifted keycodes follow the same rule: `CARET` (Shift+6) types `&`, `AMPS`
(Shift+7) types `/`, and so on. Braces, brackets, `@`, `#`, `$`, `\`, `|`, `` ` ``
and `~` are typed with Right Alt (AltGr) aliases defined at the top of the keymap.

The keymap is used on both Windows and macOS with the Turkish-Q layout, and these
keycodes type the same characters on both (`< > |` confirmed on each).

## Custom behaviors

| Behavior | Kind | Timing | Used for |
| --- | --- | --- | --- |
| `h_alt` | Hold-tap, tap-preferred | 130 ms | Ğ and Ü on hold, ⌘/Alt thumb, İ/Tab on the gaming layer |
| `hom` | Hold-tap, tap-unless-interrupted | 300 ms | Backspace / Right Shift. Becomes Shift only if another key is pressed while it's held. |
| `fmt` | Hold-tap, hold-preferred | 100 ms | Tab / Ctrl |
| `mt_cw` | Hold-tap, hold-preferred | 100 ms | Caps Word / Shift |
| `prog_caps` | Caps Word | | Caps Word that continues through `_` (`PLUS` keycode) |
| `mmnet` | Mod-morph | | Esc/Npd that becomes Tab with a left modifier |
| `mmlic` | Mod-morph | | `,`/Lower that becomes `;` with Right Shift |
| `tog_gaming_rgb` | Macro | | Toggles the gaming layer and RGB together |

## AutoSwitch

[`zmk-autoswitch`](https://github.com/casab/zmk-autoswitch) lets the AutoSwitch
KVM app move the keyboard to another computer's Bluetooth profile when the KVM
switches. It's turned on by `CONFIG_ZMK_AUTOSWITCH=y` in `reviung41.conf`.

- Pair each computer to its own profile (Util layer, BT1–BT5).
- After flashing a firmware that first adds the module, or changes its HID
  interface, remove the keyboard from each computer's Bluetooth settings and pair
  it again. Computers cache the keyboard's old list of services.
- In the AutoSwitch app, open **Devices** and add the keyboard. It should say
  "Switches channel on request".

## Power and connection settings

All in `config/reviung41.conf`.

- **RGB underglow** is on and turns itself off after 30 seconds without a
  keypress. Any key turns it back on. Toggle it on the Util layer, or cut power
  to the LEDs with **EXT PWR**.
- **Deep sleep** after 15 minutes without a keypress. Any key wakes the keyboard,
  and that key isn't typed. It never sleeps while plugged into USB power. While
  asleep it isn't connected to any computer, so AutoSwitch can't move it to
  another profile until it wakes.
- **Bluetooth transmit power** is raised to +8 dBm for range, which uses more
  battery.
- **Experimental Bluetooth features** (`CONFIG_ZMK_BLE_EXPERIMENTAL_FEATURES`):
  - The 2M radio mode is off, which steadies connections on some Windows and Mac
    Bluetooth chips.
  - Pairing uses a passkey. The computer shows 6 digits; type them on the
    keyboard (number row or the numpad layer) and press Enter. Esc cancels.
  - A computer that forgot the keyboard can pair again to its own profile
    without pressing BT CLR first.

### USB and Bluetooth

ZMK prefers USB, but only once a computer has set the keyboard up as a USB
keyboard. On a charger or a power-only cable it keeps typing over Bluetooth.
Bluetooth stays connected in the background either way. There's no setting to
turn it off while on USB.

## To do

- Check that every AltGr symbol (braces, brackets, `@`, `#`, `$`, `\`, `` ` ``,
  `~`) types correctly on both Windows and macOS.
- Tune the hold-tap timings after real use. The 100 ms mod-taps are fast.
- Add `display-name` to the layers if ZMK Studio or a display gets used.
