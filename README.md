# Retail Sales Forecasting — SKU-Level Demand Prediction

Monthly sales quantity forecasting for a multi-region retail chain, built from raw transaction data through to a tuned model that outperforms a naive seasonal baseline.

## Overview

Given ~8,000 raw point-of-sale transactions across multiple locations and three product SKUs, the goal is to forecast **monthly sales quantity per SKU × region** — a level of granularity useful for inventory planning. The project covers the full pipeline: data quality investigation, cleaning, feature engineering, model selection, hyperparameter tuning, and a recursive multi-month forecast for the following year.

## Data

| File | Description |
|---|---|
| `data_transactions.csv` | ~7,950 transactions: date, location, SKU, quantity, value |
| `data_products.csv` | Product catalog: SKU, product family, MSRP, margin |
| `data_regions.csv` | Location metadata: region type, region area |

## Methodology

**1. Exploratory Data Analysis**
Investigated data quality before any modeling: verified transaction ID uniqueness, flagged an anomalous quantity value (132, likely a data-entry error), and identified 79 negative-quantity transactions interpreted as returns. Returns were matched back to their original transaction (same SKU, location, and offsetting value — falling back to the closest prior transaction when no exact match existed) so both could be removed as a pair rather than treated as independent noise.

**2. Data Cleaning**
Parsed dates, removed matched return pairs, merged in region metadata, and ordinal-encoded region type (City < Suburban < Rural).

**3. Feature Engineering**
Data aggregated to monthly SKU × region granularity, with three families of features:
- **Temporal**: month, quarter, and a cyclical sin/cos encoding of month (so December and January are treated as adjacent rather than far apart)
- **Lag features**: quantity and transaction count at M-1, M-2, M-3
- **Rolling features**: 3-month and 6-month rolling averages

**4. Modeling — 2025 Forecast**
Three models compared on a chronological train/test split (train ≤ 2024, test = 2025): Linear Regression, Gradient Boosting, and Random Forest. Random Forest performed best and was then tuned via grid search (`ParameterGrid`, deliberately without cross-validation given the time-ordered structure of the data). Performance was benchmarked against a naive baseline — the same-month average from prior years.

**5. 2026 Forecast (Recursive)**
The tuned Random Forest was retrained on the full available history (2022 – March 2026) and used to forecast April–December 2026 recursively: each month's prediction is fed back in as a lag feature for the following month.

## Results

| Model | MAE | WAPE | R² |
|---|---|---|---|
| Linear Regression | 5.49 | 28.2% | 0.707 |
| Gradient Boosting | 4.93 | 25.3% | 0.733 |
| Random Forest (baseline params) | 4.62 | 23.7% | 0.782 |
| **Random Forest (tuned)** | **4.55** | **23.3%** | **0.789** |
| Naive (seasonal average) | 5.57 | 28.6% | 0.64 |

The tuned model beats the naive baseline across every SKU and region, with the largest gains in low-volume, high-variance segments. The most influential features were the 3-month and 6-month rolling averages, followed by seasonality (month/quarter).

**Key takeaway**: the recursive 2026 forecast provides a usable baseline for inventory planning, with reliability naturally decreasing further into the forecast horizon — accuracy can be maintained without retraining by feeding real sales figures back into the lag/rolling features at the end of each month.


## Project Structure

```
.
├── notebook.ipynb          # Full analysis: EDA → cleaning → features → modeling → forecast
├── data/
│   ├── transactions.csv
│   ├── products.csv
│   └── regions.csv
├── requirements.txt
└── README.md
```
