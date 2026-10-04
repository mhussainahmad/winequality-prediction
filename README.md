# Wine Quality Prediction

An end-to-end practice MLOps project from 2022: a DVC pipeline that trains an ElasticNet regressor on the red wine quality dataset, a Flask app that serves predictions through an HTML form and a JSON endpoint, and a GitHub Actions workflow that lints, tests and deploys to Heroku.

## Overview

The project predicts a wine's quality score (the `TARGET` column, integers 3 to 8 in this dataset) from 11 physicochemical measurements such as fixed acidity, residual sugar, pH and alcohol.

- **Pipeline (DVC).** `dvc.yaml` defines three stages, all configured through `params.yaml`:
  1. `load_data` - reads `data_given/winequality.csv`, replaces spaces in column names with underscores, and writes `data/raw/winequality.csv`.
  2. `split_data` - 80/20 train/test split (`random_state: 42`) into `data/processed/`.
  3. `train_and_evaluate` - fits `sklearn.linear_model.ElasticNet` with `alpha` and `l1_ratio` from `params.yaml`, writes RMSE/MAE/R2 to `report/scores.json` and `report/params.json`, and saves the model to `saved_models/model.joblib`.
- **Serving (Flask).** `app.py` loads the model from `prediction_service/model/model.joblib`. Inputs are validated against per-feature min/max ranges in `prediction_service/schema_in.json`; predictions outside 3 to 8 are reported as "Unexpected result".
- **CI/CD.** `.github/workflows/ci-cd.yaml` runs flake8 and pytest on Python 3.10 for pushes and pull requests to `main`, then pushes to a Heroku app named by repository secrets. `Procfile` starts the app with gunicorn.

## Results

From `report/scores.json` (ElasticNet, `alpha=0.9`, `l1_ratio=0.4`, evaluated on the 20% test split):

| Metric | Value |
|---|---|
| RMSE | 0.803 |
| MAE | 0.655 |
| R2 | 0.013 |

These numbers reproduce when the pipeline is rerun. The R2 is close to zero: with this much regularization the model predicts little more than the mean quality. The project's focus was the pipeline, serving and deployment workflow rather than model performance; the hyperparameters in `params.yaml` are the place to start tuning.

## Repository layout

```
.
├── app.py                      Flask app (form at "/", JSON POST to "/")
├── dvc.yaml / dvc.lock         DVC pipeline definition and lock file
├── params.yaml                 paths, split settings, ElasticNet hyperparameters
├── src/
│   ├── get_data.py             config loading and CSV reading
│   ├── load_data.py            stage 1
│   ├── split_data.py           stage 2
│   └── train_and_evaluate.py   stage 3
├── prediction_service/
│   ├── prediction.py           input validation and prediction
│   ├── schema_in.json          allowed range for each input feature
│   └── model/model.joblib      model used by the web app
├── webapp/                     HTML templates, CSS and JS
├── report/                     scores.json and params.json from the last run
├── tests/test_config.py        pytest tests for the prediction service
├── data_given/                 DVC-tracked source CSV (winequality.csv.dvc)
├── data_given-...-001.zip      zipped copy of the source CSV
├── template.py                 script used to scaffold the folders
├── tox.ini, setup.py           tox config and package metadata
└── Procfile                    Heroku/gunicorn entry point
```

## Getting started

```bash
git clone https://github.com/mhussainahmad/winequality-prediction.git
cd winequality-prediction
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

`requirements.txt` lists the deprecated `sklearn` alias alongside `scikit-learn`; current versions of pip refuse to install `sklearn`, so delete that line if the install fails.

No DVC remote is configured, so the source CSV is restored from the zip in the repository:

```bash
unzip data_given-20220714T175745Z-001.zip   # creates data_given/winequality.csv
```

Run the pipeline:

```bash
dvc repro          # or run the three stages directly:
python src/load_data.py --config=params.yaml
python src/split_data.py --config=params.yaml
python src/train_and_evaluate.py --config=params.yaml
dvc metrics show   # prints report/scores.json and report/params.json
```

The pipeline writes the model to `saved_models/model.joblib`. The web app reads `prediction_service/model/model.joblib`, so copy a newly trained model there to serve it.

Start the web app (http://localhost:5000):

```bash
python app.py
```

Or call the JSON endpoint:

```bash
curl -X POST http://localhost:5000/ -H "Content-Type: application/json" -d '{
  "fixed_acidity": 7.4, "volatile_acidity": 0.7, "citric_acid": 0.0,
  "residual_sugar": 1.9, "chlorides": 0.076, "free_sulfur_dioxide": 11.0,
  "total_sulfur_dioxide": 34.0, "density": 0.9978, "pH": 3.51,
  "sulphates": 0.56, "alcohol": 9.4}'
```

Run the tests from the repository root:

```bash
pytest -v      # or: tox
```

## Known issues

- `test_api_response_correct_range` fails with a `KeyError`: the API returns `{"Response": ...}`, but the test reads `res["response"]`. The other four tests pass.
- Build and IDE artifacts (`build/`, `dist/`, `src.egg-info/`, `.tox/`, `.idea/`) are committed to the repository.

## Tech stack

Python, pandas, scikit-learn, DVC, Flask, gunicorn, pytest, tox, flake8, GitHub Actions, Heroku.

## Data and credits

- The dataset is the red wine quality data from the UCI Machine Learning Repository (Cortez et al., 2009, "Modeling wine preferences by data mining from physicochemical properties"), with 1,599 samples; the quality column is renamed `TARGET`.
- The project structure follows a public DVC + Flask MLOps tutorial; the web app's "source code" link points to the reference repository, [c17hawke/dvc-plus-cml-test](https://github.com/c17hawke/dvc-plus-cml-test).

