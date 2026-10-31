# Delivery Time Lab

**Application:** [Open Delivery Time Lab](https://subhashdixit.github.io/food-delivery-time-predictor/) · [Source](https://github.com/subhashdixit/food-delivery-time-predictor)

Estimate historical delivery duration and explore how traffic, weather and distance relate to it. A complete **HTML/CSS/JavaScript application hosted on GitHub Pages**, with real offline Python training and a verified numeric model export. Inference runs in the visitor's browser, including CSV batches; there is no paid prediction API or model server.

## Problem and scope

The raw coordinates are converted to great-circle distance, with zero/unsupported coordinates treated as missing. Distances above 300 km fall outside this study and are imputed rather than silently clipped. The form asks for kilometres directly. Courier IDs define separate split groups; they are not predictors. Feature availability at order time is not independently established, so this is a retrospective duration study rather than an operational booking-time ETA.

Great-circle distance is an approximation, not a routing ETA. The split separates delivery-person IDs. The dataset does not establish current conditions, restaurant preparation delays or real-time travel.

The original repository was `Food_Delivery_Time_Prediction_End_to_End_ML_Project`. Existing notebooks, Python applications and model files are preserved for context; previous documentation is in `archive/`. The published `site/` uses the documented new model, not an implicitly trusted legacy pickle.

## Use the app

1. Open the GitHub Pages address linked from the repository About section after deployment.
2. Review the synthetic starting profile. Defaults combine training medians and common categories; they do not identify a source record.
3. Change fields and calculate the model result. Blank values use the frozen missing-data representation.
4. Compare the estimate with the held-out error scale; do not interpret MAE as an individual confidence interval.
5. Download the sample CSV, retain all feature headers and upload up to **500 rows / 2 MiB**. The browser worker computes the output locally.
6. Download predictions. The output contains row numbers and results rather than echoing your financial, assessment or sensor inputs.
7. Read the model-selection chart, test results and full machine-readable evaluation in the Evidence section.



**Privacy:** this app has no inference/upload endpoint, analytics script or browser-persistent input store. The static host receives normal asset requests/connection metadata. The local batch worker is terminated after completion or timeout. Reset restores the synthetic form; it does not promise forensic erasure of browser memory. Inputs are validated, not interpreted as executable code.

## Data and feature contract

- Source file: `Data/finalTrain.csv`.
- Original source rows: **45,584**; valid-target rows used: **45,584**.
- Source SHA-256: `e33e182ebb9c1d7e0f54c83c0cef6ddc5aa98d203e8888d33ab68cca3a079cba`.
- Selected features: **11**. [Full names, units, categories and bounds](docs/feature-contract.md).
- Attribution/provenance: Original repository dataset; upstream annotation/collection provenance is not independently established.

Source numeric values outside the documented serving contract are counted and treated as missing in training, rather than being silently converted to valid measurements. The counts are recorded in [reports/metrics.json](reports/metrics.json). Unsupported targets are excluded. Categories are normalized using the documented source adapter.

## Architecture: training and serving are separate

```mermaid
flowchart LR
 A[Original source and identity hash] --> B[Normalize selected features]
 B --> C[Grouped train / validation / test split]
 C --> D[Fit preprocessing on training only]
 D --> E[Train linear and forest candidates]
 E --> F[Validation selection]
 F --> G[Held-out evaluation]
 F --> H[Export numeric JSON]
 H --> I[Node verifies Python / JavaScript parity]
 I --> J[GitHub Pages publishes static assets]
 J --> K[Browser form and local CSV worker]
```

GitHub Pages serves static files; it does not run Python. Training is an offline developer operation. The export includes ordered features, medians, means/scales, category vocabulary and model parameters. Browser JavaScript implements the same transformation and prediction. It checks the JSON artifact SHA-256 against the evaluation manifest before enabling prediction.

A hash catches accidental mismatch; it is not independent protection against a malicious change to both files. Source reviews, branch protection and CI provide complementary controls. The app never accepts uploaded model files or executes pickle objects.

## Concepts and algorithms

**Missing values:** numeric medians are learned from training rows. A median is robust to outliers but ignores relationships between fields. Missing categories have an explicit token. All-empty profiles are rejected.

**Encoding:** numeric values become `(x − training_mean) / training_scale` after imputation. A zero-variance scale becomes 1. Categories use a training-only one-hot vocabulary, not an invented numeric ordering. Unsupported categories are rejected by serving; a new source category needs a reviewed adapter/model update.

**Linear candidate:** ridge regression learns a weighted sum and intercept while penalizing large coefficients with an L2 term. The objective is squared error plus `alpha × sum(weights²)`, with alpha 10 in this experiment. It is compact and extrapolates linearly; nonlinear interactions may remain unexplained.

**Random forest candidate:** 64 decision trees learn threshold splits and average their numeric leaf predictions. Depth and leaf-size limits reduce complexity. Forests capture nonlinear interactions without a neural network, but use larger artifacts, can overfit a small dataset and generally cannot extrapolate reliably outside observed target ranges. 

The serving tree implementation reproduces scikit-learn's float32 split inputs with `Math.fround`. Linear inference uses ordinary double-precision arithmetic. Node parity fixtures compare JavaScript outputs with real held-out Python predictions to an absolute tolerance of `1e-7`. They are arithmetic checks, not extra accuracy measurements.

## Split, model selection and results

GroupShuffleSplit; distinct courier IDs. Fixed seeds **42 and 43** produce training / validation / test rows **27,387 / 9,006 / 9,191**. Grouping limits the documented form of leakage; this is not time-separated or externally validated deployment performance.

Preprocessing and both candidates fit **training rows only**. Validation MAE selects **random_forest**. The selected model is not refitted on test data. The held-out set supplies the final reported metrics.

| Validation candidate | MAE: lower is better |
|---|---:|
| linear | 4.8281 |
| random_forest | 3.5314 |

| Held-out test metric | Selected model | Training-only baseline |
|---|---:|---:|
| MAE | 3.4686 | 7.6580 |
| RMSE | 4.4163 | 9.4795 |
| R² | 0.7829 | -0.0004 |

Errors use **minutes**. The mean-prediction baseline is fitted on training labels only. A test-set MAE is not a calibrated individual prediction interval.

**Reading the metrics:** MAE is the average absolute error in target units; RMSE penalizes large misses more strongly; R² compares squared errors with predicting the test mean and can be negative. A good R² is not evidence of causality or current population validity. Compare the baseline and examine domain coverage before interpreting a result.



Full measurements and integrity metadata are in [reports/metrics.json](reports/metrics.json). No synthetic performance figures are used.

## Run locally

Use Node.js **22+** for JavaScript development/checks and Python **3.10 or 3.11** for training and the simple local web server.

```bash
git clone https://github.com/subhashdixit/food-delivery-time-predictor.git
cd food-delivery-time-predictor
npm ci
npm test
npm run lint
npm run format
python3 -m http.server 8000 --directory site
```

Open `http://127.0.0.1:8000`. Serve over HTTP rather than opening `index.html` with `file://`, because model fetching, modules and workers need an appropriate origin. Production uses HTTPS on Pages. No API key, Docker container or backend account is needed to use this browser app.

## Reproduce training

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-training.txt
python scripts/train.py
python scripts/check_release.py
python -m black --check scripts
python -m pylint scripts
npm test
```

On Windows, activate `.venv\Scripts\activate` instead. Training reads the included source file, reconstructs the split and writes `site/model.json`, `site/sample.json`, `site/metrics.json` and `reports/`. Source data is not fetched live by an end user. Review source rights, model results and hash changes before committing a retrained release.

The two insurance apps deliberately share source/training design; their results must not be counted as independent experiments. Other legacy preprocessing/model objects are preserved but are not silently mixed with this release.

## Project structure

```text
site/index.html          accessible HTML interface
site/style.css           responsive local styling
site/app.js              form, results, plots and batch orchestration
site/model-engine.js     validated framework-free inference and CSV parsing
site/model-worker.js     bounded local batch work
site/model.json          frozen numeric model; no executable pickle
site/metrics.json        evaluation and source/artifact hashes
site/project.json        purpose, limitations and view configuration
scripts/train.py         reproducible project-specific source adapter/training
scripts/check_release.py source/model identity checks
tests/model.test.js      real-model parity, validation, CSV and metadata checks
reports/                 full metrics and held-out parity fixtures
docs/feature-contract.md names, units, categories and supported bounds
archive/                 preserved previous documentation
.github/workflows/       test and GitHub Pages release pipeline
```

## CI/CD and GitHub Pages

```mermaid
flowchart LR
 A[Feature branch and PR] --> B[Black + Pylint + ESLint + Prettier]
 B --> C[Source hashes + Node behavior/parity tests]
 C --> D[Review and merge main]
 D --> E[Repeat checks; upload only site folder]
 E --> F[GitHub Pages HTTPS deployment]
 F --> G[Check form, CSV and published model identity]
```

**CI** repeats code and behavior checks for changes, including a real Python/JavaScript model comparison. It catches regressions and packaging mistakes; it does not certify the statistical validity of the dataset. **CD** publishes the checked static artifact from `main`. PRs run checks without publishing. Actions are pinned to reviewed commit hashes, and only the deployment job gets Pages publishing permissions.

Enable **Settings → Pages → Source → GitHub Actions**. The workflow `.github/workflows/pages.yml` builds/tests and uploads `site/`; it does not publish legacy datasets, Python environments or reports as website assets. The repository About/README link points to the verified live app after rollout.

**Hosting** supplies the public asset server; **deployment** replaces its files with a particular revision. GitHub Pages needs no server process, paid inference quota or container here. Docker can be useful for a Python training environment, but is unnecessary for serving these static browser files.

**Versioning:** Git records code, configuration, model weights and evaluation together. Tag known-good releases. Update preprocessing and the exported model as one reviewed change. The model's schema version and report hashes identify the inference contract.

**Rollback:** revert a faulty change through a PR and let the same workflow publish the previous compatible artifacts. Restore model, schema and metrics together, then repeat a synthetic form/CSV check. Deleting a review branch does not delete its merged changes. Main is the maintained branch, with merged branches removed automatically.

**Retraining versus redeployment:** retraining changes learned parameters and needs a new evaluation review; redeploying unchanged files does not create a new model. Useful future work includes time-separated validation, subgroup/coverage analysis, better source documentation and calibrated uncertainty. Those are not claimed as completed.

References: [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages), [scikit-learn evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html), [common leakage pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).
