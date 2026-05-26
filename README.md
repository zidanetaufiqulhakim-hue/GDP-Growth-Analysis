# Achieving 6% GDP Growth by 2027: A Data Science Analysis of Indonesia's Macroeconomic Drivers

**Author:** Zidane Taufiqul Hakim  
**Dataset Coverage:** 1981 – 2023  
**Model:** Ridge Regression with Preprocessing Pipeline  
**Goal:** Identify the strongest GDP growth indicators and forecast pathways toward 6% growth

---

## Abstract

This project investigates the macroeconomic drivers of Indonesia's GDP growth using annual historical data from 1981 to 2023. Using a combination of correlation analysis and regularized predictive modelling, it identifies Gross Capital Formation (GCF) growth and net exports (X–M) as the two strongest levers for accelerating economic growth — significantly outweighing household consumption. A trained Ridge regression pipeline forecasts 5.54% GDP growth in 2026 based on 2025 conditions. Prescriptive scenario analysis further suggests that 6% growth by 2027 is achievable, but only under an aggressive investment and export expansion strategy.

---

## 1. Problem Definition

Indonesia's GDP follows the standard expenditure identity:

```
GDP = C + I + G + (X – M)
```

Where:
- **C** — Household Consumption
- **I** — Investment (Gross Capital Formation)
- **G** — Government Spending
- **(X–M)** — Net Exports

This project addresses two core questions:

1. **Which component** of the GDP formula is the strongest driver of year-on-year growth acceleration?
2. **What macroeconomic targets** must be achieved in 2026 to reach the government's 6% GDP growth goal in 2027?

The analysis is motivated by Indonesia's Finance Ministry target of 6% GDP growth — an ambitious benchmark given the country's long-run average of approximately 4.84%.

### Constraints

- The dataset is small (42 annual observations, 1981–2023), limiting model complexity.
- Raw data contains noise and missing values, making preprocessing the critical bottleneck.
- Simple models are preferred to avoid overfitting; this means non-crisis years drive most of the predictive signal.

---

## 2. Methodology

The project follows a four-stage pipeline: data preparation → correlation analysis → predictive modelling → scenario simulation.

### 2.1 Data Preparation

**Notebook:** `raw_data/data_formatting.ipynb` → `data_cleaning.ipynb`

Raw data from multiple CSV sources is merged into a single annual dataset. The cleaning process handles:

- Missing values via interpolation and regression-based imputation
- Label standardization across sources
- Creation of **lagged features** (lag1, lag2) — using prior-year values to predict current-year GDP growth, which reflects the real-world policy timing constraint

**Output:** `cleaned_gdp_growth_lag1.csv`

### 2.2 Feature Engineering

**Notebook:** `feature_engineering.ipynb`

Derived features are constructed to capture economic signals not directly available in raw form:

| Engineered Feature | Formula / Logic |
|---|---|
| `export_import_ratio` | Exports (US$) / Imports (US$) |
| `real_interest_rate` | Lending rate − Inflation rate |
| `exchange_rate_change_lag1` | Year-on-year IDR/USD change |

After construction, feature selection reduces the full set to eight inputs that show the strongest correlation with future GDP growth.

**Output:** `feature_engineered_gdp_growth_lag1.csv`

### 2.3 Correlation Analysis

**Notebook:** `analysis.ipynb`

Pearson correlation coefficients are computed between each lagged feature and next-year GDP growth. Both raw and engineered feature sets are analyzed to assess how feature construction affects signal clarity.

This stage answers the descriptive and explanatory questions: *which indicators move together with future growth?*

### 2.4 Predictive Modelling

**Notebook:** `modelling.ipynb`

Three models are benchmarked: ordinary linear regression, Lasso, and Ridge. Ridge is selected for the current-year GDP prediction task due to its robustness against coefficient inflation in small, noisy datasets.

The final pipeline architecture:

```
Input (8 lagged features)
    ↓
feature_engineering_pipeline   ← adds export_import_ratio, drops raw trade columns
    ↓
SimpleImputer (strategy='mean') ← handles any missing values in production
    ↓
RobustScaler                   ← robust to outliers from crisis years
    ↓
Ridge(alpha=50, random_state=42)
    ↓
Output: gdp_growth(%)
```

**Saved model:** `gdp_lasso_model.pkl`

### 2.5 Scenario Analysis (Prescriptive)

**Notebook:** `analysis.ipynb`

The trained model is used to run forward simulations. Two scenarios — moderate and aggressive — are defined by adjusting GCF growth and export targets while holding other variables at their 2025 baseline. The model then projects 2027 GDP growth based on each set of 2026 targets.

---

## 3. Results

### 3.1 Historical Overview

Indonesia's average GDP growth from 1981 to 2023 is **4.84%**. Two major disruptions stand out:

- **1998 (Asian Financial Crisis):** GDP growth collapsed to **−13%**, accompanied by a massive Rupiah devaluation (from ~2,400 to ~10,000 IDR/USD) and a sharp contraction in FDI and investment.
- **2020 (COVID-19 Recession):** GDP growth fell to **−2%**, with a notable outlier: foreign investors increased inflows during the pandemic, likely viewing the downturn as a buying opportunity.

GDP growth was generally higher during the New Order Era (pre-1998) than in the Reformation Era, despite household consumption remaining relatively stable across both periods.

### 3.2 Correlation Analysis Findings

The table below summarizes the correlation coefficients of key lagged features with future GDP growth:

