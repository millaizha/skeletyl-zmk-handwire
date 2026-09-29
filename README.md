# Skeletyl 36

Handwired 36-key [Skeletyl](https://github.com/Bastardkb/Skeletyl) running ZMK.
Two nice!nano-compatible nRF52840 controllers, split over Bluetooth, no TRRS cable.
Each half has its own battery. The left half is the central and is the half a
computer pairs with. The Bluetooth name is **Mill Skeletyl**.

The keymap is a Colemak-DH Cantor layout: home-row mods, thumb layer-taps, and
Cmd+C / Cmd+X / Cmd+V / Cmd+Z on double tap. It is aimed at macOS. Firmware
tracks ZMK `main` on Zephyr 4.1.

```
Q  W  F  P  B          J  L  U  Y  '
A  R  S  T  G          M  N  E  I  O
Z  X  C  D  V          K  H  ,  .  /
      Esc Bsp Tab    Ent Spc Del
```

---

## Thumbs

Tap the key. Hold it for the layer.

| Thumb | Tap | Hold |
|---|---|---|
| Left outer | Esc | 1 · Media |
| Left middle | Backspace | 2 · Numbers |
| Left inner | Tab | 3 · Function |
| Right inner | Enter | |
| Right middle | Space | 4 · Symbols |
| Right outer | Delete | 5 · Mouse |

A thumb that is held and then released without another key still types its tap.
Tap, then hold, and the tap repeats, so Backspace and Space repeat as usual.

---

## Layer 0 · Colemak-DH

```
Q*   W    F    P    B          J    L    U    Y    '
A⌘   R⌥   S⌃   T⇧   G          M    N⇧   E⌃   I⌥   O⌘
Z*   X*   C    D    V*         K    H    ,    .    /
          Esc  Bsp  Tab      Ent  Spc  Del
          ·1   ·2   ·3            ·4   ·5
```

`*` is a double tap. `·n` is the layer you get by holding that thumb.

| Key | Tap | Again, quickly |
|---|---|---|
| Q | Q | Cmd+C |
| Z | Z | Cmd+Z |
| X | X | Cmd+X |
| V | V | Cmd+V |

The second tap has to land within 350 ms. A lone tap of Q, Z, X, or V also waits
that long before the letter is sent, because the keyboard is watching for the
second tap.

Home-row holds:

| Left | Hold | Right | Hold |
|---|---|---|---|
| A | Cmd | N | Shift |
| R | Option | E | Control |
| S | Control | I | Option |
| T | Shift | O | Cmd |

Same-hand rolls stay letters. An opposite-hand key is what turns the hold into a
modifier, so holding A and tapping J is Cmd+J, while rolling A into S types `as`.
The tapping term is 350 ms. That window matters when you want a modifier and the
next key is on the **same** hand. Opposite-hand chords do not wait it out.

Combos, base layer only. Both keys within 50 ms:

| Keys | Result |
|---|---|
| G and M | Caps word. The next word is uppercase, then it stops. Space, punctuation, or pressing it again ends it. |
| B and J | Repeat the last key. Hold both to keep it going. |
| `,` and `.` | `:` |

---

## Layer 1 · Media

Hold the outer left thumb (Esc).

```
·    ·    Bri- Bri+ ·          ·    ·     ·     ·   ·
·    ◀◀   ▶❚   ▶▶   ·          ·    CtlSs CtlRec Ss Rec
·    ·    Vol- Vol+ Mute       ·    ·     ·     ·   ·
          ·    ·    ·        ·    ·     ·
```

`◀◀` previous, `▶❚` play/pause, `▶▶` next. `Bri` is display brightness.
The capture keys sit on N, E, I, and O, the same row as the transport keys.
The Control versions are on the index and middle, since those are the ones used most.

| Key | Chord | What it does |
|---|---|---|
| CtlSs, on N | Cmd+Ctrl+Shift+4 | Selection copied to the clipboard |
| CtlRec, on E | Cmd+Ctrl+Shift+5 | Recording toolbar, result copied to the clipboard |
| Ss, on I | Cmd+Shift+4 | Selection saved as a file |
| Rec, on O | Cmd+Shift+5 | Recording toolbar, result saved as a file |

Cmd+Shift+5 opens the macOS toolbar. Choose record-screen or record-selection there, then start it. Cmd+Ctrl+Esc stops a recording.

The held thumb is transparent, so a tap still sends Esc.

---

## Layer 2 · Numbers

Hold the middle left thumb (Backspace).

```
1    2    3    4    5          6    7    8    9    0
Cmd  Opt  Ctl  Sft  `          ;    Sft  Ctl  Opt  Cmd
\    [    {    -    =          +    _    }    ]    |
          Bsp  ·    Ent      Spc  ,    .
```

Modifiers on this layer are real modifier keys, not home-row holds. The right-hand
ones are the right-side Shift, Control, Option, and Command.

---

## Layer 3 · Function

Hold the inner left thumb (Tab).

```
F1   F2   F3   F4   F5         F6   F7   F8   F9   F10
Cmd  Opt  Ctl  Sft  F11        F12  Sft  Ctl  Opt  Cmd
Bt0  Bt1  Bt2  Bt3  Boot       Ins  Home PgDn PgUp End
          .    Word ·        Boot Bt4  Clr
```

| Key | What it does |
|---|---|
| Word | Caps word, same as G+M on the base layer. Not Caps Lock. |
| Bt0–Bt4 | Select Bluetooth profile 0–4. Safe. Switches host, does not erase anything. |
| Clr | Forget the bond for the profile that is selected right now. Outer right thumb, so it takes Tab plus that thumb. |
| Boot, left bottom row | Reboot the **left** controller into the UF2 bootloader. |
| Boot, inner right thumb | Reboot the **right** controller into the UF2 bootloader. |

Boot and Clr are only on this layer. Clearing a bond does not clear the other profiles.

---

## Layer 4 · Symbols

Hold the middle right thumb (Space).

```
!    @    #    $    %          ^    &    *    (    )
Cmd  Opt  Ctl  Sft  ~          :    ←    ↑    ↓    →
·    ·    ·    ·    ·          Rpt  ·    <    >    ?
          ·    ·    ·        ·    ·    ·
```

`Rpt` is on K. It repeats the last key, so an arrow can be tapped and then
repeated while Space stays held. B+J on the base layer does the same thing.

---

## Layer 5 · Mouse

Hold the inner right thumb (Delete).

```
·    ·    B4   B5   ·          Wh↑ B1   B2   Lock ·
·    B3   B2   B1   ·          Wh↓  ←    ↓    ↑    →
·    ·    ·    ·    ·          B3   Wh←  Wh→  ·    ·
          Opt  Ctl  Sft      ·    ·    ·
```

B1 is left click, B2 right click, B3 middle click. B4 and B5 are the side buttons.
On the home row the buttons sit middle, right, left, which is the original order.
L and U, the keys above left and down, are another left click and right click.
K, beneath them, is another middle click.

`Lock` is on Y. Tap it while Delete is held and the mouse layer stays on after
you let go. Tap Y again to leave.

Three QMK mouse-speed keys and mouse buttons 6, 7, and 8 are empty. ZMK accelerates
inside the move behavior itself, and it only defines buttons 1–5.

---

## Timing

| | |
|---|---|
| Tapping term | 350 ms, for home-row mods, thumb layers, and the double taps |
| Quick tap | 175 ms on home-row mods, 200 ms on thumbs |
| Typing cutoff | A home-row key typed within 150 ms of the previous key is always a letter |

Change these in `config/skeletyl_handwire.keymap`, then rebuild both halves.
The left half is the one that applies the keymap.

---

## Wiring

Both halves use the same 4×5 matrix. Row 3 is the thumb row.

### Rows, orange

Sampled inputs with a pull-down.

| Row | nRF52840 | nice!nano |
|---|---|---|
| R0 | P0.17 | D2 |
| R1 | P0.20 | D3 |
| R2 | P0.22 | D4 |
| R3, thumbs | P0.24 | D5 |

### Columns, black

Driven outputs.

| Column | nRF52840 | nice!nano |
|---|---|---|
| C0 | P0.08 | D0 / RX |
| C1 | P1.00 | D6 |
| C2 | P0.11 | D7 |
| C3 | P1.04 | D8 |
| C4 | P1.06 | D9 |

Diodes run column → switch → diode → row, with the black band toward the row.
That is `diode-direction = "col2row"`.

### How the two halves meet

Each half scans its own columns 0–4. The right firmware adds `col-offset = <5>`,
so the two local matrices become one 4×10 keyboard.

The right half's **letter** columns are mirrored: C0 is the pinky side, C4 is
toward the center gap. The right **thumb** cluster follows that same direction.
Both halves put the three thumbs on local C2, C3, C4. On the right, C4 is the
inner thumb, C3 the middle, and C2 the outer.

```
RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4)   RC(0,9) RC(0,8) RC(0,7) RC(0,6) RC(0,5)
RC(1,0) RC(1,1) RC(1,2) RC(1,3) RC(1,4)   RC(1,9) RC(1,8) RC(1,7) RC(1,6) RC(1,5)
RC(2,0) RC(2,1) RC(2,2) RC(2,3) RC(2,4)   RC(2,9) RC(2,8) RC(2,7) RC(2,6) RC(2,5)
                        RC(3,2) RC(3,3) RC(3,4)   RC(3,9) RC(3,8) RC(3,7)
