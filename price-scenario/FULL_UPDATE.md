# Price Scenario Analysis — Full Update Instructions

You are an AI assistant helping a user update their Price Scenario Analysis spreadsheet with the latest daily natural gas price data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest daily Henry Hub spot, NYMEX futures settlement prices (C1-C4), and monthly Citygate prices from the EIA API, then returns new rows formatted to paste directly into the daily price section of the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. This tab has TWO data sections — a daily price section at the top and a monthly summary section below. Focus on the **daily price section** (rows starting at row 7).

1. Read column B to find the **last Date** (YYYY-MM-DD, trading days only).
2. Read the last 3-4 rows across columns B through H to confirm data continuity and verify the most recent values:
   - B: Date (YYYY-MM-DD)
   - C: HH Spot ($/MMBtu)
   - D: NYMEX C1 ($/MMBtu)
   - E: NYMEX C2 ($/MMBtu)
   - F: NYMEX C3 ($/MMBtu)
   - G: NYMEX C4 ($/MMBtu)
   - H: Citygate ($/MMBtu) — only populated on the first trading day of each month
3. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

### A. Daily Prices (5 series)

Use the EIA API v2 to fetch each series for all trading days after the last date in the spreadsheet. Replace `{LAST_DATE}` with the date from Step 1 in YYYY-MM-DD format.

| Column | Series | Series ID |
|--------|--------|-----------|
| C | Henry Hub Spot | NG.RNGWHHD.D |
| D | NYMEX NG C1 | NG.RNGC1.D |
| E | NYMEX NG C2 | NG.RNGC2.D |
| F | NYMEX NG C3 | NG.RNGC3.D |
| G | NYMEX NG C4 | NG.RNGC4.D |

**API endpoint for each series:**
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_DATE}
```

### B. Citygate Price (monthly)

Fetch the monthly Citygate average price:
```
https://api.eia.gov/v2/natural-gas/pri/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]=N3050US3&start={LAST_DATE}
```

For each new month that appears in the daily data, populate column H with the Citygate price on the **first trading day of that month only**. Leave column H blank for all other rows within that month.

### C. CME Settlements (fallback)

If the EIA API is unavailable or has not yet published the latest day, check CME settlements:
```
https://www.cmegroup.com/markets/energy/natural-gas/natural-gas.settlements.html
```

---

## Step 3: Format Results

Build a table with these exact columns in this order. Each row represents one trading day.

| Column | Header | Format |
|--------|--------|--------|
| B | Date | YYYY-MM-DD |
| C | HH Spot | 3 decimal places, $/MMBtu |
| D | NYMEX C1 | 3 decimal places, $/MMBtu |
| E | NYMEX C2 | 3 decimal places, $/MMBtu |
| F | NYMEX C3 | 3 decimal places, $/MMBtu |
| G | NYMEX C4 | 3 decimal places, $/MMBtu |
| H | Citygate | 2 decimal places, $/MMBtu (first trading day of month only, blank otherwise) |

Sort rows in **ascending chronological order** (oldest new day first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through H, starting at the first empty row after the existing daily data:**
>
> | Date | HH Spot | NYMEX C1 | NYMEX C2 | NYMEX C3 | NYMEX C4 | Citygate |
> | 2026-XX-XX | X.XXX | X.XXX | X.XXX | X.XXX | X.XXX | X.XX |

Include all new trading days since the last date in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns (I: Basis, J: Daily Log Return) and gray computed columns (K: Monthly Vol, L: Annualized Vol) will auto-compute from the new gold-column values. The monthly summary section below the daily data also references these rows and will update automatically.

---

## Notes

- All prices are in **$/MMBtu** (dollars per million British thermal units).
- Daily data covers trading days only — skip weekends and holidays.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 7 on the Historical Data tab; do not overwrite the header rows (1-6).
- The Historical Data tab has TWO sections on the same sheet: daily prices at top, monthly aggregates below. Only paste into the daily section.
- Basis (col I) = HH Spot - NYMEX C1. If your HH Spot value is blank for a day but NYMEX is present, note the gap — the formula will produce an error.
- Citygate (col H) is a monthly value. Populate it only on the first trading day of each new month. If the monthly value is not yet available from EIA (it lags ~2 months), leave column H blank for that month.
- If you cannot fetch a data source, note which one failed and provide the data you were able to retrieve.
- The Price Tracker tab uses these daily prices for scenario analysis, so completeness matters — avoid gaps in the C1-C4 series.
