# NG Production Nowcast — Full Update Instructions

You are an AI assistant helping a user update their NG Production Nowcast spreadsheet with the latest monthly demand data and weekly storage levels. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return tables ready to paste.

## What This Does

Downloads the latest monthly natural gas consumption by sector, imports/exports, and EIA-914 production data, plus weekly storage levels from the EIA API. Returns two separate tables — one for the monthly section and one for the weekly storage section — formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. This tab has TWO separate data sections with their own header rows.

### A. Monthly Supply/Demand Section (upper section, data starts at row 7)

1. Read column B to find the **last Month** (YYYY-MM format).
2. Read the last 3-4 rows across columns B through M to confirm data continuity:
   - B: Month (YYYY-MM)
   - C: Days in Month
   - D: Residential (MMCF)
   - E: Commercial (MMCF)
   - F: Industrial (MMCF)
   - G: Electric (MMCF)
   - H: Lease/Plant (MMCF)
   - I: Pipeline (MMCF)
   - J: Vehicle (MMCF)
   - K: Tot Imports (MMCF)
   - L: Tot Exports (MMCF)
   - M: EIA-914 Prod (MMCF)
3. Record the last month — you will only fetch data **after** this month.

### B. Weekly Storage Section (lower section, below the monthly data)

1. Locate the weekly storage section header row (below the monthly data).
2. Read column B to find the **last Week Ending Date** (YYYY-MM-DD, Friday dates).
3. Read the last 3-4 rows across columns B and C:
   - B: Week Ending Date
   - C: Storage Level (BCF)
4. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

### A. Monthly Consumption by Sector (7 series)

Use the EIA API v2 to fetch each series for all months after the last month in the spreadsheet. Replace `{LAST_MONTH}` with the month from Step 1 in YYYY-MM format.

| Column | Sector | Series ID |
|--------|--------|-----------|
| D | Residential | NG.N3010US2.M |
| E | Commercial | NG.N3020US2.M |
| F | Industrial | NG.N3035US2.M |
| G | Electric Power | NG.N3045US2.M |
| H | Lease & Plant | NG.N9160US2.M |
| I | Pipeline & Distribution | NG.N9170US2.M |
| J | Vehicle Fuel | NG.N3025US2.M |

**API endpoint for each series:**
```
https://api.eia.gov/v2/natural-gas/cons/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}
```

### B. Imports and Exports (5 series, summed to 2 columns)

**Total Imports (col K)** = sum of three series:
| Component | Series ID |
|-----------|-----------|
| Imports from Canada | NG.N9102CN2.M |
| Imports from Mexico | NG.N9102MX2.M |
| LNG Imports | NG.N9103US2.M |

**Total Exports (col L)** = sum of three series:
| Component | Series ID |
|-----------|-----------|
| Exports to Canada | NG.N9132CN2.M |
| Exports to Mexico | NG.N9132MX2.M |
| LNG Exports | NG.N9133US2.M |

**API endpoint for trade series:**
```
https://api.eia.gov/v2/natural-gas/move/impc/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}
```

### C. EIA-914 Production (col M)

Fetch monthly marketed production:
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]=NG.NGMPMUS2.M&start={LAST_MONTH}
```

Note: EIA-914 production data lags approximately 2 months. If no new production data is available for a given month, leave column M blank for that row.

### D. Weekly Storage Levels

Fetch weekly storage for all weeks after the last date in the weekly section:
```
https://api.eia.gov/v2/natural-gas/stor/wkly/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=weekly&data[0]=value&facets[series][]=NW2_EPG0_SWO_R48_BCF&start={LAST_DATE}
```

---

## Step 3: Format Results

### Table 1: Monthly Supply/Demand

Build a table with these exact columns. Each row represents one month.

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM |
| C | Days | Integer (days in that month) |
| D | Residential | Integer, MMCF |
| E | Commercial | Integer, MMCF |
| F | Industrial | Integer, MMCF |
| G | Electric | Integer, MMCF |
| H | Lease/Plant | Integer, MMCF |
| I | Pipeline | Integer, MMCF |
| J | Vehicle | Integer, MMCF |
| K | Tot Imports | Integer, MMCF (sum of 3 import series) |
| L | Tot Exports | Integer, MMCF (sum of 3 export series) |
| M | EIA-914 Prod | Integer, MMCF (blank if not yet available) |

Sort rows in **ascending chronological order** (oldest new month first).

### Table 2: Weekly Storage

Build a table with these columns. Each row represents one weekly report.

| Column | Header | Format |
|--------|--------|--------|
| B | Week Ending | YYYY-MM-DD (Friday) |
| C | Storage Level | Integer, BCF |

Sort rows in **ascending chronological order** (oldest new week first).

---

## Step 4: Return Results

Return **two separate paste-ready tables**.

> **Table 1 — Paste into Historical Data tab, monthly section, columns B through M, starting at the first empty row after the existing monthly data:**
>
> | Month | Days | Residential | Commercial | Industrial | Electric | Lease/Plant | Pipeline | Vehicle | Tot Imports | Tot Exports | EIA-914 Prod |
> | 2026-XX | XX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | X,XXX | XXX,XXX | XXX,XXX | XXX,XXX |

> **Table 2 — Paste into Historical Data tab, weekly storage section, columns B through C, starting at the first empty row after the existing weekly data:**
>
> | Week Ending | Storage Level |
> | 2026-XX-XX | X,XXX |

Include all new data since the last dates in the spreadsheet. If there is no new data available for either section, say so explicitly for that section.

After pasting monthly data, the blue formula columns (N-W: BCF/d conversions = MMCF / 1000 / Days) and gray computed columns (X-Y: Total Demand BCF/d, Net Imports BCF/d) will auto-compute. In the weekly section, column D (Storage Change = C - C[-1]) will also auto-compute.

The **Weekly Estimates** tab uses INDEX-MATCH formulas from these Historical Data values to compute the weekly production nowcast via mass balance: Implied Production = Total Demand + Storage Change / 7 - Net Imports.

---

## Notes

- All monthly input values are in **MMCF** (million cubic feet). Do not convert to BCF — the blue formula columns handle that conversion.
- Weekly storage levels are in **BCF** (billion cubic feet).
- The Historical Data tab has **two separate sections** with their own header rows on the same sheet. Paste monthly data into the monthly section and weekly data into the weekly section. Do not mix them.
- Monthly data typically lags ~2 months. EIA-914 production data may lag even further. It is normal for the most recent month to have demand data but no production value yet.
- Weekly storage is current — EIA publishes every Thursday at 10:30 AM ET with data through the prior Friday.
- Data ordering must be **ascending chronological** (oldest first) in both sections.
- Days in month (col C): Use the actual calendar days (28/29 for Feb, 30 or 31 for other months). This is critical because the BCF/d conversion divides by this value.
- Total Imports (col K) and Total Exports (col L) are each the sum of three sub-series (Canada pipeline, Mexico pipeline, LNG). Sum them before pasting — the spreadsheet expects a single total in each column.
- The nowcast model achieves 1.33% MAPE vs EIA-914, so data accuracy in these input columns directly affects forecast quality.
- If you cannot fetch a data source, note which one failed and provide the data you were able to retrieve. Partial updates are acceptable.
