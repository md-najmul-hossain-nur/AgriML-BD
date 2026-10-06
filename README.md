<div align="center">

# 🌾 AgriML-BD

### Machine Learning for Bangladesh Agriculture

Crop recommendation and production estimation using agricultural, soil, and weather data.

[🌐 **View Live App**](https://agri-ml-bd.vercel.app/)

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Web%20App-Flask-000000?logo=flask&logoColor=white)
![scikit-learn](https://img.shields.io/badge/ML-scikit--learn-F7931E?logo=scikitlearn&logoColor=white)

</div>

---

## 🌱 About the Project

AgriML-BD is a machine-learning project built around agricultural data from Bangladesh. It provides:

| Capability | Description |
|---|---|
| 🌿 **Crop recommendation** | Ranks suitable crops based on soil, weather, district, and season data. |
| 📈 **Production estimation** | Estimates crop production using the selected crop, cultivated area, soil, and weather inputs. |

The repository includes data preparation, augmentation, model comparison, and a Flask web application. Predictions are estimates for decision support—not guaranteed agricultural advice.

## 👥 Student Information

| Student Name | Student ID |
|---|---:|
| Md. Najmul Hossain Nur | `0112230536` |
| Asif Mustoba Sazzad | `0112230236` |
| Md. Yusuf Siyam | `0112230545` |
| Md. Nazibullah | `011221448` |

## 🧠 Models

### Experiment comparison

[`master_comparison.py`](./master_comparison.py) compares these models:

- **Random Forest**
- **HistGradientBoosting**
- **scikit-learn MLP** — called DNN in the experiment results, but implemented with `MLPRegressor` and `MLPClassifier` (not TensorFlow or PyTorch).

### Models used by the web app

| Task | Model |
|---|---|
| Production estimation | HistGradientBoosting Regressor |
| Crop recommendation | Random Forest Classifier |

> The Flask app trains or loads its own models; it does not load the best models from the comparison experiments.

### Recorded highlights

| Task | Best recorded model and pipeline | Result |
|---|---|---:|
| Production estimation | HistGradientBoosting · merged data | **R² = 0.9455** |
| Crop recommendation | MLP · merged data | **Accuracy = 94.03%** |

These results are recorded in [`RESULTS.txt`](./RESULTS.txt). Production regression metrics use the `log1p(Production)` target, and results may vary with data and experiment setup.

## 🗂️ Project Structure

```text
AgriML-BD/
├── Data/
│   ├── dataset/                    # Source datasets
│   └── Marge/
│       ├── Marge.py                # Merge and prepare source data
│       └── merged_dataset.csv      # Combined dataset
├── Preprocessing/
│   └── Preprocesse-data.py         # Prepare original-data splits
├── Augmentation/
│   ├── augment_data.py             # Generate synthetic examples
│   └── augmented_preprocess.py     # Prepare augmented-data splits
├── Output/                         # Model comparison charts
├── static/                         # Web app images and logos
├── app.py                          # Flask application
├── master_comparison.py            # Train and compare models
└── requirements.txt                # Python dependencies
```

## 📊 Data

Place the source CSV files in `Data/dataset/`:

| Dataset | Purpose |
|---|---|
| `SPAS-Dataset-BD.csv` | Main agricultural records |
| `65 Years of Weather Data Bangladesh (1948 - 2013).csv` | Historical rainfall by weather station |
| `Crop_recommendation.csv` | Crop recommendation data with N, P, K, and pH |

The merge script uses SPAS as its base and creates `Data/Marge/merged_dataset.csv`. Rainfall is added using mapped stations' historical averages; N/P/K/pH values are added using crop-class averages. These are **proxies**, not measurements from individual farms.

## 🚀 Setup & Run

Run the commands from the repository root. Python 3.10 or newer is recommended.

### 1. Install dependencies

```powershell
python -m pip install -r requirements.txt
python -m pip install matplotlib
```

### 2. Generate data and compare models

```powershell
python Data/Marge/Marge.py
python Preprocessing/Preprocesse-data.py
python Augmentation/augment_data.py
python Augmentation/augmented_preprocess.py
python master_comparison.py
```

### 3. Launch the web application

```powershell
python app.py
```

Then open **[http://127.0.0.1:8080](http://127.0.0.1:8080)** in your browser.

> **Before running:** Put the three source CSV files in `Data/dataset/`. The app needs `Data/Marge/merged_dataset.csv`; model comparison also needs the preprocessing and augmentation outputs generated above.

## ⚠️ Important Notes

- Augmented rows are synthetic and do not represent new real-world observations.
- Soil nutrients and rainfall in the merged dataset are averages used as proxies.
- The source production unit is not converted; confirm it before interpreting the app's estimate as tons.

---

<div align="center">

**Built with data, machine learning, and a focus on Bangladesh agriculture.**

</div>
