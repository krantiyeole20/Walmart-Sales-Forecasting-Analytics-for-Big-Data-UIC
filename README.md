# Walmart Sales Forecasting — Analytics for Big Data

> Forecasting weekly sales across 45 Walmart stores using **Apache Spark (PySpark)** for large‑scale data processing and a suite of regression models for prediction — achieving **~98% accuracy** with gradient‑boosted trees.

This project tackles a classic retail problem: accurately predicting **weekly sales** for Walmart stores and departments. Sales are driven by seasonality, holidays, store size, regional economics (CPI, unemployment, fuel prices), and promotional markdowns. Forecasting them well is hard but valuable — better forecasts mean better inventory, staffing, and financial planning. This repo builds an end‑to‑end pipeline that ingests multiple raw datasets with Spark, explores them, engineers features, and benchmarks four machine‑learning models against the task.

---

## 📊 The Data

The work is built on the [Walmart Recruiting — Store Sales Forecasting](https://www.kaggle.com/competitions/walmart-recruiting-store-sales-forecasting) dataset, spanning **February 2010 → October 2012** across **45 stores** (types A, B, and C, distinguished by size). Three raw files are joined into a single modeling table:

| File | Contents |
|------|----------|
| `train.csv` | Weekly sales per `Store` × `Dept` × `Date`, with a holiday flag |
| `stores.csv` | Store metadata — `Type` (A/B/C) and `Size` |
| `features.csv` | Regional & promotional signals — `Temperature`, `Fuel_Price`, `CPI`, `Unemployment`, and five `MarkDown` columns |

---

## 🛠️ Tech Stack

- **Apache Spark 3.5.1 (PySpark)** on Hadoop 3 — distributed loading, joining, and cleaning of the raw CSVs
- **Java 8** runtime + **findspark** for the Spark session
- **pandas / NumPy** — feature engineering and in‑memory analytics
- **Matplotlib / Seaborn** — exploratory visualizations
- **scikit‑learn** — Linear Regression, Random Forest, Decision Tree (+ GridSearchCV)
- **XGBoost** — gradient‑boosted regression (the top performer)
- Developed and run in **Google Colab**

---

## 🔍 What the Project Does

### 1. Distributed Ingestion & Cleaning (PySpark)
Each CSV is read into a Spark DataFrame, inspected for row counts and date ranges, then joined: `train ⋈ stores ⋈ features`. The five `MarkDown` columns carry `"NA"` strings that are converted to true nulls before the joined dataset is materialized to pandas for analysis.

### 2. Exploratory Data Analysis
A rich set of visualizations surfaces the structure of the data, including:
- **Seasonality** — weekly sales over the full timeline, with the dramatic November–December holiday spikes
- **Temporal breakdowns** — sales by month, quarter, and year (box plots)
- **Holiday effect** — holiday vs. non‑holiday sales comparison
- **Store types** — how A/B/C stores differ in size and sales
- **Economic drivers** — sales plotted against Temperature, Fuel Price (and its rate of change), CPI, and Unemployment
- **A correlation heatmap** across all numeric features

### 3. Outlier Handling & Feature Engineering
- Negative sales and extreme outliers (`Weekly_Sales > 200,000`) are removed
- Calendar features (`Day`, `Month`, `Quarter`, `Year`, `Week`) are derived from the date index
- `IsHoliday` boolean → integer; store `Type` is **ordinally encoded by size** (A > B > C)
- Missing values are imputed, producing a clean fully‑numeric feature matrix

### 4. Modeling & Benchmarking
Four regression models are trained on an 80/20 split and compared head‑to‑head:

| Model | Accuracy (R² score) | MAE | RMSE |
|-------|:------:|:---:|:----:|
| **XGBoost Regressor** | **98.35%** | **1,570.7** | 2,852.1 |
| **Random Forest** | 98.02% | 1,336.0 | 3,121.8 |
| Decision Tree (GridSearchCV) | 87.82% | — | 7,744.6 |
| Linear Regression | 8.83% | 14,485.7 | 21,188.2 |

**XGBoost wins**, with Random Forest a close second. Linear Regression performs poorly — strong evidence that the sales relationships are highly **non‑linear**, exactly where tree‑based ensembles shine.

### 5. Interpretation
The XGBoost feature‑importance ranking shows the strongest drivers of weekly sales:

| Rank | Feature | Importance |
|:----:|---------|:----------:|
| 1 | Size | 0.350 |
| 2 | Dept | 0.215 |
| 3 | Type | 0.147 |
| 4 | Store | 0.099 |
| 5 | CPI | 0.038 |
| 6 | Week | 0.031 |
| 7 | IsHoliday | 0.025 |

Store **size**, **department**, and **type** dominate — physical and structural store attributes matter far more than weekly economic fluctuations. Validation predictions (May–Oct 2012) track actual sales closely, and a residual plot confirms errors stay tight around zero.

---

## 📁 Repository Contents

| File | Description |
|------|-------------|
| `BigData_Project.ipynb` | The complete pipeline — Spark ingestion, EDA, feature engineering, modeling, and evaluation |
| `IDS561 Project Report.pdf` | Written project report |
| `bigdata project.pdf` | Project presentation / slides |

---

## 🚀 Running It Yourself

1. Open `BigData_Project.ipynb` in **Google Colab** (recommended) or a local Jupyter environment.
2. Download the dataset (`train.csv`, `stores.csv`, `features.csv`) from the [Kaggle competition](https://www.kaggle.com/competitions/walmart-recruiting-store-sales-forecasting) and point the notebook's paths at them.
3. The early cells install Spark 3.5.1, Java 8, `findspark`, and `xgboost` — run them top to bottom.
4. Execute the notebook in order to reproduce the ingestion, analysis, and model benchmarks.

---

## 🔮 Future Work

- **Time‑series models** — incorporate ARIMA/SARIMA or Prophet to model seasonality explicitly, and sequence models (LSTM/Temporal Fusion Transformer) for longer horizons.
- **Per‑department / per‑store forecasting** — train hierarchical or grouped models rather than a single global regressor, since `Dept` and `Store` are such strong signals.
- **Markdown signal recovery** — instead of zero‑filling the promotional `MarkDown` columns, model their availability and impact directly.
- **Hyperparameter optimization at scale** — move tuning onto Spark MLlib or a distributed search (Optuna / Ray Tune) to exploit the full cluster.
- **Productionization** — wrap the pipeline as a scheduled Spark job with a model registry and a lightweight forecasting dashboard.
- **Richer features** — add lag/rolling‑window sales features, holiday‑proximity indicators, and external economic data.

---

## 📝 Conclusion

This project demonstrates a complete big‑data workflow — from **distributed ingestion with Apache Spark** through **exploratory analysis, feature engineering, and comparative modeling** — to forecast Walmart weekly sales with high accuracy. The standout result is that **gradient‑boosted trees (XGBoost) reach ~98% R²**, dramatically outperforming linear methods and confirming the non‑linear, structurally driven nature of retail sales. Beyond the headline accuracy, the analysis offers interpretable insight: store size, department, and type are the dominant levers on revenue.

---

*This work was completed as a course project for **IDS 561 — Analytics for Big Data** at the **University of Illinois Chicago (UIC)**.*

**Author:** Kranti Yeole