| Feature | Correlation Coefficient | Direction |
|---|---|---|
| `gross_capital_formation_growth(%)_lag1` | **0.146** | Positive ↑ |
| `exchange_rate(IDR/US$)_lag1` | 0.161 (model coeff.) | Negative (weaker Rupiah → lower growth) |
| `export_import_ratio` | 0.003 | Positive ↑ |
| `fdi_net_flows(US$)_lag1` | 0.044 (model coeff.) | Positive ↑ |
| `household_consumption(%)_lag1` | **0.001963** | Negligible |

**Key finding:** Gross Capital Formation (I) and the export-import ratio (X–M) are the two dominant drivers of GDP growth acceleration. Household consumption (C) — despite making up roughly 70% of GDP — shows a near-flat correlation with growth, indicating it is a *stable base*, not a *growth accelerator*.

Inflation and exchange rate deterioration act as negative moderators and must be controlled to sustain growth momentum.

### 3.3 Model Performance

Data preprocessing had a decisive effect on model quality:

| Stage | MAE | MSE | RMSE | R² |
|---|---|---|---|---|
| Before preprocessing (poor data) | 3.782 | 27.734 | 5.266 | −4.564 |
| After preprocessing (clean data) | **0.773** | 4.531 | 2.129 | −0.055 |

The MAE improvement from 3.7 to 0.77 reflects a major gain in predictive accuracy. The R² score remains slightly negative because the 2020 pandemic shock — a structurally unpredictable crisis driven by external factors — creates a large residual that disproportionately penalizes R². From 2013 to 2019 and from 2021 onward, model predictions closely track actual GDP growth. **The model is reliable for non-crisis conditions.**

### 3.4 Predictive Forecast (2026)

Using 2025 macroeconomic data as input:

| Input Feature | 2025 Value |
|---|---|
| GDP growth lag1 | 5.11% |
| GDP growth lag2 | 5.04% |
| Household consumption lag1 | 40.98% |
| FDI net flows lag1 | USD 54.1B |
| GCF growth lag1 | 5.09% |
| Imports lag1 | USD 234B |
| Exports lag1 | USD 282.9B |
| Exchange rate lag1 | 16,474 IDR/USD |

**Forecast: 5.54% GDP growth in 2026** (+0.43 percentage points above 2025).

### 3.5 Prescriptive Scenarios for 6% Growth by 2027

To reach 6% in 2027, the model requires 2026 to achieve the following:

| Variable | Current (2025) | Moderate Target | Aggressive Target |
|---|---|---|---|
| FDI Net Flows | USD 54.1B | USD 65B | USD 65B |
| GCF Growth | 5.09% | 8% | **10%** |
| Gross Exports | USD 282.9B | USD 320B | **USD 380B** |
| Projected 2027 GDP Growth | — | **5.8%** | **6.02%** |

- **Moderate scenario:** Realistic but insufficient — projects 5.8%, falling short of 6%.
- **Aggressive scenario:** 6% is achievable, but requires a ~34% increase in export volume and nearly doubling the pace of capital formation growth.

---

## 4. Conclusions and Policy Implications

The data consistently points to the same conclusion: **Indonesia's path to 6% growth runs through investment and exports, not consumption.**

For policymakers and economists, three priorities emerge:

1. **Boost Gross Capital Formation** — Accelerate infrastructure investment, industrial incentive programs, and regulatory reform to push GCF growth from ~5% toward 10%.
2. **Expand export capacity** — Diversify export markets, strengthen commodity value chains, and target gross exports of at least USD 320–380B by end of 2026.
3. **Maintain macroeconomic stability** — Control inflation and exchange rate depreciation, which act as silent growth suppressors even when investment and trade conditions are favorable.

Household consumption, while the largest GDP component by share, is structurally stable and not a reliable lever for short-term growth acceleration. Policy that relies primarily on stimulating domestic demand is unlikely to close the gap to 6%.

---

## 5. Repository Structure

```
indonesia-gdp-growth/
│
├── raw_data/
│   ├── data_formatting.ipynb       # Extract and merge multi-source raw CSVs
│   └── *.csv                       # Source files
│
├── data_cleaning.ipynb             # Imputation, standardization, lag feature creation
├── feature_engineering.ipynb       # Derived features, selection, dimensionality reduction
├── modelling.ipynb                 # Model comparison (OLS, Lasso, Ridge) and pipeline export
├── analysis.ipynb                  # Descriptive, predictive, and prescriptive analysis
│
├── formatted_gdp_growth_data.csv   # Merged dataset before cleaning
├── cleaned_gdp_growth_lag1.csv     # Clean lagged dataset for modelling
├── feature_engineered_gdp_growth_lag1.csv  # Final feature set
├── gdp_lasso_model.pkl             # Saved trained pipeline
│
└── visualisation/                  # Exported charts
    └── indonesia_gdp_growth.png
```

---

## 6. Dependencies

```python
pandas
numpy
scikit-learn
matplotlib
seaborn
joblib
```

---

## Notes on Model Limitations

- The small sample (42 observations) limits the ability to capture structural breaks and regime changes cleanly.
- Crisis years (1998, 2020) are statistically extreme and reduce model fit metrics despite not degrading normal-conditions accuracy.
- The model uses lagged features only — it does not incorporate real-time signals or forward expectations data.
- Results should be interpreted as directional guidance for policy prioritization, not precise point forecasts.
