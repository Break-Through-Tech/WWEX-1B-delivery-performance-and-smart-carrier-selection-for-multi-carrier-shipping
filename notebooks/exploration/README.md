# `notebooks/exploration/`

This is where everyone does their data-quality work, one folder per assigned column
group:

```
notebooks/exploration/
  identifiers/
    findings.ipynb
    fixes.ipynb
  weights/
    findings.ipynb
    fixes.ipynb
  categoricals/
    ...
```

If you're picking up a new column group, copy this structure: make a folder named
after your columns, with a `findings.ipynb` and a `fixes.ipynb` inside.

## `findings.ipynb` — look, don't touch

Explore your assigned columns and write down every data quality issue you find:
missing values, bad formats, impossible values, duplicates, whatever. Read
`train.csv`/`test.csv`, print/plot what's wrong, explain it in markdown cells. **Never
change or drop anything in the actual data here** — this notebook is a report, not a
cleaning step.

The one exception: if you find that a whole *row* should probably be dropped (not just
a value fixed), see `../../data/row_flags/README.md`. In practice this is really just
the identifiers group's job, since a duplicate is fundamentally about `shipment_id`
(the primary key) — if your columns turn up something that looks like a full-row
problem, it's worth flagging in your findings and mentioning it to the identifiers
group rather than inventing a new row-drop file.

## `fixes.ipynb` — your column-level fixes

Write the actual fix for each issue you found, as a small function that takes the raw
column(s) as input and returns the corrected column:

```python
def clean_carrier(raw_df):
    return raw_df["carrier"].str.strip().str.title()
```

Rules that keep everyone's fixes safe to combine:

- **Always start from the raw column**, never from another notebook's already-cleaned
  output. That way it doesn't matter what order fixes get applied in later.
- **Don't drop rows here.** If a value is unfixable (can't tell what it should be),
  turn it into a missing value (`NaN`) or flag it — don't remove the row. Row drops are
  handled once, centrally, for everyone (see below).
- **Only touch your own columns.** If a fix genuinely needs another column for context
  (like checking `shipment_mode` to fix `weight_lbs`), that's fine as long as you read
  it from the raw data, not from someone else's cleaned version.

## `clean.py` — the shared step (not built yet)

Once everyone's `findings.ipynb` and `fixes.ipynb` are done, one shared script will
actually produce the clean dataset. It has two stages, always in this order:

1. **Drop the duplicate rows.** Read `data/row_flags/flagged_rows.csv` and drop every
   `(file, row_index)` in it from the matching raw file (`train.csv` or `test.csv`).
   This happens first and only once, so every column-level fix after it is working
   with the same, already-deduped rows.
2. **Apply every column-level fix, one at a time.** For each column group, call its
   `fixes.ipynb` function(s) and write the result into the matching column of the
   (now deduped) dataframe. Since every fix reads from the raw column and only writes
   its own column, these can run in any order — nobody's fix depends on another's
   having already run.

The output is `train_clean.csv`/`test_clean.csv` (or similar), which is what everyone
should build models on from that point on — not the raw files, and not anyone's
individual notebook output.
