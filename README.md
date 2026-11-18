# Reviung41 ZMK Config

ZMK firmware configuration for a Reviung41 keyboard with a nice!nano controller.
The keymap is built for a computer set to the **Turkish-Q** keyboard layout. It
uses these modules of its own, one feature per repo:

- [`zmk-autoswitch`](https://github.com/casab/zmk-autoswitch): lets the AutoSwitch
  KVM app change the keyboard's Bluetooth profile.
- [`zmk-lazy-layer-mod`](https://github.com/casab/zmk-lazy-layer-mod): the Lower
  thumb, a layer key that holds Cmd/Win only when held on its own.
- [`zmk-rgb-led-overlay`](https://github.com/casab/zmk-rgb-led-overlay): per-LED
  control on top of the underglow (the center LED is turned off; indicators
  light single LEDs).
- [`zmk-rgb-layer-indicator`](https://github.com/casab/zmk-rgb-layer-indicator):
  the purple gaming LEDs.
- [`zmk-rgb-ble-indicator`](https://github.com/casab/zmk-rgb-ble-indicator): shows
  the Bluetooth profile when it changes.
- [`zmk-rgb-battery-indicator`](https://github.com/casab/zmk-rgb-battery-indicator):
  shows the battery level on a key press.
- [`zmk-rgb-media-indicator`](https://github.com/casab/zmk-rgb-media-indicator):
  animates the LEDs for media and screen-brightness keys.

## Files

| File | What it holds |
| --- | --- |
| `config/reviung41.keymap` | Layers, key bindings, custom behaviors, Turkish symbol aliases. Each layer has a diagram comment above its bindings. |
| `config/reviung41.conf` | Feature settings: RGB underglow, Bluetooth transmit power, AutoSwitch. |
| `config/west.yml` | Which code is fetched: ZMK and the modules above, all from their `main` branches. |
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
| 1 | `_GAMING` | Util layer, space key. Turn off with the right outer thumb. | Plain Shift, Ctrl and Alt with no tap actions, so they react instantly. The two bottom-middle LEDs are purple while it's on. |
| 2 | `_LOWER` | Hold the left inner thumb. Held alone, it also holds Cmd (Win on Windows). | Cmd + left-hand keys, plus arrows and symbols on the right. |
| 3 | `_CMDSHIFT` | On Lower, hold the left key of the middle row | Cmd+Shift+key for every letter. |
| 4 | `_RAISE` | Hold the right inner thumb | Symbols, Home/End/PgUp/PgDn, Delete. |
| 5 | `_NUMPAD` | Hold the top-left key | Digits, F1–F12, `= - + * / . ,`. |
| 6 | `_UTIL` | Hold both inner thumbs | RGB, Bluetooth profiles 1–5 and clear, battery level, media, screen brightness, external power, reset, bootloader. |

### Base layer

```
| Esc/Npd |  Q  |  W  |  E  |  R  |  T  |   |  Y  | U/Ü |  I  |  O  |  P  | Bspc/RSft |
| CpsW/Sft|  A  |  S  |  D  |  F  | G/Ğ |   |  H  |  J  |  K  |  L  |  Ş  |     İ     |
| Tab/Ctl |  Z  |  X  |  C  |  V  |  B  |   |  N  |  M  |  Ö  |  Ç  |  .  |   Enter   |
                     | ⌘/Alt | ,/Lwr⌘ | Space | "/Rse | RAlt |
```

- `Esc/Npd` types Tab instead of Esc while a left modifier is held, so
  Cmd+Tab and Shift+Tab work from that key.
- `,/Lwr⌘` types `,` when tapped, and `;` while Right Shift is held. Held, it
  opens Lower after the usual 200 ms. It also acts as Cmd, but **only when held
  on its own** (the `zmk-lazy-layer-mod` module):
  - Use any key while holding it (a symbol, an arrow, `⌘C`...) and no extra Cmd
    is sent. Symbols and arrows type plainly; the left-hand `⌘` keys send their
    own Cmd. On Windows, nothing here opens the Start menu. A key counts from
    the moment it's pressed, so holding Raise for Util (which only turns on
    after Raise's own 200 ms) doesn't let Cmd in either.
  - Keep holding it alone for another 100 ms (0.3 s in total) and Cmd goes down,
    so you can Cmd+click with the mouse.
  - Ctrl, Shift or Alt held together with it (before Cmd goes down) also keep
    Cmd out: Ctrl + Lower is just Ctrl with the Lower layer, not Ctrl+Cmd. For
    Cmd combinations use ⌘Ctl, ⌘Alt and the ⇧⌘ layer on Lower.
  - Release it alone before that and it taps Cmd/Win, like a quick press of the
    Win key (on Windows that opens Start).
  - If Cmd is already down and you then type a symbol, the symbol drops Cmd; on
    Windows that can still open Start. Type symbols before the 0.3 s to avoid it.
  - From the gaming layer, Lower never holds Cmd/Win.
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
| `lwr_cmd` + `lazy_cmd` | Hold-tap, tap-preferred + lazy layer-mod (module) | 200 ms + 100 ms | The Lower thumb: tap `,`; hold Lower, plus Cmd when held alone |
| `nc_*` | Mod-morph | | Lower's right-half symbols and arrows, dropping Cmd if it's already down |
| `led_overlay` | Node (module) | | Per-LED layer between the underglow and the strip; keeps LED 10 (center) off |
| `rgb_layer_indicator` | Node (module) | | LEDs 4 and 5 purple while the gaming layer is on |
| `rgb_ble_indicator` | Node (module) | 2 s | Bluetooth profile change lights the active profile's area blue |
| `bat_ind` | Behavior (module) | 3 s | Util key: battery level as a left-to-right gauge |
| `rgb_media_indicator` | Node (module) | 350 ms | Media and brightness keys animate the LEDs |

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

- **RGB underglow** is yours to set with the RGB keys on the Util layer. It
  turns off after 30 seconds without a keypress, and any key brings it back.
  **EXT PWR** cuts power to the LEDs.
- **Single LEDs.** The 11 LEDs are numbered along the strip's wiring (from the
  Reviung41 v1.4 KiCad netlist), seen from above:

  ```
          1   0                                  9   8
       2                                                7
            3                10                   6
                       4               5
  ```

  - LED 10, in the center, is left out of your RGB effect. Only the play/pause
    pulse and passing animations light it briefly.
  - Indicators fade in and out instead of switching.
  - Gaming on: LEDs 4 and 5 are purple. Your own RGB stays on the rest.
  - Left to right, the 10 lit LEDs are 2, 1, 3, 0, 4, 5, 9, 6, 8, 7.
  - Bluetooth profile change (key or AutoSwitch): the keyboard is split left to
    right into one area per profile in use (paired profiles, plus the active one
    if it's empty), and the active profile's area lights blue for 2 s. Two
    profiles in use: halves. Three: thirds. Five: 2 LEDs each.
  - **BATTERY** key on the Util layer (under BT1): fills the LEDs left to right
    for 3 s, 100% = all 10, 50% = 5. Green from 50%, orange from 25%, red below.
  - Media keys (teal, about 0.35 s): next track sweeps left to right, previous
    right to left, volume up grows from the center, volume down shrinks into
    it, play/pause pulses the center LED. Screen brightness +/- animate like
    volume, in amber. Sweeps and rings leave the center LED out.
  - Indicators show even when your RGB is off: the LEDs get power just for
    them. While the keyboard is idle, the gaming LEDs are hidden, but the
    temporary ones (Bluetooth, battery, media) still show, so a KVM switch
    after a break still flashes the profile.
- **Deep sleep** after 15 minutes without a keypress. Any key wakes the keyboard,
  and that key isn't typed. It never sleeps while plugged into USB power. While
  asleep it isn't connected to any computer, so AutoSwitch can't move it to
  another profile until it wakes. Waking restarts the keyboard on the base
  layer.
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
