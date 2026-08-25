# FMCG Demand Forecasting & Inventory Optimization

## 1. Executive Summary

**Objective:** Build an end-to-end retail/FMCG demand-planning workflow that forecasts SKU-store demand for the next 28 days and converts the forecast into inventory-replenishment decisions.

**Public data:** Walmart M5 retail forecasting dataset supplied by the user. It is used as a public retail/FMCG proxy; no proprietary HUL data is assumed.

**Final workflow:**

Historical sales → EDA → representative SKU-store selection → forecasting baselines → statistical forecasting → machine-learning forecasting → model selection → 28-day out-of-sample evaluation → safety stock → reorder point → business recommendations.

## 2. Dataset

The uploaded dataset contains:

- 30,490 SKU-store series
- 3,049 items
- 10 stores
- 3 categories: FOODS, HOUSEHOLD, HOBBIES
- 7 departments
- 3 states: CA, TX, WI
- 1,913 historical sales days
- 28-day future evaluation period
- Calendar features including weekday, month, year, events and SNAP indicators
- Weekly selling prices by store and item

The historical sales matrix is wide: one row represents a SKU-store series and columns `d_1` to `d_1913` represent daily unit sales.

## 3. Data Cleaning

### Calendar
- Parsed dates to datetime.
- Replaced missing event labels with `None` so that “no event” is explicit.
- Checked duplicate date/day combinations.

### Selling prices
- Converted prices to numeric.
- Removed missing and non-positive prices.
- Removed duplicate store-item-week combinations.
- Sorted by store, item and week.

### Sales
The sales files required **no value-level correction**: no missing sales values, negative unit sales, non-integer unit observations, or duplicate IDs were found in the uploaded files.

This is important: the actual demand values were not altered.

## 4. Exploratory Data Analysis

### Category mix

By historical unit volume:

- **FOODS: 68.6%**
- **HOUSEHOLD: 22.0%**
- **HOBBIES: 9.3%**

FOODS was therefore a natural primary category for the project, while the final modeling panel retained multiple categories to avoid overfitting the story to one category.

### Intermittent demand

Approximately **68.2% of SKU-store-day cells have zero sales**. This indicates substantial intermittent demand and is why the project does not rely on ordinary MAPE alone.

### Weekly pattern

In the initial aggregate analysis, Saturday had the highest average daily unit sales and Wednesday the lowest. This is descriptive evidence of weekly seasonality; it is not treated as proof of causality.

## 5. Representative Forecasting Panel

To keep the project interpretable and computationally manageable, seven active SKU-store series were selected from the three categories. Selection emphasized distinct demand behavior rather than blindly choosing one product.

The panel includes high-volume, relatively stable, volatile and lower-volume/intermittent patterns.

## 6. Forecasting Design

### Validation design

- Train on the first **1,885** historical days.
- Validate on the following **28** historical days (`d_1886`–`d_1913`).
- Select the best model per SKU-store using validation MAE, with RMSE as a secondary criterion.
- Refit the selected approach using all 1,913 historical days.
- Forecast the next 28 days.
- Compare the forecast with the actual future 28 days contained in `sales_train_evaluation.csv`.

This prevents future leakage during model selection.

### Models

1. Naive forecast
2. 7-day seasonal naive forecast
3. 7-day moving average
4. Damped Holt-Winters exponential smoothing
5. Gradient Boosting using lag, rolling and calendar features

### Machine-learning features

- Lag 1 day
- Lag 7 days
- Lag 14 days
- Lag 28 days
- 7-day rolling mean
- 28-day rolling mean
- 7-day rolling standard deviation
- Day-of-week position
- Calendar-period indicator
- Month
- Weekend indicator

## 7. Validation Results

Across the selected seven series, the mean validation MAE by model was:

| Model | Mean MAE |
|---|---:|
| 7-day Moving Average | **9.70** |
| Gradient Boosting | 10.43 |
| 7-day Seasonal Naive | 11.59 |
| Holt-Winters | 16.07 |
| Naive | 24.07 |

