# Skeletyl 36 — handwired, 2x nice!nano, ZMK BLE split

36-key Skeletyl (3x5 + 3 thumbs per half), handwired to two nice!nano-compatible
nRF52840 controllers. Wireless split over BLE, no TRRS. Left half is the
central and is the half your Mac pairs with. Bluetooth name: **Mill Skeletyl**.

Firmware is built and verified against **ZMK `main`** with **Zephyr 4.1.0**.

```
Q W F P B   J L U Y '
A R S T G   M N E I O
Z X C D V   K H , . /
  ESC BSPC TAB   RET SPACE DEL
```

Thumbs are layer-taps (tap the key, hold for the layer). Home-row mods and the
Cmd+C/X/V/Z double taps are documented in `config/skeletyl_handwire.keymap`.

---

## Repo layout

| Path | Purpose |
|---|---|
| `config/west.yml` | West manifest. **Required** — `west init -l config` reads this. |
| `config/skeletyl_handwire.keymap` | Keymap, all three layers. Applies to both halves. |
| `config/skeletyl_handwire.conf` | Kconfig for both halves. |
| `config/skeletyl_handwire_left.conf` | Central-only Kconfig (merged on top for the left half). |
| `boards/shields/skeletyl_handwire/` | The custom shield: matrix, transform, physical layout, split roles. |
| `build.yaml` | GitHub Actions build matrix and output file names. |
| `.github/workflows/build.yml` | Calls ZMK's reusable build workflow. |
| `zephyr/module.yml` | Makes the repo root a Zephyr module so the shield is found. |
| `firmware/` | Locally built `.uf2` files (gitignored). |

---

## Matrix wiring — identical on BOTH halves

Each half is a local 4x5 matrix. Row 3 is the thumb row.

### Rows — ORANGE (sampled inputs, pull-down)

| Matrix row | nRF52840 pin | nice!nano pad |
|---|---|---|
| R0 | P0.17 | D2 |
| R1 | P0.20 | D3 |
| R2 | P0.22 | D4 |
| R3 (thumbs) | P0.24 | D5 |

### Columns — BLACK (driven outputs)

| Matrix column | nRF52840 pin | nice!nano pad |
|---|---|---|
| C0 | P0.08 | D0 / RX |
| C1 | P1.00 | D6 |
| C2 | P0.11 | D7 |
| C3 | P1.04 | D8 |
| C4 | P1.06 | D9 |

All nine pins are plain GPIO on the nice!nano and are free — nothing else in
this firmware claims them.

### Diodes

`COLUMN -> SWITCH -> DIODE -> ROW`, black band (cathode) facing the ROW.

```
diode-direction = "col2row";
```

For `col2row` ZMK drives the columns and samples the rows, which is why the
pull-downs sit on the row GPIOs.

---

## Matrix transform

Logical matrix is 4 rows x 10 columns. The left half takes global columns 0–4;
the right half sets `col-offset = <5>` in `skeletyl_handwire_right.overlay` and
takes global columns 5–9. Both halves flash the same transform and each applies
its own offset, which is how every ZMK BLE split works.

The halves are **mirrored as built** — column order runs the opposite way on
the right half. This was confirmed on hardware: pressing the right half's top
row from the center gap outward originally produced `; y u l j`.

- Left: C0 = far-**left** (pinky) … C4 = far-right (index reach)
- Right: C0 = far-**right** (pinky) … C4 = far-left (index reach)

Rows are *not* mirrored — row 0 is the top row on both halves.

Thumbs occupy the three innermost columns of each half. Because of the
mirroring, the right thumbs count outward:

- Left thumbs = C2, C3, C4 → global columns 2, 3, 4 (outer → inner)
- Right thumbs = C2, C1, C0 → global columns 7, 6, 5 (inner → outer)

So the right half's entries are listed in **descending** column order, which is
what makes position 5 (keymap `J`) land nearest the center gap:

```
RC(0,0..4) then RC(0,9..5)      10 keys   top row
RC(1,0..4) then RC(1,9..5)      10 keys   home row
RC(2,0..4) then RC(2,9..5)      10 keys   bottom row
RC(3,2..4) then RC(3,7..5)       6 keys   thumbs (3 left + 3 right)
                                --------
                                36 keys   = 18 per half
```

Verified from the compiled devicetree — physical left-to-right on each half:

```
LEFT    Q W F P B / A R S T G / Z X C D V / ESC LOWER SPACE
RIGHT   J L U Y ; / M N E I O / K H , . / / RET LOWER BSPC
```

`RC(3,0)`, `RC(3,1)`, `RC(3,8)` and `RC(3,9)` are deliberately absent — no
switch is wired at those intersections.

---

## Keymap

Cantor Remix layers 0–5. Game layers are not in this firmware. The full binding
list, including timings, is `config/skeletyl_handwire.keymap`.

| Thumb | Tap | Hold |
|---|---|---|
| Left outer | Esc | Layer 1, media |
| Left middle | Backspace | Layer 2, numbers |
| Left inner | Tab | Layer 3, function / Bluetooth |
| Right inner | Enter | — |
| Right middle | Space | Layer 4, symbols |
| Right outer | Delete | Layer 5, mouse |

Home row: A/R/S/T are Gui, Alt, Ctrl, Shift. N/E/I/O are Shift, Ctrl, Alt, Gui.
Double-tap Q, Z, X, or V for Cmd+C, Cmd+Z, Cmd+X, Cmd+V.

Layer 3, held with Tab:

```
F1  F2  F3  F4  F5   | F6  F7  F8  F9  F10
         ... F11     | F12 ...
BT0 BT1 BT2 BT3 BOOT | INS HOME PGDN PGUP END
      .  CAPS  ___   | BOOT BT4  BTCLR
```

