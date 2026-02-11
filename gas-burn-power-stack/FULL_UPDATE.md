# Gas Burn & Power Stack Model — Full Update Instructions

You are an AI assistant helping a user update their Gas Burn & Power Stack Model spreadsheet with the latest monthly natural gas consumption and power generation data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest monthly natural gas consumption by sector (Electric Power, Residential, Commercial, Industrial) and natural gas generation from the EIA API, along with Henry Hub spot prices averaged to monthly, and returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Month** (YYYY-MM format).
2. Read the last 3-4 rows across columns B through H to confirm data continuity and verify the most recent values:
   - B: Month (YYYY-MM)
   - C: Elec Power (MMCF)
   - D: Residential (MMCF)
   - E: Commercial (MMCF)
   - F: Industrial (MMCF)
   - G: HH Price ($/MMBtu)
   - H: NG Gen (000 MWh)
3. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. Monthly Natural Gas Consumption by Sector (4 series)

Use the EIA API v2 to fetch each series for all months after the last month in the spreadsheet.

| Column | Sector | Series ID |
|--------|--------|-----------|
| C | Electric Power | NG.N3045US2.M |
| D | Residential | NG.N3010US2.M |
| E | Commercial | NG.N3020US2.M |
| F | Industrial | NG.N3035US2.M |

**API endpoint for each series:**
```
https://api.eia.gov/v2/natural-gas/cons/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}-01
```

Replace `{SERIES_ID}` with each series ID above and `{LAST_MONTH}` with the last month in YYYY-MM format.

### B. Henry Hub Spot Price (daily, average to monthly)

Fetch daily Henry Hub spot prices from EIA:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_MONTH}-01
```

Compute the **average of all trading-day prices** within each calendar month to produce column G.

### C. Natural Gas Generation (EIA Grid Monitor)

Fetch monthly NG generation in MWh from:
```
https://www.eia.gov/electricity/gridmonitor/
```

Navigate to the generation by fuel type data and extract the monthly natural gas generation value. Convert to **thousands of MWh (000 MWh)** by dividing the raw MWh value by 1,000.

Alternatively, use the EIA Electricity API:
```
https://api.eia.gov/v2/electricity/electric-power-operational-data/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=generation&facets[fueltypeid][]=NG&facets[location][]=US&facets[sectorid][]=99&start={LAST_MONTH}
```

This returns generation in thousand MWh, which maps directly to column H.

---

## Step 3: Format Results

Build a table with these exact columns in this order:

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM |
| C | Elec Power | Integer, MMCF |
| D | Residential | Integer, MMCF |
| E | Commercial | Integer, MMCF |
| F | Industrial | Integer, MMCF |
| G | HH Price | 2 decimal places, $/MMBtu |
| H | NG Gen | Integer, 000 MWh |

Sort rows in **ascending chronological order** (oldest new month first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through H, starting at the first empty row after the existing data:**
>
> | Month | Elec Power | Residential | Commercial | Industrial | HH Price | NG Gen |
> | 2026-XX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | X.XX | XXX,XXX |

Include all new months since the last month in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns will auto-compute: Elec BCF/d, Total MMCF, Total BCF/d, NG Share %, Elec % of Total, and YoY Growth %. The gray columns will auto-compute: Res/Com/Ind BCF/d conversions.

---

## Notes

- All consumption values are in **MMCF** (million cubic feet). The blue formula columns convert to BCF/d by dividing by 1,000 and the number of days in the month.
- NG Gen is in **thousands of MWh (000 MWh)** — not raw MWh.
- EIA consumption data typically has a **2-month lag**. For example, in February 2026, data through November or December 2025 is usually the latest available.
- Data ordering must be **ascending chronological** (oldest first).
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- If the EIA API returns no new data, report that clearly and note the typical publication schedule (consumption data releases around the end of each month for data 2 months prior).
- If NG Gen data has a different lag than consumption data, include the consumption rows and leave NG Gen blank for months where generation data is not yet available.
- If you cannot fetch a data source, note which one failed and use the most recent available value from the spreadsheet as a fallback.
- HH Price is the monthly average of daily Henry Hub spot prices — not the futures settlement.
