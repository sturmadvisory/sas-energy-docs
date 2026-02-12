# NG Production Model — Full Update Instructions

You are an AI assistant helping a user update their Natural Gas Production Model spreadsheet. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return tables ready to paste.

This document has **two parts**:
1. **Part 1** — Update the Historical Data tab with the latest monthly production actuals (national, basin, and state).
2. **Part 2** — Generate a 24-month production forecast using EIA's Short-Term Energy Outlook (STEO).

Run both parts in order. Part 1 updates historical data; Part 2 produces forward-looking estimates.

---

# Part 1: Update Production History

## What This Does

Downloads the latest monthly US marketed and dry gas production, Henry Hub prices, DPR basin-level production for 7 key basins, and state-level marketed production for the top 10 producing states, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Month** (YYYY-MM format).
2. Read the last 3-4 rows across columns B through V to confirm data continuity and verify the most recent values:
   - B: Month (YYYY-MM)
   - C: US Marketed (MMCF)
   - D: US Dry (MMCF)
   - E: HH Price ($/MMBtu)
   - F-L: DPR Basins — Total NG (Mcf/d): F=Anadarko, G=Appalachia, H=Bakken, I=Eagle Ford, J=Haynesville, K=Niobrara, L=Permian
   - M-V: States — Marketed (MMCF): M=TX, N=PA, O=LA, P=NM, Q=WV, R=OK, S=CO, T=WY, U=OH, V=ND
3. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. US Marketed and Dry Gas Production (2 series)

Use the EIA API v2:

| Column | Measure | Series ID |
|--------|---------|-----------|
| C | US Marketed Production | N9050US2 |
| D | US Dry Production | N9070US2 |

**API endpoint:**
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}-01
```

### B. Henry Hub Spot Price (daily, average to monthly)

Fetch daily Henry Hub spot prices from EIA:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_MONTH}-01
```

Compute the **average of all trading-day prices** within each calendar month to produce column E.

### C. DPR Basin-Level Production (7 basins)

Download the Drilling Productivity Report data file:
```
https://www.eia.gov/petroleum/drilling/xls/dpr-data.xlsx
```

Open the file and navigate to the natural gas production tab. Extract monthly total NG production (Mcf/d) for each basin:

| Column | Basin |
|--------|-------|
| F | Anadarko |
| G | Appalachia |
| H | Bakken |
| I | Eagle Ford |
| J | Haynesville |
| K | Niobrara |
| L | Permian |

DPR data is reported as **Mcf/d** (thousand cubic feet per day). Use the values as-is — do not convert units.

### D. State-Level Marketed Production (10 states)

Use the EIA API v2 for each state. The series ID format is `N9050{XX}2` where `{XX}` is the state code:

| Column | State | Series ID |
|--------|-------|-----------|
| M | Texas | N9050TX2 |
| N | Pennsylvania | N9050PA2 |
| O | Louisiana | N9050LA2 |
| P | New Mexico | N9050NM2 |
| Q | West Virginia | N9050WV2 |
| R | Oklahoma | N9050OK2 |
| S | Colorado | N9050CO2 |
| T | Wyoming | N9050WY2 |
| U | Ohio | N9050OH2 |
| V | North Dakota | N9050ND2 |