`BT0`–`BT4` select a Bluetooth profile. `BTCLR` clears only the selected
profile and sits on the outer right thumb, so it needs Tab held as well.
`BOOT` on the left half reboots the left controller; `BOOT` on the right thumb
reboots the right one. Neither is on the base layer.

---

## Building

### GitHub Actions (normal route)

`.github/workflows/build.yml` calls ZMK's reusable build workflow on every
push. Push the repo, open the **Actions** tab, wait for the run, then download
the `firmware` artifact. Inside the zip:

```
skeletyl_handwire_left.uf2
skeletyl_handwire_right.uf2
settings_reset.uf2
```

### Locally, with Docker

Uses the same container image as CI, so no host toolchain is needed.

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

### ZMK version

`config/west.yml` tracks `main` because the `nice_nano//zmk` board target comes
from ZMK's Zephyr 4.1 / hardware-model-v2 migration, which landed after the
newest tagged release (v0.3.0). ZMK's workflow hard-errors if you build a plain
`nice_nano` while a `zmk` variant exists.

To pin to a release instead: set `revision: v0.3` in `config/west.yml`, change
`@main` to `@v0.3` in `.github/workflows/build.yml`, and change every `board:`
in `build.yaml` to `nice_nano_v2`. v0.3 predates the board variant and only
understands the old flat board names.

---

## Flashing on macOS

`settings_reset.uf2` wipes stored settings — BLE bonds, saved profiles, and the
split pairing between the two halves. Flash it on each half **first** on a
fresh build, so the halves negotiate a clean pairing.

Entering the bootloader: with the board plugged in over USB, quickly
double-tap RESET (or short RST to GND twice). A removable volume appears in
Finder, usually named `NICENANO`.

### Left half

1. Plug the **left** half into the Mac over USB.
2. Double-tap reset / short RST to GND twice to enter the bootloader.
3. Drag `settings_reset.uf2` onto the mounted volume.
4. Re-enter the bootloader (double-tap reset again).
5. Drag `skeletyl_handwire_left.uf2` onto the volume.
6. Unplug.

### Right half

1. Plug the **right** half into the Mac over USB.
2. Double-tap reset / short RST to GND twice.
3. Drag `settings_reset.uf2` onto the volume.
4. Re-enter the bootloader.
5. Drag `skeletyl_handwire_right.uf2` onto the volume.
6. Unplug.

### Then

1. Power-cycle both halves with their battery switches.
2. On the Mac: **System Settings → Bluetooth**, pair **Mill Skeletyl**.
   Pair the **left** half only. The right half connects to the left over ZMK's
   own split link and never appears in the Bluetooth list.
3. Test all 36 keys.

### Expected macOS weirdness

The UF2 volume disappears the instant the copy finishes, because the controller
reboots itself as soon as it has the whole file. Finder often reports a copy
error, an "incomplete" copy, or that the disk was ejected improperly. **This is
normal and the flash succeeded.** Only treat it as a failure if the keyboard
does not come back up.

From the Terminal instead:

```bash
ls /Volumes                                   # find the bootloader volume
cp firmware/skeletyl_handwire_left.uf2 /Volumes/NICENANO/
```

`cp` may likewise exit non-zero on the reboot. Again, expected.

---

## Hardware test checklist

Test the **left** half over USB first, before dealing with Bluetooth at all —
that isolates matrix problems from radio problems. Open a keyboard tester and
press every key exactly once.

| Symptom | Almost always | What to check |
|---|---|---|
| One key dead | That key's switch, diode or a solder joint | Reflow both switch pins. Check the diode is not cracked and that the black band faces the **row**. Check continuity column → switch → diode → row. |
| A whole row dead (5 keys, or 3 on the thumb row) | That row's GPIO wire | Reflow the orange wire at the controller pad and at the first switch in the chain. R0=P0.17/D2, R1=P0.20/D3, R2=P0.22/D4, R3=P0.24/D5. |
| A whole column dead (3 keys, 4 with a thumb) | That column's GPIO wire | Reflow the black wire at the controller pad. C0=P0.08/D0, C1=P1.00/D6, C2=P0.11/D7, C3=P1.04/D8, C4=P1.06/D9. |
| Keys work but produce the **wrong** letters | Logical transform, not wiring | The matrix is fine. Note which physical key produced which letter and adjust only `default_transform` in `skeletyl_handwire.dtsi`. Do not rewire. This already happened once — the right half read mirrored, and was fixed by reversing its column order in the transform. |
| A key fires twice per press | Debounce or a cold joint | Reflow the joint first. If it persists, raise `debounce-press-ms` / `debounce-release-ms` on `kscan0`. |
| Two keys fire together | Missing or backwards diode | A backwards diode ghosts into its neighbours. Check the band direction. |
| Left works, right does nothing | Split pairing, right-half config, or right-half power | Confirm the right half has power (its own battery/switch, and check the LED). Confirm you flashed `skeletyl_handwire_right.uf2` to the right half and not the left build. Then reset settings on both halves (below). |
| Each half works alone over USB, but not together | Stale bonding state on one or both halves | Flash `settings_reset.uf2` to **both** halves, then reflash the correct left/right firmware to each, then power-cycle both. |
| Pairing to the Mac is broken | Stale bond on the Mac side too | Remove **Mill Skeletyl** from macOS Bluetooth, then use `BTCLR` on the SYSTEM layer (or reflash `settings_reset.uf2`), then pair again. |
| Keyboard drops out after a few minutes | Deep sleep, working as configured | Any keypress wakes it. To change the 15-minute timeout, edit `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` in `config/skeletyl_handwire.conf`. |

A dead row **and** a dead column crossing at one key usually means two separate
joints, not one fault — check them independently.
