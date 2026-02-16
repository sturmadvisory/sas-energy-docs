# Portfolio Risk Model — Full Update Instructions

You are an AI assistant helping a user update their Portfolio Risk Model spreadsheet with the latest natural gas price data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest monthly Henry Hub average price and state citygate prices from the EIA API, then returns new rows formatted to paste directly into the Historical Prices tab. The Risk Dashboard uses these prices and their log returns for VaR, drawdown, and correlation calculations.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Prices** tab. Data starts at row 7. This tab uses **monthly** data (not daily).

1. Read column B to find the **last Month** (YYYY-MM-01 format).
2. Read the last 3-4 rows across columns B through H to confirm data continuity and verify the most recent values:
   - B: Month (YYYY-MM-01)
   - C: HH Avg ($/MMBtu — monthly average of daily Henry Hub spot)
   - D: CA ($/MCF — California citygate)
   - E: MA ($/MCF — Massachusetts citygate)
   - F: IL ($/MCF — Illinois citygate)
   - G: NY ($/MCF — New York citygate)
   - H: TX ($/MCF — Texas citygate)
3. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. Daily HH Spot (to compute monthly average)

Fetch Henry Hub daily spot for all trading days after the last month:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={FIRST_DAY_OF_NEXT_MONTH}&sort[0][column]=period&sort[0][direction]=asc
```

Compute the **monthly average** of HH Spot for each complete month. Do not include a partial current month unless the user requests it.

### B. Monthly Citygate Prices (5 states)

Fetch each state's citygate price for all months after the last month in the spreadsheet.

| Column | State | Hub Proxy | Series ID |
|--------|-------|-----------|-----------|
| D | California | SoCal Citygate | N3050CA3 |
| E | Massachusetts | Algonquin Citygate | N3050MA3 |
| F | Illinois | Chicago Citygate | N3050IL3 |
| G | New York | Transco Z6 | N3050NY3 |
| H | Texas | Waha | N3050TX3 |

**API endpoint for each citygate series:**
```
https://api.eia.gov/v2/natural-gas/pri/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={FIRST_DAY_OF_NEXT_MONTH}&sort[0][column]=period&sort[0][direction]=asc
```

---

## Step 3: Format Results

Build a table with these exact columns in this order. Each row represents one month.

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM-01 |
| C | HH Avg | 2 decimal places, $/MMBtu (average of daily HH for the month) |
| D | CA | 2 decimal places, $/MCF |
| E | MA | 2 decimal places, $/MCF |
| F | IL | 2 decimal places, $/MCF |
| G | NY | 2 decimal places, $/MCF |
| H | TX | 2 decimal places, $/MCF |

Sort rows in **ascending chronological order** (oldest month first).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Prices tab, columns B through H, starting at the first empty row after the existing data:**
>
> | Month | HH Avg | CA | MA | IL | NY | TX |
> | 2026-XX-01 | X.XX | X.XX | X.XX | X.XX | X.XX | X.XX |

Include all new complete months since the last month in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns auto-compute:
- I-M: Basis = Citygate - HH for each state (positive = premium to HH)
- N-S: Monthly log returns = LN(price / prior month price) for all 6 series

The Risk Dashboard tab references these prices and returns for VaR, drawdown, and correlation matrix calculations.

---

## Notes

- HH Avg is in **$/MMBtu**. Citygate prices are in **$/MCF** (approximately equivalent to $/MMBtu).
- This tab uses **monthly** data — one row per month, not daily.
- Monthly citygate data from EIA typically lags 2-3 months. If a recent month is not yet available, note which months are missing and provide what you can.
- Only include **complete** months. Do not include a partial current month unless specifically requested.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Prices tab layout.
- Data starts at row 7; do not overwrite the header rows (1-6).
- The 5 citygate states map to major NG trading hubs: CA=SoCal, MA=Algonquin, IL=Chicago, NY=Transco Z6, TX=Waha.
- Basis columns show the regional premium or discount relative to Henry Hub. Positive = premium (pipeline constraints, local demand), negative = discount (oversupply, takeaway constraints like Waha).
- Log returns require the prior month's price to compute. The first new row's return formula will reference the last existing row's price, so there must be no gap in months.
- If you cannot fetch a data source, note which one failed and provide the data you were able to retrieve.
- The Risk Dashboard depends on complete, continuous monthly data for accurate VaR and correlation calculations. Gaps will distort the risk metrics.
