# openXC7 Virtex-7 device database (prjxray `database/virtex7`)

Canonical, version-controlled copy of the prjxray **virtex7** device database
(segbits / tilegrid / part data) used by the openXC7 flow for the VC707
(`xc7vx485tffg1761-2`).

Upstream prjxray **gitignores its `database/` tree** (it is fuzzer *output*),
so this data has historically lived only on the fuzzing machine. This repo
exists to make it a first-class, diffable, releasable artifact — the canonical
source of truth and the durable off-machine backup.

## What's here

The full `database/virtex7` tree, committed **raw** (no `.gitignore` hiding the
generated `.db`/`.json`):

* `segbits_*.db`   — feature → bit mappings (the fuzzed encoding)
* `tilegrid.json`, `tileconn.json`, `*.json` — geometry / connectivity
* `part.yaml`, `*.csv` — part metadata

These are text and `diff`/`blame` cleanly — every segbit change is reviewable.

## Derived artifacts (NOT stored here — regenerated, published as release assets)

* **nextpnr chipdb** `xc7vx485t.bin` — built from this DB via nextpnr-xilinx
  bbaexport + RapidWright. Its `.bba` format is tied to a specific
  nextpnr-xilinx commit; that commit is pinned in each release's manifest.
* **json2dcp wire oracle** `xc7vx485tffg1761-2.oracle.txt.gz` — built from
  RapidWright (`BuildWireOracle.java`).

A release tag bundles this DB commit + the chipdb + the oracle + `SHA256SUMS`,
so the three can never drift out of sync (the previous failure mode, where the
chipdb and DB carried different dates from different repos).

## Reconciliation workflow

Fuzzing / hand-patching happens **in place in this directory** (it is the
prjxray build area). Because the DB is now tracked:

```
# after a fuzz/patch session that improves the DB:
git status        # shows exactly which segbits changed (invisible before)
git diff          # review the encoding deltas
git add -A && git commit -m "..."   # capture the local patches
git tag device-db-YYYY-MM-DD        # cut a release; CI regenerates chipdb+oracle
```

Pulling a newer upstream release is a `git merge` against this repo —
conflicts between local patches and the release surface explicitly instead of
one copy silently clobbering the other.

See `FIXES.md` for the substantive segbit corrections this DB carries over a
stock prjxray fuzz.
