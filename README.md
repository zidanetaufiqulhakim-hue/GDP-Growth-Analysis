# Indonesia GDP Growth Analysis

This project studies Indonesia's GDP growth using annual macroeconomic data and an interpretable regression pipeline. The goal is to understand which indicators matter most, forecast GDP growth from prior-year conditions, and test what kind of macroeconomic improvements are associated with a path toward 6% growth.

![Indonesia GDP Growth](visualisation/indonesia_gdp_growth.png)

## Problem Breakdown

The project is built around four questions:

1. How has Indonesia's GDP growth behaved historically?
2. Which macroeconomic indicators move most closely with GDP growth?
3. Can GDP growth be predicted from lagged macroeconomic conditions?
4. What changes in key indicators are associated with higher-growth scenarios?

## Working Steps

1. `raw_data/data_formatting.ipynb`
   Extract Indonesia-specific data from multiple raw files and merge everything into one annual dataset.
2. `data_cleaning.ipynb`
   Repair missing values with interpolation and regression-based imputation, standardize labels, and create lagged features.
3. `feature_engineering.ipynb`
   Build derived features such as `real_interest_rate`, `exchange_rate_change_lag1`, and `export_import_ratio`, then reduce the feature set.
4. `modelling.ipynb`
   Compare linear regression, Lasso, and Ridge models, then save the best pipeline.
5. `analysis.ipynb`
   Use the cleaned data and trained model for descriptive insights, prediction, and scenario analysis.

## Model Behavior

The model is designed to be interpretable rather than complex. Instead of using black-box methods, the project relies on regularized linear models so the effect of each feature remains readable.

The modelling workflow behaves as follows:

- it starts from prior-year macroeconomic variables rather than same-year future information
- it engineers a few compact signals, such as trade balance and real interest pressure
- it scales the inputs to keep coefficients comparable
- it applies regularization to reduce overfitting in a small time-series dataset

In the model comparison notebook:

- Lasso performs best for the next-year GDP growth setup
- Ridge performs best for the current-year GDP growth setup

This means the final forecasting logic favors simple linear structure with coefficient shrinkage, not a highly flexible nonlinear model.

## Inputs

The dataset combines GDP growth with annual macroeconomic indicators, including:

- household consumption
- unemployment rate
- under-earning percentage
- inflation rate
- lending interest rate
- FDI net flows
- tax revenue
- state revenue
- gross capital formation growth
- imports and exports
- IDR/USD exchange rate

## Outputs

The main outputs of the project are:

- `formatted_gdp_growth_data.csv`: merged annual dataset before cleaning
- `cleaned_gdp_growth_lag1.csv`: cleaned lagged dataset for modelling
- `feature_engineered_gdp_growth_lag1.csv`: reduced feature set after engineering and selection
- `gdp_lasso_model.pkl`: saved trained forecasting pipeline
- `visualisation/`: charts for descriptive, correlation, and modelling results

The analysis notebook also produces:

- historical GDP growth visualizations
- correlation-based feature insights
- a forward GDP growth prediction
- moderate and aggressive scenario analyses for reaching higher growth

## Repository Map

- `raw_data/`: source CSV files and raw formatting notebook
- `data_cleaning.ipynb`: cleaning and imputation
- `feature_engineering.ipynb`: derived features and selection
- `modelling.ipynb`: model comparison and pipeline export
- `analysis.ipynb`: insights, forecast, and scenarios
- `visualisation/`: exported plots

## Author

Zidane Taufiqul Hakim
