# zmk-config

ZMK configuration for my keyboards. Two boards, one config, one build matrix.

| Board | Keys | Status |
|---|---|---|
| [chrl-kbd v1](#chrl-kbd-v1) | 42 (3×6+3 split) | PCBs ordered, not yet assembled |
| [Lily58](#lily58) | 58 | In use, running the 42-key training keymap |

Both boards run the **same layout**. The Lily58 runs it with its 16 surplus keys
made inert, so the muscle memory transfers intact.

## Layout

QWERTY base with home-row mods, and four layers on top.

| Layer | Held with | Holds |
|---|---|---|
| Base | — | QWERTY; mods on `ASDF` / `JKL;`, ctrl on the left thumb |
| LO | left thumb | F1–F12, shifted symbols, clipboard on `ZXCV` |
| HI | right thumb | numbers, arrows on `HJKL`, Home/PgDn/PgUp/End |
| ADJ | both thumbs | Bluetooth, media, bootloader, reset |
| Game | `,`+`/` combo | no home-row mods, real shift and ctrl |

Home-row mods are `tap-preferred` at 200 ms — a quick tap always gives the
letter, so single-key vim commands are safe. GUI is deliberately *only* a
home-row mod, which frees the outer left thumb for ctrl; that makes
ctrl+shift+key a thumb plus one finger instead of two home-row mods at once.

## chrl-kbd v1

42-key wireless split, Choc v1 hotswap, dual nice!nano v2. Hardware design
lives in a separate repo — PCB, case, and the decision log.

- Keymap: `config/chrl_kbd_v1.keymap`
- Settings: `config/chrl_kbd_v1.conf`
- Shield: `config/boards/shields/chrl_kbd_v1/`

The board is reversible — one PCB design, flipped, with solder jumpers on the
controller footprint selecting the nets. Both halves therefore share a single
pin definition, and only the matrix transform differs. See the comments in
`chrl_kbd_v1.dtsi`; the column reversal on the right half is deliberate and
documented there, because it is the thing most likely to be "fixed" into a
mirrored keyboard by someone tidying up.

**Not yet built or flashed.** The pin map came from the fabricated PCB, but
nothing has been verified on hardware.

### Displays

Both halves are footprinted and routed for a nice!view on pro-micro pins
0/1/2. That is *not* where the stock `nice_view_adapter` shield looks, so
adding it would give a blank screen rather than a build error. Populating a
display needs an SPI pinctrl override. Details in `config/chrl_kbd_v1.conf`.

## Lily58

The old board, now the training board for the new layout. Its keymap is the
42-key layout mapped onto the Lily58 with the 16 keys the new board does not
have set to `&none`:

- the number row (12)
- the inner `[` and `]` (2)
- the outermost thumb on each half (2)

They are `&none` rather than left working on purpose. Reaching for them has to
fail, or the habit survives until the new board arrives and you relearn under
pressure.

- Keymap: `config/lily58.keymap`
- Settings: `config/lily58.conf`

## Building

Every push builds both boards via GitHub Actions. Download the artefact from
the run, then flash each half:

1. Double-tap the reset button to mount the nice!nano as a USB drive.
2. Copy the matching `.uf2` onto it. It reboots itself.

Left and right firmware are **not** interchangeable — the halves differ in
split role and matrix offset. Flash `..._left.uf2` to the left half.

To build only one board, comment out the other entries in `build.yaml`.
