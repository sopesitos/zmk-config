# Corne — 3-piece dongle firmware (ZMK)

This config builds **three** firmwares for a Typeractive Corne running on
3× nice!nano v2:

```
        BLE                BLE
 left  ◀────▶   DONGLE   ◀────▶  right
(periph)      (central)         (periph)
                 │ USB
                 ▼
                PC
```

- **`corne_dongle`** — BLE **central**. Runs the keymap and connects to the PC
  over **USB**. Bare board (no display, no ZMK Studio).
- **`corne_left`** / **`corne_right`** — BLE **peripherals**. Each scans its own
  keys and reports key positions to the dongle. Both keep their nice!view.

Your keymap is **unchanged**: all 7 layers (Base, Lower, Raise, gamerz, musica,
after effects, mac), the `LongPress` hold-tap, and every shortcut are preserved.
(The one exception: the old `&studio_unlock` key on the Raise layer is now
`&trans`, because ZMK Studio was dropped. Remap it to anything you like in
`config/corne.keymap`.)

Pinned to ZMK **v0.3** (see `config/west.yml`) — no external modules.

---

## Repo layout

```
build.yaml                                  # 4 builds: left, right, dongle, settings_reset
.github/workflows/build.yml                 # GitHub Actions build (push -> .uf2 artifacts)
config/
  west.yml                                  # ZMK v0.3 manifest (unchanged)
  corne.keymap                              # your keymap — single source of truth
  corne.conf                                # shared settings for the two halves
  corne_dongle.keymap                       # one line: #include "corne.keymap"
  corne_dongle.conf                         # dongle BLE tuning
boards/shields/corne_dongle/
  Kconfig.shield                            # declares the corne_dongle shield
  Kconfig.defconfig                         # central role + 2 peripherals + BT headroom
  corne_dongle.overlay                      # mock kscan + Corne matrix transform
```

> **Folder name:** the config folder is lowercase `config/` (ZMK's standard, what
> the GitHub Actions workflow expects). If your existing GitHub repo uses a
> different name, either rename it to `config` or set `config_path:` on the
> workflow.

---

## Build

This builds in the cloud via GitHub Actions — no local toolchain needed.

1. Commit these files to your `zmk-config` GitHub repo and `git push`.
2. Open the repo's **Actions** tab → the latest **Build ZMK firmware** run.
3. Download the **firmware** artifact (a zip) once it's green. It contains:
   - `corne_dongle.uf2`
   - `corne_left.uf2`
   - `corne_right.uf2`
   - `settings_reset.uf2`

---

## Flash (first time — order matters)

Each nice!nano enters its bootloader on a **double-tap of the reset button**; it
then shows up as a USB drive (`NICENANO`). Copy the `.uf2` onto that drive — it
reboots automatically.

**Step 1 — clear old pairings (all three boards).**
Your halves are currently bonded to each other. Flash **`settings_reset.uf2`** to
the **dongle, the left half, and the right half** so they forget old bonds and
can pair to the dongle.

**Step 2 — flash the real firmware.**
- `corne_dongle.uf2` → the dongle
- `corne_left.uf2` → the left half
- `corne_right.uf2` → the right half

**Step 3 — power up.**
Plug the **dongle into the PC via USB** and switch on both halves. The dongle
scans and connects to both halves automatically (give it a few seconds). Start
typing — the dongle reports as a USB keyboard.

> The keyboard only works while the **dongle is powered** (it's the central). The
> halves run on their own batteries.

---

## Editing your keymap later

Edit **`config/corne.keymap`** only — the dongle includes it, so all three builds
stay in sync. Push, and Actions rebuilds all firmwares. Reflash whichever boards
changed (a keymap edit only needs the **dongle** reflashed, since it runs the
keymap).

The Raise/Lower layers still have `&bt BT_SEL n` / `&bt BT_CLR`. On the dongle
these manage optional Bluetooth **host** profiles (e.g. connecting the dongle to
a device wirelessly instead of USB). They do **not** affect the dongle↔halves
link, which is automatic.

---

## Troubleshooting

- **A half won't connect:** re-flash `settings_reset.uf2` to that half **and** the
  dongle, then re-flash their normal firmware. Bonds must be cleared on both ends.
- **Only one half works:** confirm the dongle was built with
  `CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2` (set in
  `boards/shields/corne_dongle/Kconfig.defconfig`).
- **Left/right swapped or wrong keys:** make sure you flashed `corne_left.uf2` to
  the left board and `corne_right.uf2` to the right.
- **Nothing types:** the dongle must be plugged into the PC and powered; the
  halves must be on and connected.