The best model **depends on the SKU-store series**. Gradient Boosting performed best for several higher-volume/volatile series, while Holt-Winters was selected for some lower-volume series. This is a useful business lesson: a single forecasting model does not have to be optimal for every SKU.

On the validation panel, the average MAE of the selected per-series model was **8.41**, versus **24.07** for the simple naive benchmark — about a **65.0% reduction in MAE** on this selected validation panel.

## 8. Out-of-Sample 28-Day Evaluation

After model selection, the selected approach was refit using all historical observations and evaluated on the next 28 actual days.

Average metrics across the seven selected series:

- **MAE: 7.99**
- **RMSE: 9.82**
- **WAPE: 39.25%**

The chosen forecasts totalled approximately **6,963 units**, versus **6,725 actual units** across the seven series and 28-day horizon.

The aggregate forecast was therefore approximately 3.5% above actual demand for this small evaluation panel. This should not be interpreted as a universal model bias because the panel is deliberately small and representative rather than a full-dataset benchmark.

## 9. Inventory Optimization

The public M5 data does not provide supplier lead time or on-hand inventory, so inventory recommendations require explicit assumptions.

### Assumptions

- Lead time = **7 days**
- Target service level = **95%**
- Safety stock based on recent seasonal-naive residual variability

### Logic

**Lead-time demand** = Average forecast demand × Lead time

**Safety stock** = z × demand variability × √Lead time

For a 95% service level, `z ≈ 1.645`.

**Reorder Point** = Lead-time demand + Safety stock

### Example

For `FOODS_3_586` at `CA_3`:

- Average forecast = ~73.3 units/day
- Lead time = 7 days
- Lead-time demand = ~512.9 units
- Safety stock = ~82.1 units
- Reorder point = ~595 units

Interpretation: under the stated assumptions, replenishment should be triggered when inventory approaches roughly 595 units.

## 10. Business Insights

### Insight 1 — Forecasting should be SKU-specific

Different series showed different winning models. A single model across every product can leave performance on the table.

### Insight 2 — Intermittent demand matters

The high share of zero-sales observations means inventory and forecast evaluation must be designed carefully for sparse demand.

### Insight 3 — Forecasting alone is not enough

The most business-relevant output is not merely “predicted sales.” It is the translation of predicted demand into a replenishment rule.

### Insight 4 — Service level and inventory cost are a trade-off

Higher safety stock protects against stockouts but increases working capital and holding cost. The 95% service-level assumption is a business choice, not a universal optimum.

### Insight 5 — Data visibility supports decisions

Sales, price and calendar information provide the inputs needed to improve planning responsiveness.

## 11. Limitations

1. The public dataset is retail/Walmart data, not HUL data.
2. The project uses a representative seven-series panel, not all 30,490 series, for interpretability and speed.
3. True inventory on-hand and supplier lead-time data are unavailable, so inventory recommendations are illustrative and assumption-driven.
4. The model set is deliberately practical rather than exhaustive; more advanced global/hierarchical forecasting could be explored later.
5. Results depend on the selected panel and should not be presented as universal M5 leaderboard performance.

## 12. HUL Interview Explanation

A concise explanation:

> “I built an independent FMCG demand-planning project using the public Walmart M5 retail dataset. I first analyzed SKU-store demand patterns, including strong intermittency. I compared simple baselines, Holt-Winters and a lag-based Gradient Boosting model on a 28-day validation window, selected the best approach per series, then evaluated the forecasts on the next 28 actual days. Finally, I translated the forecast into safety-stock and reorder-point recommendations using explicit service-level and lead-time assumptions. The main lesson was that forecasting should ultimately support an inventory decision, not just produce a prediction.”

## 13. Resume-Ready Version

**FMCG Demand Forecasting & Inventory Optimization — Independent Project**

- Built SKU-store demand forecasts using historical sales, price and calendar data for 28-day FMCG demand prediction.
- Converted forecasts into safety-stock and reorder-point decisions under demand and lead-time variability.

Once the exact final code/notebook is reviewed and committed, quantified results can be added to these bullets where appropriate.