```

`RC(3,0)`, `RC(3,1)`, `RC(3,5)`, and `RC(3,6)` have no switch. 15 letters and 3
thumbs per half, 36 keys total.

---

## Build

Pushing to GitHub runs `.github/workflows/build.yml`. Download the `firmware`
artifact from the Actions run:

```
skeletyl_handwire_left.uf2
skeletyl_handwire_right.uf2
settings_reset.uf2
```

`config/west.yml` tracks ZMK `main` because the `nice_nano//zmk` board target
does not exist in the v0.3 release. To pin v0.3 instead, set `revision: v0.3`
in `config/west.yml`, change `@main` to `@v0.3` in the workflow, and change
every `board:` in `build.yaml` to `nice_nano_v2`.

A local build uses the same image as CI:

```bash
docker volume create zmkws
docker run --rm -v zmkws:/ws -v "$PWD":/hostrepo:ro \
  zmkfirmware/zmk-build-arm:stable bash -c '
    mkdir -p /ws/config && cp -R /hostrepo/config/. /ws/config/
    cd /ws && west init -l /ws/config && west update && west zephyr-export'

docker run --rm -v zmkws:/ws -v "$PWD":/hostrepo:ro \
  zmkfirmware/zmk-build-arm:stable bash -c '
    rm -rf /ws/repo /ws/config && mkdir -p /ws/repo /ws/config
    cp -R /hostrepo/. /ws/repo/ && cp -R /ws/repo/config/. /ws/config/
    cd /ws && west zephyr-export
    for s in skeletyl_handwire_left skeletyl_handwire_right settings_reset; do
      west build -s /ws/zmk/app -d /ws/build/$s -b "nice_nano//zmk" -- \
        -DZMK_CONFIG=/ws/config -DZMK_EXTRA_MODULES=/ws/repo -DSHIELD=$s
      cp /ws/build/$s/zephyr/zmk.uf2 /ws/$s.uf2
    done; ls -l /ws/*.uf2'
```

