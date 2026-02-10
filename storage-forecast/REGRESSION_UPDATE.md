# NG Storage Forecast — Regression Update Instructions

You are an AI assistant helping a user update their Natural Gas Storage Forecast spreadsheet with fresh regression-based forecasts. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

The spreadsheet has a regression model that predicts the weekly storage change "surprise" — the deviation from the 5-year seasonal average. The model coefficients are pre-trained and stored in the Model Reference tab. You will:

1. Read the coefficients and current data from the spreadsheet
2. Fetch fresh fundamentals data from public sources
3. Compute updated surprise predictions for the 3 most recent weeks + 3 weeks ahead
4. Return results formatted for the user to paste into the Storage Tracker

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and extract:

### From Model Reference tab — "REGRESSION COEFFICIENTS" section:
Read all 8 coefficients:
| # | Feature | Column |
|---|---------|--------|
| 0 | Intercept | D (row with "Intercept" in B) |
| 1 | HDD Dev Normal | D |
| 2 | CDD Dev Normal | D |
| 3 | HH Price YoY | D |
| 4 | Surplus/Deficit (/100) | D |
| 5 | Gas Rig YoY | D |
| 6 | Winter Dummy | D |
| 7 | Prior Wk Surprise | D |

### From Storage Tracker tab:
- Latest date (column A, row 4)
- Latest storage total (column B, row 4)
- Latest 5-Yr Avg Change for each of the last few weeks (column D)
- Latest Surprise values (column E) — the most recent non-blank value is the "prior week surprise"
- Latest Reg Surprise if present (column P)

### From Model Reference tab — "SEASONAL STORAGE LEVELS" section:
- Read the 5-Yr Avg level for the current ISO week and the next 3 ISO weeks

---

## Step 2: Fetch Fresh Data

### A. Weather — US HDD and CDD deviation from normal
Go to: `https://www.eia.gov/naturalgas/storage/`

On the latest Weekly Natural Gas Storage Report page, find the "Weather" section showing US Total HDD and CDD with deviations from normal. Extract:
- `us_hdd_dev_normal` (positive = colder than normal)
- `us_cdd_dev_normal` (positive = hotter than normal)

If weather data is not yet available for the most recent week, use the prior week's values.

### B. Henry Hub Price
Go to EIA for recent Henry Hub spot prices. Get:
- Current week's average HH price
- The HH price from exactly 52 weeks ago (same ISO week last year)
- Compute: `hh_yoy = current_price - price_52wk_ago`

### C. Gas-Directed Rig Count
Go to: `https://bakerhughes.com/rig-count/north-america-rig-count`

Get the latest US gas-directed rig count and the count from 52 weeks ago.
- Compute: `rig_yoy = current_rigs - rigs_52wk_ago`

---

## Step 3: Compute Features and Predictions

For each week you're predicting, build the feature vector:

```
features = [
    1.0,                                          # Intercept
    hdd_dev_normal,                               # US HDD deviation from normal
    cdd_dev_normal,                               # US CDD deviation from normal
    hh_yoy,                                       # Henry Hub price YoY change ($/MMBtu)
    (current_storage - fiveyr_avg_level) / 100,   # Surplus/deficit scaled
    rig_yoy,                                      # Gas rig count YoY change
    1.0 if month in [11,12,1,2,3] else 0.0,      # Winter dummy
    prior_week_surprise                            # Previous week's actual or predicted surprise
]
```

Then:
```
predicted_surprise = sum(features[i] * coefficients[i] for i in range(8))
blended_change = five_yr_avg_change + predicted_surprise
blended_level = prior_level + blended_change
```

### For 3 weeks ahead (future), approximate features:
| Feature | Week +1 | Week +2 | Week +3 |
|---------|---------|---------|---------|
| HDD/CDD Dev | Current value | Current * 0.5 | Current * 0.25 |
| HH Price YoY | Current | Same | Same |
| Surplus/Deficit | From current level | From W+1 forecast level | From W+2 forecast level |
| Gas Rig YoY | Current | Same | Same |
| Winter Dummy | From calendar | From calendar | From calendar |
| Prior Surprise | Last actual surprise | W+1 predicted surprise | W+2 predicted surprise |

---

## Step 4: Return Results

Return a table with these columns. Include the 3 most recent historical weeks (with actual data) and 3 forecast weeks:

```
| Week | Date | Type | HDD Dev | CDD Dev | HH YoY | Surplus | Rig YoY | Winter | Prior Surp | Pred Surprise | 5yr Avg Chg | Blended Chg | Blended Level |
```

Mark historical rows as "Actual" and future rows as "Forecast".

Then provide paste-ready values:

> **Paste into Storage Tracker columns P (Reg Surprise) and Q (Blended Chg):**
>
> | Date | Reg Surprise (col P) | Blended Chg (col Q) |
> | ... | ... | ... |

For the 3 forecast weeks, also provide:

> **Updated Blended Forecast for Dashboard reference:**
>
> | Week +1 | Week +2 | Week +3 |
> | Level: X,XXX | Level: X,XXX | Level: X,XXX |

---

## Notes

- All storage values are in BCF (billion cubic feet)
- Surprise = Actual weekly change - 5-year average weekly change
- Positive surprise = more injection (or less withdrawal) than seasonal norm
- Negative surprise = more withdrawal (or less injection) than seasonal norm
- The model R-squared and diagnostics are in the "REGRESSION MODEL" section of Model Reference
- If you cannot fetch a data source, note which one and use the most recent available value from the spreadsheet