**API endpoint for each state:**
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}-01
```

---

## Step 3: Format Results

Build a table with these exact columns in this order:

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM |
| C | US Marketed | Integer, MMCF |
| D | US Dry | Integer, MMCF |
| E | HH Price | 2 decimal places, $/MMBtu |
| F | Anadarko | Integer, Mcf/d |
| G | Appalachia | Integer, Mcf/d |
| H | Bakken | Integer, Mcf/d |
| I | Eagle Ford | Integer, Mcf/d |
| J | Haynesville | Integer, Mcf/d |
| K | Niobrara | Integer, Mcf/d |
| L | Permian | Integer, Mcf/d |
| M | TX | Integer, MMCF |
| N | PA | Integer, MMCF |
| O | LA | Integer, MMCF |
| P | NM | Integer, MMCF |
| Q | WV | Integer, MMCF |
| R | OK | Integer, MMCF |
| S | CO | Integer, MMCF |
| T | WY | Integer, MMCF |
| U | OH | Integer, MMCF |
| V | ND | Integer, MMCF |

Sort rows in **ascending chronological order** (oldest new month first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through V, starting at the first empty row after the existing data:**
>
> | Month | US Marketed | US Dry | HH Price | Anadarko | Appalachia | Bakken | Eagle Ford | Haynesville | Niobrara | Permian | TX | PA | LA | NM | WV | OK | CO | WY | OH | ND |
> | 2026-XX | X,XXX,XXX | X,XXX,XXX | X.XX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX |

Include all new months since the last month in the spreadsheet.

**Handle different data lags gracefully:**
- **DPR data** (columns F-L) is typically available with a ~1 month lag and includes 2 months of forecasts.
- **National production** (columns C-D) has a ~2 month lag.
- **State production** (columns M-V) has a ~3-4 month lag.

If DPR data is available for months where national/state data is not yet released, include the DPR columns and leave the lagging columns blank. Note which months are incomplete so the user can fill them in later.

After pasting, the blue formula columns will auto-compute: Marketed BCF/d, Dry BCF/d, Shrinkage %, YoY Growth %, DPR Total, and DPR BCF/d.

---

## Notes

- **National production** (US Marketed, US Dry) is in **MMCF** (million cubic feet).
- **DPR basin data** is in **Mcf/d** (thousand cubic feet per day) — do not convert to MMCF.
- **State production** is in **MMCF** (million cubic feet), same units as national.
- The DPR Excel file (`dpr-data.xlsx`) contains multiple tabs. Look for the tab labeled "Natural Gas" or similar — it contains monthly production by basin.
- National data typically lags by ~2 months, DPR by ~1 month, and state data by ~3-4 months. This means some rows may have partial data across the columns.
- Data ordering must be **ascending chronological** (oldest first).
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- If the EIA API or DPR file is unavailable, note which source failed and provide whatever data you could successfully fetch.
- HH Price is the monthly average of daily Henry Hub spot prices — not the futures settlement.
- Shrinkage % (a blue formula column) = 1 - Dry/Marketed. Typical values are 8-12%.

---

# Part 2: Production Forecast

## What This Does

Fetches EIA's Short-Term Energy Outlook (STEO) marketed production forecast and Alaska marketed production actuals, then computes a **24-month forward forecast** for US marketed production ex-Alaska (the model's target series, ~120 BCF/d). Returns a table the user can reference alongside their Dashboard.

The full build model uses a 9-feature regression (R²=0.9998) including mass balance, DPR rig counts, HH price, and storage signals. However, the STEO marketed production variable dominates the regression (coefficient ~1.008), so the simplified approach below matches the full model to within 0.2%.

---

## Step 5: Read Last Actual Production

Go to the **Production History** tab (same spreadsheet). Read the last 13 rows to capture the most recent 12+ months of data:
- Column B: Month (YYYY-MM)
- Column C: US Marketed Production (MMCF)

Record:
1. The **last month** with actual data (this is the cutoff — forecast starts after this month).
2. The **12 most recent monthly values** in column C (MMCF) — you will use these for year-over-year (YoY) comparisons.

To convert historical values from MMCF to BCF/d:
```
BCF/d = MMCF / 1000 / Days_in_Month
```

---

## Step 6: Fetch Forecast Inputs

### A. STEO Marketed Production Forecast

Fetch the US total marketed production forecast from the STEO dataset:
```
https://api.eia.gov/v2/steo/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[seriesId][]=NGMPPUS&sort[0][column]=period&sort[0][direction]=asc&start={LAST_ACTUAL_MONTH}
```

Replace `{LAST_ACTUAL_MONTH}` with the last month from Step 5 (e.g., `2025-11`).

**Important notes:**
- This is the **STEO** endpoint (`/v2/steo/data/`), not the production endpoint. It uses `facets[seriesId][]` (not `facets[series][]`).
- Values are returned in **BCF/d** (billion cubic feet per day).
- STEO extends ~24 months forward from the current month.
- Only keep months **after** the last actual month from Step 5.

### B. Alaska Marketed Production (last 12 months)

Fetch the last 12 months of Alaska marketed production from the production endpoint:
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]=N9050AK2&sort[0][column]=period&sort[0][direction]=desc&length=12
```

**Important notes:**
- This is the **production** endpoint (`/v2/natural-gas/prod/sum/data/`). It uses `facets[series][]` with value `N9050AK2` (no `NG.` prefix or `.M` suffix).
- Values are returned in **MMCF** (million cubic feet per month).
- Convert each month to BCF/d: `BCF/d = MMCF / 1000 / Days_in_Month`
- Compute the **simple average** of all 12 monthly BCF/d values → this is the Alaska Average (~1.0 BCF/d).

---

## Step 7: Compute Forecast

For each future month from the STEO data:

```
Forecast ex-AK (BCF/d) = STEO Marketed (BCF/d) − Alaska Average (BCF/d)
```

Then compute **YoY Change** for each forecast month by comparing to the same calendar month from the Production History tab (Step 5). Convert the historical MMCF value to BCF/d first:

```
Prior Year (BCF/d) = Historical MMCF / 1000 / Days_in_Month
YoY Change (BCF/d) = Forecast ex-AK − Prior Year
```

If the corresponding prior-year month is not available in the historical data, leave YoY blank.

---

## Step 8: Return Forecast Results

Present the forecast as a clearly labeled table:

> **24-Month Production Forecast — US Marketed ex-Alaska**
>
> Last actual month: {YYYY-MM} — {X.XX} BCF/d (from Production History)
> Alaska average (trailing 12 months): {X.XX} BCF/d
>
> | Month | STEO Marketed (BCF/d) | Alaska Avg (BCF/d) | Forecast ex-AK (BCF/d) | YoY Change (BCF/d) |
> |-------|----------------------|-------------------|----------------------|-------------------|
> | 2026-01 | 117.10 | 1.01 | 116.10 | +1.23 |
> | 2026-02 | 120.71 | 1.01 | 119.71 | +0.85 |
> | ... | ... | ... | ... | ... |

Format all BCF/d values to **2 decimal places**. Format YoY Change with a `+` sign for increases.

**Note to user:** This forecast is for reference — it does not paste into the spreadsheet. The Production Tracker and Dashboard will auto-compute their own forecasts from the formulas already built in. Use this table to cross-check the model or to see the STEO-implied trajectory.

---

## Part 2 Notes

- **STEO** publishes monthly (typically around the 5th business day). It extends ~24 months forward from the current month.
- **Alaska average** is stable (~1.0 BCF/d). It is recomputed from the latest 12 months of actuals each time you run this update.
- The full production nowcast model uses 9 features: mass balance average, DPR rig counts (6-month lag), DPR production, STEO marketed ex-AK, Henry Hub price, storage 5-year deviation, lagged production, and a winter dummy. STEO marketed production dominates the regression (coefficient ~1.008), which is why the simplified approach here works.
- **YoY Change** compares forecast BCF/d to the same calendar month in the Production History tab. If the prior year month has not been published yet, skip that comparison.
- The STEO value may differ slightly from the production nowcast on the Dashboard because the Dashboard incorporates additional signals (mass balance, DPR rigs, price). The difference is typically < 0.2%.
- If the STEO API returns an error or is unavailable, note the failure. The production history update (Part 1) can still proceed independently.