| Path | What it is |
|---|---|
| `config/west.yml` | West manifest. `west init -l config` reads this. |
| `config/skeletyl_handwire.keymap` | All six layers. Used by both halves. |
| `config/skeletyl_handwire.conf` | Kconfig for both halves. |
| `config/skeletyl_handwire_left.conf` | Central-only options, merged for the left half. |
| `boards/shields/skeletyl_handwire/` | Matrix, transform, physical layout, split roles. |
| `build.yaml` | GitHub Actions build matrix and artifact names. |
| `zephyr/module.yml` | Makes the repo root a Zephyr module so the shield is found. |

---

## Flash on macOS

`settings_reset.uf2` wipes Bluetooth bonds and the split pairing. Flash it on
each half first when the keyboard is new, or when the halves will not talk to
each other. Skip it for an ordinary keymap update.

Plug in over USB and double-tap reset, or short RST to GND twice. A volume named
`NICENANO` appears. Drag the `.uf2` onto it.

The volume disappears the moment the copy finishes, because the controller
reboots. Finder often reports a failed copy or a bad eject. That is a successful
flash. Treat it as a failure only if the keyboard does not come back.

**Left.** Settings reset, then `skeletyl_handwire_left.uf2`.
**Right.** Settings reset, then `skeletyl_handwire_right.uf2`.

Power-cycle both halves. In System Settings → Bluetooth, pair **Mill Skeletyl**.
Pair the left half only. The right half connects to the left and does not appear
in the Bluetooth list.

Plug straight into the Mac. A USB hub often fails to mount the bootloader drive
even though the keyboard itself works through that hub.

After the first successful flash, layer 3 can reopen the bootloader without the
reset pads. Hold Tab and press Boot on the half you want to update.

```bash
ls /Volumes
cp skeletyl_handwire_left.uf2 /Volumes/NICENANO/
```

`cp` may exit non-zero when the volume vanishes. The flash can still have worked.

---

## When a key is wrong

Test the left half over USB before debugging Bluetooth. That separates a wiring
fault from a radio fault.

| What you see | Where to look |
|---|---|
| One key dead | That switch, its diode, or a joint. The black band faces the row. Continuity is column → switch → diode → row. |
| A whole row dead | The orange wire for that row. R0 P0.17, R1 P0.20, R2 P0.22, R3 P0.24. |
| A whole column dead | The black wire for that column. C0 P0.08, C1 P1.00, C2 P0.11, C3 P1.04, C4 P1.06. |
| Keys work, letters are in the wrong places | `default_transform` in `skeletyl_handwire.dtsi`. The wiring can stay. |
| One key types twice | Reflow the joint. If it persists, raise the debounce times on `kscan0`. |
| Two keys fire from one press | A diode is missing or backwards. |
| Left works, right is silent | Right half power, or the right firmware was flashed onto the wrong controller. Then reset settings on both halves. |
| Each half works on USB, they do nothing together | Flash `settings_reset.uf2` to both, reflash left and right, power-cycle both. |
| The Mac will not pair | Remove Mill Skeletyl from Bluetooth, hold Tab and press Clr, then pair again. |
| It sleeps after a while | 15 minutes, set by `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` in `config/skeletyl_handwire.conf`. Any key wakes it. |

The right half does not type over USB. It only sends keys to the left half. Plug
the left half in, or leave it powered, when you want to test the right half.
