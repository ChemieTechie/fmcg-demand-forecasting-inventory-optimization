# FMCG Demand Forecasting & Inventory Optimization

An end-to-end retail/FMCG demand-planning project using the public Walmart M5 dataset to forecast SKU-store demand and translate forecasts into inventory-replenishment decisions.

## Objective

Forecast the next 28 days of SKU-store demand using historical sales and calendar information, evaluate practical forecasting approaches, and convert the resulting forecasts into safety-stock and reorder-point recommendations.

## Project workflow

Historical sales → Data cleaning → EDA → Representative SKU-store selection → Forecasting baselines → Statistical & ML forecasting → 28-day validation → Out-of-sample evaluation → Safety stock → Reorder point → Business recommendations

## Dataset

The project uses the public Walmart M5 retail forecasting dataset as a retail/FMCG proxy; no proprietary HUL data is assumed.

- 30,490 SKU-store series
- 3,049 items
- 10 stores
- 3 categories: FOODS, HOUSEHOLD, HOBBIES
- 7 departments
- 3 states: CA, TX, WI
- 1,913 historical sales days
- 28-day future evaluation horizon
- Calendar, event, SNAP and weekly selling-price information

Raw M5 data is intentionally **not committed to this repository** because of its size. See `data/README.md` for the expected file layout and source.

## Key EDA findings

- FOODS accounts for approximately **68.6%** of historical unit volume.
- HOUSEHOLD accounts for approximately **22.0%** and HOBBIES **9.3%**.
- Approximately **68.2%** of SKU-store-day observations have zero sales, indicating substantial intermittent demand.
- Aggregate daily demand shows a weekly pattern, with Saturday highest and Wednesday lowest in the initial descriptive analysis.

## Forecasting approach

Five approaches were compared:

1. Naive forecast
2. 7-day seasonal naive
3. 7-day moving average
4. Damped Holt-Winters exponential smoothing
5. Gradient Boosting using lag, rolling and calendar features

The validation design is chronological: the first 1,885 historical days are used for training and the following 28 days for model selection. The selected approach is then refit on all 1,913 historical days and evaluated on the next 28 actual days from the evaluation file.

### Validation results

| Model | Mean validation MAE |
|---|---:|
| 7-day Moving Average | **9.70** |
| Gradient Boosting | 10.43 |
| 7-day Seasonal Naive | 11.59 |
| Holt-Winters | 16.07 |
| Naive | 24.07 |

The selected per-series approaches achieved an average validation MAE of **8.41**, versus **24.07** for the naive benchmark on the selected seven-series panel.

### 28-day out-of-sample evaluation

- MAE: **7.99**
- RMSE: **9.82**
- WAPE: **39.25%**
- Forecast total: approximately **6,963 units**
- Actual total: approximately **6,725 units**

These results apply to the deliberately selected seven-series panel and should not be presented as full-dataset M5 leaderboard performance.

## Inventory optimization

Because the public dataset does not contain true supplier lead times or on-hand inventory, the inventory layer uses explicit assumptions:

- Lead time: **7 days**
- Target service level: **95%**
- Safety stock based on recent seasonal-naive residual variability

Core calculations:

`Lead-time demand = average forecast demand × lead time`

`Safety stock = z × demand variability × √lead time`

`Reorder point = lead-time demand + safety stock`

For a representative `FOODS_3_586 / CA_3` series, the analysis produced approximately 73.3 units/day forecast demand, 512.9 units of seven-day lead-time demand, 82.1 units of safety stock and a 595-unit reorder point under these assumptions.

## Repository structure

```text
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
├── notebooks/
│   └── FMCG_Demand_Forecasting_Inventory_Optimization.ipynb
├── outputs/
│   ├── forecasts/
│   ├── model_results/
│   └── figures/
└── report/
    └── PROJECT_REPORT.md
```

## Limitations

- The dataset is public Walmart retail data, not HUL data.
- The modeling panel contains seven representative SKU-store series rather than all 30,490 series.
- Inventory recommendations are illustrative because true lead time and on-hand inventory are unavailable.
- The model set is practical rather than exhaustive; global/hierarchical forecasting can be explored as a future extension.

## HUL interview takeaway

> I built an independent FMCG demand-planning project using the public Walmart M5 retail dataset. I analyzed SKU-store demand patterns, including strong intermittency, compared baseline, statistical and machine-learning forecasting approaches on a 28-day validation horizon, selected the best approach per series, evaluated it on the next 28 actual days, and translated the forecast into safety-stock and reorder-point recommendations. The main lesson was that forecasting should ultimately support an inventory decision, not just produce a prediction.
