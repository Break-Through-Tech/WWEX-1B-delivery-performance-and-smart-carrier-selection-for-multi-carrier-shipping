# `data/row_flags/`

This folder holds a list of specific rows that should probably be dropped before
modeling. Nobody edits `data/train.csv` or `data/test.csv` directly — instead, the
rows to drop are written down here, and a later shared step reads this list and does
the actual dropping, once.

**This is only for duplicate `shipment_id` rows.** `shipment_id` is the primary key —
every other column group's data quality issues (missing values, bad units, messy
categories, etc.) get fixed *inside* their own column, not by dropping the row. So the
identifiers/dates group is the only one that needs to write here, and there's one file,
not a folder per column group.

## What's in `flagged_rows.csv`

| column        | meaning                                                              |
|---------------|-----------------------------------------------------------------------|
| `file`        | `"train"` or `"test"` — which file this row came from.                |
| `row_index`   | the row's position in that file (row 0 is the first data row — same as `pd.read_csv(...).index`). |
| `shipment_id` | included for readability when skimming the file.                      |
| `reason`      | why it's flagged: `"exact_duplicate_shipment_id"` or `"train_test_id_overlap"`. |

**Why `file` + `row_index`, and not just `shipment_id`?** Two reasons:

- `train.csv` and `test.csv` are separate files. Row 5 in `train.csv` and row 5 in
  `test.csv` are unrelated rows, so a flag needs to say which file it's about.
- The problem *is* duplicate ids — two rows can share the exact same `shipment_id`.
  If a flag only said "drop this `shipment_id`", it would drop every row with that id
  instead of just the one extra copy. `row_index` always points at one specific row,
  no ambiguity.

There are three cases in here, and they all work the same way, just across different
files:

- **Duplicate rows within `train.csv`**: the same `shipment_id` appears twice in the
  same file (exact copies). Keep one, drop the other.
- **Duplicate rows within `test.csv`**: the same thing, but checked separately in
  `test.csv` — it has its own handful of duplicate ids, unrelated to train's.
- **`shipment_id`s that show up in both `train.csv` and `test.csv`**: one row in each
  file. Both get flagged so whoever decides what to do about it (drop from train,
  drop from test, or leave it) can see both sides.

## Where this comes from and where it goes

1. `notebooks/exploration/identifiers/findings.ipynb` finds the duplicates and writes
   `flagged_rows.csv`. It never drops anything itself — diagnostic only.
2. `notebooks/exploration/identifiers/fixes.ipynb`, and every other column group's
   `fixes.ipynb`, only contain **column-level** fixes (fixing a value, not dropping a
   row). None of them touch `flagged_rows.csv`.
3. A shared script (not built yet — something like `clean.py`) will read
   `flagged_rows.csv`, drop those rows from the raw files, and then apply everyone's
   column-level fixes on top of that to produce the final clean dataset.

Until that script exists, `flagged_rows.csv` is just sitting here as the agreed-on
list — nothing has actually been dropped from `train.csv`/`test.csv` yet.
