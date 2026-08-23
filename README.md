# zmk-config

ZMK configuration for my keyboards. Every push builds **three** firmware
variants in parallel, each downloadable as its own archive.

| Variant | Board | Layout | Artifact |
|---|---|---|---|
| chrl-kbd v1 | chrl-kbd v1 | 42-key | `chrl-kbd-v1` |
| Lily58, original | Lily58 | 58-key, as it always was | `lily58-original` |
| Lily58, training | Lily58 | 42-key, surplus keys inert | `lily58-42key` |

The Lily58 keeps a route back to its old layout, so adapting to 42 keys never
costs you a working keyboard.

## Repo layout

```
build.yaml             + config/            -> chrl-kbd v1
build-lily58.yaml      + config-lily58/     -> Lily58, original
build-lily58-42.yaml   + config-lily58-42/  -> Lily58, training
```

Three config directories rather than one, because **ZMK derives the keymap
filename from the shield name** — shield `lily58_left` always reads
`<config>/lily58.keymap`. Two Lily58 layouts therefore cannot share a config
directory, and `config_path` is an input to the reusable workflow rather than a
per-matrix-entry key. Hence three calls in `.github/workflows/build.yml`.

Each directory needs its own `west.yml`, since the workflow runs
`west init -l <config_path>`. They are identical apart from `self.path`; if you
bump the ZMK revision, **bump it in all three**.

## Layout

QWERTY base with home-row mods, and four layers on top. Shared by chrl-kbd v1
and the Lily58 training build.

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

The board is reversible: one PCB design, flipped, with solder jumpers on the
controller footprint selecting the nets. Both halves share a single pin
definition and only the matrix transform differs. Two things in
`config/boards/shields/chrl_kbd_v1/chrl_kbd_v1.dtsi` look like typos and are
not — the descending `row-gpios` and the reversed right half of the transform.
Both are commented at length. Do not "fix" them.

**Not yet built or flashed.** The pin map came from the fabricated PCB, but
nothing has been verified on hardware.

### Displays

Both halves are footprinted and routed for a nice!view on pro-micro pins
0/1/2. That is *not* where the stock `nice_view_adapter` shield looks, so
adding it would give a blank screen rather than a build error. Populating a
display needs an SPI pinctrl override. Details in `config/chrl_kbd_v1.conf`.

## Lily58, training build

The 42-key layout mapped onto the Lily58, with the 16 keys the new board does
not have set to `&none`:

- the number row (12)
- the inner `[` and `]` (2)
- the outermost thumb on each half (2)

They are `&none` rather than left working on purpose. Reaching for them has to
fail, or the habit survives until the new board arrives and you relearn under
pressure.

## Building

Every push builds all three. Open the run, download the archive you want from
Artifacts, then flash each half:

1. Double-tap the reset button to mount the nice!nano as a USB drive.
2. Copy the matching `.uf2` onto it. It reboots itself.

Left and right firmware are **not** interchangeable — the halves differ in
split role and matrix offset. Flash `..._left.uf2` to the left half.

Switching a Lily58 between layouts is just flashing the other archive; both
halves need doing.
