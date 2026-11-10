# 68030 breadboard adaptor

KiCad project for the adapter that carries the MC68030 PGA of Sparky1, a homebrew 68030 breadboard
computer (`../sparky1`). Two copper layers, no inner planes.

## Which revision is real

- **`v1.0.0` (git tag, 2022) is the one on the board.** Its silkscreen says 1.0.0.
- **Branch `v1-kicad10` is v1.0.0 migrated to KiCad 10**, with net-label fixes and this board's module
  symbol (`module/`). It is the branch Sparky1's schematic pins as a submodule.
- **v2.0.0, on `master`, was never manufactured** (Graeme, 2026-10-30). It rearranges J3 and J4 and
  has 2 and 1 grounds there, and adds `/RESET`, `/RMC` and `/DBEN` to the headers.

Anything about the fitted board must come from the tag or `v1-kicad10`, never `master`:
`git show v1.0.0:68030-breadboard.kicad_pcb`. Counting from `master` has already put wrong figures in
`../sparky1/CLAUDE.md`. Both revisions have 512 tracks, and A14 and A17 measure the same on both;
the other nets have not been compared.

## The module symbol

`module/` holds this board's interface symbol for Sparky1's schematic: its pins are the header pins,
named for their signals, extracted from this schematic. After any change to the headers, run
`make -C module` and commit the result with it; `make -C module check` confirms the symbol still
matches the schematic. The tools are the `lib/kicad-tools` submodule.

## The fitted board, v1.0.0

Two rows of 52 pins at 2.54 mm, J2 then J3 and J1 then J4:

| Row | Pins | Grounds (row position) | +5 V (row position) |
|---|---|---|---|
| Data | J2: D0–D31, J3: control | 33, 40 (J3 pins 1, 8) | 48, 52 (J3 pins 16, 20) |
| Address | J1: A0–A31, J4: control | 33, 46 (J4 pins 1, 14) | 34, 52 (J4 pins 2, 20) |

So no ground falls among the 64 address and data pins. D31 and A31 sit beside a ground, and D0 and
A0 are 81 mm from one. `/RESET`, `/RMC` and `/DBEN` are on no header: on the board `/RESET` reaches
the CPU by a hand-soldered wire to the PGA.

## Things to do

- **Add more ground pins and distribute them among the data and address pins.** The 32-pin runs have
  none today, so every return current has to go round to the far end of the row.
  - Aim for a ground every 4 signals, and every 8 at worst, with at least one beside each byte lane
    of the data bus. `../sparky1/CLAUDE.md` ("Breadboard adapters: ground pins on the external
    connectors") has the reasoning. Apple's 68030 processor-direct slot, on a machine of about the
    same clock, has a ground or +5 V pin for every 8 signals, spread through the bus (Table 14-7 area
    of `../datasheets/Apple Guide to the Macintosh Family Hardware 2nd Edition`).
  - The headers are full, so this means longer rows.
  - Keep the grounds at the same positions as on the memory board (`sram-breadboard-adapter`), so
    that they land on common breadboard rows and can be strapped with short links.
  - The extra pins must reach a low-impedance ground, or they only move the bottleneck. Check that
    two copper layers can give them one.
- **Put the oscillator and the FPU on the next version, if there is one** (Graeme, 2026-10-30). No
  68882 is fitted today, and busdecoder has no pins left to decode one, so the decode has to come
  with it.

## The next board is probably the backplane's CPU card

There may never be another adapter: the likely next step is straight to `../sparky1-backplane`
(Graeme, 2026-10-30). Everything above then belongs to the CPU card, which goes in slot 1 and is
already meant to carry the oscillator. What carries over:

- **The grounds are solved at the connector, and have to be kept on the card.** The DIN 41612 has
  row b between the address and data rows, with a ground every fourth pin: one for every 8 of those
  lines, the "at worst" target above. The card must not lose that: a solid ground plane under the
  connector, four layers and a short via at every row b ground.
- **`/RESET` stops being a hand-soldered wire:** it is on the bus (row b).
- **The FPU stays local to the card.** It should need no bus pins, if its decode is in the card's
  CPLD; check that against `../sparky1-backplane/doc/card-rules.md` when the card is designed.
