# NG Balance Report — Full Update Instructions

You are an AI assistant helping a user update their Natural Gas Balance Report spreadsheet with the latest monthly EIA data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest monthly natural gas supply, demand, trade, storage, and price data from the EIA API across 19 series, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Month** (MMM-YY format, e.g., Jan-26).
2. Read column C to confirm **Days in Month** values for recent rows.
3. Read the last 3-4 rows across columns B through V to confirm data continuity and verify the most recent values:
   - B: Month (MMM-YY)
   - C: Days in Month
   - D: Dry Prod (MMCF)
   - E: Pipe Imp CA (MMCF)
   - F: Pipe Imp MX (MMCF)
   - G: LNG Imp (MMCF)
   - H: Residential (MMCF)
   - I: Commercial (MMCF)
   - J: Industrial (MMCF)
   - K: Electric (MMCF)
   - L: Vehicle (MMCF)
   - M: Lease-Plant (MMCF)
   - N: Pipeline Use (MMCF)
   - O: Pipe Exp CA (MMCF)
   - P: Pipe Exp MX (MMCF)
   - Q: LNG Exp (MMCF)
   - R: Net Wdraw (MMCF)
   - S: Work Gas (MMCF)
   - T: L48 Stor (MMCF)
   - U: AK Stor (MMCF)
   - V: HH Price ($/MMBtu)
4. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. Monthly EIA Series (16 series)

Use the EIA API v2 to fetch each series for all months after the last month in the spreadsheet. Replace `{START_DATE}` with the first day of the month after the last row (e.g., if last row is Nov-25, use `2025-12-01`).

**API key:** `ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc`

| Column | Field | Series ID | API Endpoint |
|--------|-------|-----------|--------------|
| D | Dry Prod | NG.N9070US2.M | /prod/sum/data/ |
| E | Pipe Imp CA | NG.N9102CN2.M | /move/impc/data/ |
| F | Pipe Imp MX | NG.N9102MX2.M | /move/impc/data/ |
| G | LNG Imp | NG.N9103US2.M | /move/impc/data/ |
| H | Residential | NG.N3010US2.M | /cons/sum/data/ |
| I | Commercial | NG.N3020US2.M | /cons/sum/data/ |
| J | Industrial | NG.N3035US2.M | /cons/sum/data/ |
| K | Electric | NG.N3045US2.M | /cons/sum/data/ |
| L | Vehicle | NG.N3025US2.M | /cons/sum/data/ |
| M | Lease-Plant | NG.N9160US2.M | /prod/sum/data/ |
| N | Pipeline Use | NG.N9170US2.M | /cons/sum/data/ |
| O | Pipe Exp CA | NG.N9132CN2.M | /move/expc/data/ |
| P | Pipe Exp MX | NG.N9132MX2.M | /move/expc/data/ |
| Q | LNG Exp | NG.N9133US2.M | /move/expc/data/ |
| R | Net Wdraw | NG.N5070US2.M | /stor/sum/data/ |
| S | Work Gas | NG.N5020US2.M | /stor/sum/data/ |

**API endpoint pattern:**
```
https://api.eia.gov/v2/natural-gas/{ENDPOINT}?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={START_DATE}
```

### B. Storage (L48 and AK)

- T: L48 Storage — use the L48 working gas in storage series from the storage endpoint.
- U: AK Storage — use the Alaska working gas in storage series. If unavailable, use the difference between the US total working gas (S) and the L48 value (T).

### C. Henry Hub Spot Price (daily, average to monthly)

Fetch daily Henry Hub spot prices:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={START_DATE}
```

For each month, compute the **simple average of all trading-day prices** within that calendar month. Round to 2 decimal places.

---

## Step 3: Format Results

Build a table with these exact columns in this order. Each row represents one calendar month.

| Column | Header | Format |
|--------|--------|--------|
| B | Month | MMM-YY (e.g., Jan-26) |
| C | Days | Integer (days in that month) |
| D | Dry Prod | Integer, MMCF |
| E | Pipe Imp CA | Integer, MMCF |
| F | Pipe Imp MX | Integer, MMCF |
| G | LNG Imp | Integer, MMCF |
| H | Residential | Integer, MMCF |
| I | Commercial | Integer, MMCF |
| J | Industrial | Integer, MMCF |
| K | Electric | Integer, MMCF |
| L | Vehicle | Integer, MMCF |
| M | Lease-Plant | Integer, MMCF |
| N | Pipeline Use | Integer, MMCF |
| O | Pipe Exp CA | Integer, MMCF |
| P | Pipe Exp MX | Integer, MMCF |
| Q | LNG Exp | Integer, MMCF |
| R | Net Wdraw | Integer, MMCF |
| S | Work Gas | Integer, MMCF |
| T | L48 Stor | Integer, MMCF |
| U | AK Stor | Integer, MMCF |
| V | HH Price | 2 decimal places, $/MMBtu |

Sort rows in **ascending chronological order** (oldest new month first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through V, starting at the first empty row after the existing data:**
>
> | Month | Days | Dry Prod | Pipe Imp CA | Pipe Imp MX | LNG Imp | Residential | Commercial | Industrial | Electric | Vehicle | Lease-Plant | Pipeline Use | Pipe Exp CA | Pipe Exp MX | LNG Exp | Net Wdraw | Work Gas | L48 Stor | AK Stor | HH Price |
> | Dec-25 | 31 | X,XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | XXX,XXX | X,XXX,XXX | X,XXX,XXX | XX,XXX | X.XX |

Include all new months since the last month in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue CONVERTED columns (W-AO) will auto-compute the BCF/d conversions using `MMCF / 1000 / Days`, and the gray COMPUTED columns (AP-AT) will auto-compute Total Imports, Total Exports, Net Imports, Total Demand, and Total Supply from those converted values.

---

## Notes

- **ALL input values must be in MMCF** (million cubic feet). The blue formula layer handles BCF/d conversion automatically.
- Conversion formula used by the spreadsheet: `BCF/d = MMCF / 1000 / Days in Month`.
- Stock items (Working Gas, L48 Storage, AK Storage) convert as: `BCF = MMCF / 1000` (not divided by days — they are point-in-time levels, not flows).
- HH Price passes through unchanged as $/MMBtu.
- The Monthly Balance tab pulls from the CONVERTED columns via INDEX-MATCH — do not edit that tab.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- EIA monthly data typically has a **2-month lag** (e.g., in February you may only have data through November or December).
- If a series returns no new data while others do, note which series are lagging and leave those cells blank. The user can fill them when the data becomes available.
- If you cannot fetch a data source, note which one failed. Do not substitute estimated values.
- Column C (Days in Month) must be hardcoded: Jan=31, Feb=28 or 29, Mar=31, Apr=30, May=31, Jun=30, Jul=31, Aug=31, Sep=30, Oct=31, Nov=30, Dec=31. Check for leap years.
- Not all 16 EIA series update at the same time. Production and consumption data often lag trade and storage data by an extra month. Return whatever months have data available and note any gaps.
