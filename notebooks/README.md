# `notebooks/`

## `clean_data.ipynb` / `clean_test_data.ipynb` — the canonical pipeline

These are the single source of truth for turning `data/train.csv` /
`data/test.csv` into model-ready data. They dedup rows (dropping anything in
`data/row_flags/flagged_rows.csv`), canonicalize the categorical columns
(carrier/service_level/shipment_mode), fix ZIP codes that lost a leading
zero, and impute missing weight/dimension values. Each one writes its output
to `engineered_features.csv` / `test_engineered_features.csv` in this
directory.

**That output is generated, not committed.** Run the notebook locally to
produce it — don't commit the CSV. It's a few hundred thousand cells of
derived data, it changes every time the pipeline changes, and if two people
commit a same-named file from different versions of the pipeline there's no
way to tell them apart. `.gitignore` already excludes it.

If you're adding a model notebook, point it at `engineered_features.csv`
after running this pipeline — see `../models/README.md`.

### A note on feature engineering

Earlier work explored adding derived columns on top of this (date features,
`lane`/`carrier_service`, dimensional-weight/density features, interaction
terms — see `exploration/`). An ablation test (CatBoost, point regression on
`transit_days`) found these did not improve on the base columns, likely
because tree-based models already capture that kind of interaction by
splitting on the underlying columns. They were left out of the canonical
pipeline for that reason. This was only tested for tree models; it's an open
question for linear/other model families, so the code is kept in
`exploration/` rather than deleted.

## `exploration/`

Per-column-group data quality findings and fixes. See
`exploration/README.md`.
