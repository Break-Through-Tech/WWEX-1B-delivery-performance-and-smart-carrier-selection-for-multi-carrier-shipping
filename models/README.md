# `models/`

Model training and evaluation notebooks live here — one notebook per model
(or closely related group of models, e.g. a point-vs-quantile comparison).

## Convention

- **Read from `../notebooks/engineered_features.csv`.** Generate it first by
  running `notebooks/clean_data.ipynb` locally (see `notebooks/README.md` —
  that file is gitignored, not committed). Don't re-derive your own cleaned
  dataset in a model notebook; every model should be trained on the same
  pipeline output so results are comparable.
- **Target column is `transit_days`** unless the notebook says otherwise.
- **Report a baseline.** At minimum, MAE against `promised_days` (i.e. "how
  good is just trusting the promised delivery date"), so a model's MAE means
  something on its own.
- **Use a held-out split that isn't reused for tuning.** If you tune
  hyperparameters, do it against a validation split and report final numbers
  on a separate test split you haven't touched.
- **Name the file after the model**, e.g. `catboost_transit.ipynb`,
  `lgb_transit.ipynb`.
