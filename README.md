# AgriML-BD

AgriML-BD is a machine-learning project for Bangladesh agriculture. It combines crop, district, season, soil-proxy, and weather data to:

- **Recommend crops** as a classification task, returning the three highest-ranked crop classes.
- **Estimate crop production** as a regression task using the selected crop, cultivated area, soil, and climate inputs.

The repository contains the data preparation and augmentation pipelines, model comparison experiments, and a Flask web application. Predictions are experimental decision-support estimates, not guaranteed agricultural advice.

## Contents

- [Project workflow](#project-workflow)
- [Repository structure](#repository-structure)
- [Data](#data)
- [Requirements](#requirements)
- [Run the project](#run-the-project)
- [Model comparison](#model-comparison)
- [Web application and API](#web-application-and-api)
- [Important limitations](#important-limitations)

## Project workflow

```text
Source CSV files
      |
      v
Data/Marge/Marge.py
  Clean and merge agriculture, weather, and crop-recommendation sources
      |
      +-------------------------------+
      |                               |
      v                               v
Preprocessing/                  Augmentation/
Preprocesse-data.py             augment_data.py
      |                               |
      |                               v
      |                       augmented_preprocess.py
      |                               |
      +---------------+---------------+
                      v
             master_comparison.py
             RF / Gradient Boosting / MLP
                      |
                      v
                 Output/*.png

Data/Marge/merged_dataset.csv ---> app.py (Flask predictions)
```

The preprocessing scripts create separate regression and classification datasets. The regression target is `Production` after a `log1p` transformation; the classification target is `Crop Name`. The model-comparison script evaluates three data pipelines (merged, preprocessed, and augmented) with Random Forest, HistGradientBoosting, and scikit-learn MLP models.

## Repository structure

```text
.
├── Data/
│   ├── dataset/                         # Three input CSV datasets
│   └── Marge/
│       ├── Marge.py                     # Clean, map, and merge source data
│       └── merged_dataset.csv           # 4,607 rows × 20 columns
├── Preprocessing/
│   ├── Preprocesse-data.py              # Clean, encode, scale, and split
│   └── Data/                            # Original-data splits and encodings
├── Augmentation/
│   ├── augment_data.py                  # Generate synthetic training examples
│   ├── augmented_preprocess.py          # Group-split and scale augmented data
│   └── Data/                            # Augmented data, splits, and plots
├── Output/                              # Model comparison plots
├── static/                              # Flask app images and logos
├── api/
│   └── index.py                         # WSGI entry point importing app
├── app.py                               # Flask UI and /predict endpoint
├── master_comparison.py                 # Nine model/pipeline comparisons
├── RESULTS.txt                          # Saved experiment metrics and notes
├── HOW_TO_RUN.txt                       # Short run guide
└── requirements.txt                     # Core Python dependencies
```

## Data

The source CSV files are expected under `Data/dataset/`:

| File | Use |
|---|---|
| `SPAS-Dataset-BD.csv` | Main agricultural records, including crop, district, season, area, production, temperature, and humidity |
| `65 Years of Weather Data Bangladesh (1948 - 2013).csv` | Historical rainfall observations by weather station |
| `Crop_recommendation.csv` | Crop recommendation records with N, P, K, and pH values |

`Data/Marge/Marge.py` uses the SPAS data as its base table. It removes invalid crop records, fills missing season/AP Ratio values, maps districts to weather stations to add station-average rainfall, and maps crop names to crop-class average N/P/K/pH values. The result is `Data/Marge/merged_dataset.csv` (4,607 rows × 20 columns in the checked-in dataset).

The merged data includes:

- Crop, district, and season
- Area and production
- Soil-proxy values: N, P, K, and pH
- Temperature, humidity, and rainfall
- Additional source fields such as AP Ratio, Transplant, Growth, and Harvest

## Requirements

- Python 3.10 or newer
- Dependencies in `requirements.txt`
- Matplotlib for the preprocessing/augmentation/model-comparison charts (it is used by the scripts but is not currently listed in `requirements.txt`)

Install from the repository root:

```powershell
python -m pip install -r requirements.txt
python -m pip install matplotlib
```

## Run the project

Run commands from the **repository root**, because the data scripts use repository-relative paths. The source CSV files must be present in `Data/dataset/`.

```powershell
# 1. Merge the source datasets
python Data/Marge/Marge.py

# 2. Prepare the original-data train/validation/test sets
python Preprocessing/Preprocesse-data.py

# 3. Generate an augmented dataset
python Augmentation/augment_data.py

# 4. Prepare train/validation/test sets from augmented data
python Augmentation/augmented_preprocess.py

# 5. Train and compare the models; generate charts in Output/
python master_comparison.py

# 6. Start the web app
python app.py
```

Open [http://127.0.0.1:8080](http://127.0.0.1:8080) after starting the app.

Steps 1–4 create the generated files consumed by the comparison script and Flask app. Some generated CSVs and model artifacts are ignored by Git, so rerun the corresponding pipeline step if they are missing.

## Model comparison

`master_comparison.py` compares Random Forest (RF), HistGradientBoosting (GB), and MLP (reported as DNN) on three pipelines. The values below are the recorded test results in [`RESULTS.txt`](./RESULTS.txt); they are not a guarantee of performance on new farms or future seasons.

| Data pipeline | Model | Production R² | Production RMSE | Production MAE | Crop accuracy | Crop F1 |
|---|---:|---:|---:|---:|---:|---:|
| Merged | RF | 0.9376 | 0.6075 | 0.4203 | 0.9020 | 0.9019 |
| Merged | GB | **0.9455** | **0.5679** | **0.3866** | 0.9032 | 0.9023 |
| Merged | MLP/DNN | 0.8261 | 1.0142 | 0.7434 | **0.9403** | **0.9331** |
| Preprocessed | RF | 0.8921 | 0.6101 | 0.4407 | 0.8941 | 0.8931 |
| Preprocessed | GB | 0.9067 | 0.5674 | 0.4171 | 0.8914 | 0.8903 |
| Preprocessed | MLP/DNN | 0.8562 | 0.7041 | 0.4828 | **0.9236** | **0.9060** |
| Augmented | RF | 0.9082 | 0.6986 | 0.4516 | 0.9199 | 0.9196 |
| Augmented | GB | 0.8979 | 0.7366 | 0.5090 | 0.9179 | 0.9181 |
| Augmented | MLP/DNN | 0.8804 | 0.7974 | 0.5534 | **0.9223** | 0.9184 |

In the recorded comparison, merged-data GB has the highest yield R² and lowest yield RMSE/MAE; merged-data MLP has the highest crop accuracy and F1. The charts are written to `Output/merged_dataset_comparison.png`, `Output/preprocessed_dataset_comparison.png`, `Output/augmented_dataset_comparison.png`, and `Output/master_comparison.png`.

Regression metrics are calculated against the `log1p(Production)` target, so the reported RMSE/MAE values are on the transformed scale rather than the source production unit.

The Flask app does **not** load the comparison script's best-performing models. It trains or loads its own bundle: a HistGradientBoosting regressor for production and a RandomForest classifier for crop ranking.

## Web application and API

The Flask interface is served at `/`. On startup, `app.py` loads `app_models.pkl` if present; otherwise it trains its app-specific models from `Data/Marge/merged_dataset.csv` and attempts to save the bundle. To create a usable bundle, the merged dataset must exist.

The JSON endpoint `POST /predict` expects the following fields:

```json
{
  "district": "Dhaka",
  "season": "Rabi",
  "crop": "Aman",
  "N": 90,
  "P": 42,
  "K": 43,
  "ph": 6.5,
  "area": 1.0,
  "avgTemp": 25,
  "minTemp": 18,
  "maxTemp": 32,
  "avgHum": 70,
  "minHum": 50,
  "maxHum": 90,
  "rainfall": 150
}
```

Use district, season, and crop labels available in the merged dataset. The endpoint returns `recommended_crop`, `top3`, `confidence`, and `yield_tons`. The production model predicts on a log-transformed target and converts the result back with `expm1`.

Example request:

```powershell
$body = @{
  district = "Dhaka"; season = "Rabi"; crop = "Aman"
  N = 90; P = 42; K = 43; ph = 6.5; area = 1.0
  avgTemp = 25; minTemp = 18; maxTemp = 32
  avgHum = 70; minHum = 50; maxHum = 90; rainfall = 150
} | ConvertTo-Json

Invoke-RestMethod -Uri http://127.0.0.1:8080/predict `
  -Method Post -ContentType "application/json" -Body $body
```

## Important limitations

- **Soil values are proxies:** N/P/K/pH are crop-class averages from the crop-recommendation dataset, not measurements from the user's field.
- **Rainfall is a proxy:** the merge uses a mapped weather station's historical average; it is not necessarily rainfall for the relevant district, year, or season.
- **Augmented rows are synthetic:** noise and neighbor-based interpolation increase the dataset size but do not add real agricultural observations. Augmented evaluation is not equivalent to testing only on untouched real-world observations.
- **Prediction units need confirmation:** the app labels the estimate `yield_tons`, but the merge pipeline does not convert the source `Production` unit. Confirm the source unit before interpreting the displayed value as tons.
- **Confidence is not calibrated:** the app derives its displayed confidence from a fixed range and the gap between classifier probabilities. It should not be interpreted as a validated probability of correctness or as the test accuracy.
- **Metrics are experiment-specific:** results depend on the available dataset, preprocessing, random split, and saved code/results. Re-running the pipeline can produce different metrics.
