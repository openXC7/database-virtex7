# Segbit fixes carried in this database

Substantive corrections this virtex7 DB carries over a stock prjxray fuzz.
These are what make the fully-open VC707 flow produce correct bitstreams.
(Commit hashes refer to the prjxray fuzzer-code repo that produced them.)

## CLB OUTMUX.O5 duplicate → incomplete encoding  (`6fd1d26`)
The "telegraph bug." Duplicate `OUTMUX.O5` segbit definitions; last-wins
picked the incomplete one (missing the shared xMUX-enable bit) → output muxes
encoded as `0x55`. Fix: dedupe keeping the **complete** definition. This was
the root cause of long-chased open-flow output corruption.

## IOB18 OLOGIC SING-row segbits  (`afb49cc`, fuzzer `036-iob18-ologic-sing`)
SING-row IOB18 sites had no OLOGIC segbits. Added via a dedicated sub-fuzzer.

## LIOI / RIOI OLOGIC inversion segbits  (local fuzz, superset)
`segbits_lioi*.db` / `segbits_rioi_sing.db` gained the `OLOGIC_Y*.IS_*_INVERTED`
features (`IS_CLKDIV_INVERTED`, `IS_D1..D4_INVERTED`, etc.) — +88 lines per
file over the last release (216 vs 128). Plus `mask_lioi*.db` and
`tileconn.json`. This is why the build-area copy is a strict superset of the
released DB.

## LIOB18 / RIOB18 IOB_Y1 IN_ONLY  (`09588f3`)
Dropped spurious bits `!38_32 !38_34` from the Y1 (slave-site) input-only
encoding, fixing left-HP single-ended LVCMOS inputs.

## Tilegrid SING propagation for HP-only LIOB18 / LIOI columns  (`1c63972`, `1574e9c`)
Propagated SING-row bits and added the `iob18_sing` sub-fuzzer so HP-only
columns get correct SING-row geometry.

## Top-SING tile alias start_offset (OLOGIC/IOB "invalid word address")
`tilegrid.json`: the **top** SING-row tiles (`LIOI_SING`/`RIOI_SING`/`LIOB18_SING`/
`RIOB18_SING` at `bits.CLB_IO_CLK.offset == 99`) had alias `start_offset: 0`,
but it must be **2** (matching the bottom SING tiles at `offset 0`). With
`start_offset 0` the aliased regular OLOGIC_Y0 / IOB features (regular words
2–3) mapped to absolute frame words 101–102, which don't exist — fasm2frames
emitted `invalid word address` and silently dropped them (e.g. VC707 counter
LED on `LIOI_SING_X82Y51.OLOGIC_Y0.OMUX.D1`). With `start_offset 2` the
effective offset becomes `99-2=97`, so regular word 3 → abs 100, word 2 → abs
99 — landing in the tile's real 2-word window.

Confirmed empirically: fuzzer `036-iob18-ologic-sing` (re-run at N=50) places
`OLOGIC_Y0.OMUX.D1` at abs word 100, exactly what `start_offset 2` produces.
28 tiles fixed (7 each of LIOI/RIOI/LIOB18/RIOB18_SING). (The dedicated
`segbits_*_sing.db` are NOT consulted for aliased tiles, so this is a tilegrid
fix, not a segbits one.) Follow-up: fix the generator (`fuzzers/005-tilegrid`)
so a regenerated tilegrid doesn't reintroduce `start_offset 0`.

## Known remaining gap (worked around downstream, not yet fixed in DB)
LVCMOS18 single-ended **input** on certain HP-bank sites (e.g. AU33) is missing
segbit `bit_0042101c_063_18` and sets a spurious `IBUF_HP_BANK_GLUE` — the
rx-distortion bug. Currently patched at the frame level in the demo
(`uartram/patch_rx_iob.py`); a proper fix belongs here as a segbit correction.
