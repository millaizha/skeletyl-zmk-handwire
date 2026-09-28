# Skeletyl 36 — handwired, 2x nice!nano, ZMK BLE split

36-key Skeletyl (3x5 + 3 thumbs per half), handwired to two nice!nano-compatible
nRF52840 controllers. Wireless split over BLE, no TRRS. Left half is the
central and is the half your Mac pairs with. Bluetooth name: **Mill Skeletyl**.

Firmware is built and verified against **ZMK `main`** with **Zephyr 4.1.0**.

```
Q W F P B   J L U Y ;
A R S T G   M N E I O
Z X C D V   K H , . /
    ESC LWR SPC   RET LWR BSPC
```

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

Physical columns are numbered left-to-right **on each half**, so:

- Left: C0 = far-left (pinky) … C4 = far-right (index reach)
- Right: C0 = far-left (index reach) … C4 = far-right (pinky)

Thumbs occupy only the three innermost columns of each half:

- Left thumbs = C2, C3, C4 → global columns 2, 3, 4
- Right thumbs = C0, C1, C2 → global columns 5, 6, 7

```
RC(0,0..9)      10 keys   top row
RC(1,0..9)      10 keys   home row
RC(2,0..9)      10 keys   bottom row
RC(3,2..7)       6 keys   thumbs (3 left + 3 right)
                --------
                36 keys   = 18 per half
```

`RC(3,0)`, `RC(3,1)`, `RC(3,8)` and `RC(3,9)` are deliberately absent — no
switch is wired at those intersections.

---

## Keymap

### Layer 0 — BASE (Colemak-DH)

```
Q   W   F   P   B   |   J   L   U   Y   ;
A   R   S   T   G   |   M   N   E   I   O
Z   X   C   D   V   |   K   H   ,   .   /
        ESC LWR SPC | RET LWR BSPC
```

### Layer 1 — LOWER (hold either thumb LWR)

```
1    2    3    4    5   |  6     7     8     9    0
F1   F2   F3   F4   F5  | LEFT  DOWN   UP  RIGHT  DEL
BT0  BT1  BT2  BT3  BT4 | HOME  PGDN  PGUP  END   TAB
          SYS  --  SPC  | RET   --   BSPC
```

`BT0`–`BT4` select Bluetooth profile 0–4. These are safe — they only switch
which host profile is active and never erase a bond. Use them to hop between
paired machines.

The left outer thumb becomes **SYS** on this layer.

### Layer 2 — SYSTEM (hold LOWER + left outer thumb)

```
BTCLR  BTCLRA   --      --  --  |  --   --   --      --      --
OUTTOG OUTUSB   OUTBLE  --  --  |  --   F6   F7      F8      F9
BOOT-L RESET-L  --      --  --  |  --   F10  F11  RESET-R  BOOT-R
                --      --  --  |  --   --   --
```

| Key | Effect |
|---|---|
| `BTCLR` | Clear the bond for the **currently selected** profile |
| `BTCLRA` | Clear the bonds for **all** profiles |
| `OUTTOG` / `OUTUSB` / `OUTBLE` | Switch host output between USB and BLE |
| `BOOT-L` / `BOOT-R` | Reboot that half into the UF2 bootloader |
| `RESET-L` / `RESET-R` | Soft reset that half |

The two Bluetooth-clear keys are destructive, so they deliberately need three
keys held at once (LOWER + left outer thumb + the key). They cannot be hit by
accident while typing, and nothing destructive lives on the base layer.

`BOOT-L`/`RESET-L` are physically on the left half and act on the left
controller; `BOOT-R`/`RESET-R` are on the right half and act on the right
controller. After the first flash this means you can re-enter the bootloader on
either half from the keyboard itself, without shorting RST to GND again.

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
| Keys work but produce the **wrong** letters | Logical transform, not wiring | The matrix is fine. Note which physical key produced which letter and adjust only `default_transform` in `skeletyl_handwire.dtsi`. Do not rewire. |
| A key fires twice per press | Debounce or a cold joint | Reflow the joint first. If it persists, raise `debounce-press-ms` / `debounce-release-ms` on `kscan0`. |
| Two keys fire together | Missing or backwards diode | A backwards diode ghosts into its neighbours. Check the band direction. |
| Left works, right does nothing | Split pairing, right-half config, or right-half power | Confirm the right half has power (its own battery/switch, and check the LED). Confirm you flashed `skeletyl_handwire_right.uf2` to the right half and not the left build. Then reset settings on both halves (below). |
| Each half works alone over USB, but not together | Stale bonding state on one or both halves | Flash `settings_reset.uf2` to **both** halves, then reflash the correct left/right firmware to each, then power-cycle both. |
| Pairing to the Mac is broken | Stale bond on the Mac side too | Remove **Mill Skeletyl** from macOS Bluetooth, then use `BTCLR` on the SYSTEM layer (or reflash `settings_reset.uf2`), then pair again. |
| Keyboard drops out after a few minutes | Deep sleep, working as configured | Any keypress wakes it. To change the 15-minute timeout, edit `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` in `config/skeletyl_handwire.conf`. |

A dead row **and** a dead column crossing at one key usually means two separate
joints, not one fault — check them independently.
