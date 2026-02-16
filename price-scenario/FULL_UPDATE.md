# Price Scenario Analysis — Full Update Instructions

You are an AI assistant helping a user update their Price Scenario Analysis spreadsheet with the latest natural gas price data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return tables ready to paste.

## What This Does

Downloads the latest daily Henry Hub spot price and monthly state citygate prices from the EIA API, then returns new rows formatted to paste directly into the Historical Data tab. The spreadsheet has two data sections: a daily HH section and a monthly citygate section.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. This tab has TWO data sections on the same sheet:

### A. Daily Price Section (top, starting at row 7)
1. Read column B to find the **last Date** (YYYY-MM-DD, trading days only).
2. Read the last 3-4 rows across columns B-C to confirm data continuity:
   - B: Date (YYYY-MM-DD)
   - C: HH Spot ($/MMBtu)
3. Record the last daily date.

### B. Monthly Summary Section (below the daily section)
1. Scroll down to find the monthly section (header row says "MONTHLY PRICE SUMMARY").
2. Read the last 2-3 rows to find the **last Month** (YYYY-MM format or first of month date).
3. Read across the columns to confirm the layout:
   - B: Month
   - C: HH Avg ($/MMBtu)
   - D: CA Citygate ($/MCF)
   - E: MA Citygate ($/MCF)
   - F: IL Citygate ($/MCF)
   - G: NY Citygate ($/MCF)
   - H: TX Citygate ($/MCF)
   - I: PA Citygate ($/MCF)
4. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. Daily HH Spot Price

Fetch Henry Hub daily spot for all trading days after the last daily date:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_DAILY_DATE}&sort[0][column]=period&sort[0][direction]=asc
```

### B. Monthly Citygate Prices (6 states)

Fetch each state's citygate price for all months after the last month. Replace `{LAST_MONTH}` with the first day of the month after the last month in the spreadsheet (e.g., if last month is 2025-11, use `2025-12-01`).

| Column | State | Hub Proxy | Series ID |
|--------|-------|-----------|-----------|
| D | California | SoCal Citygate | N3050CA3 |
| E | Massachusetts | Algonquin Citygate | N3050MA3 |
| F | Illinois | Chicago Citygate | N3050IL3 |
| G | New York | Transco Z6 | N3050NY3 |
| H | Texas | Waha | N3050TX3 |
| I | Pennsylvania | Dominion South | N3050PA3 |

**API endpoint for each citygate series:**
```
https://api.eia.gov/v2/natural-gas/pri/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}&sort[0][column]=period&sort[0][direction]=asc
```

### C. HH Monthly Average

For each new month, compute the **average** of all daily HH Spot values in that month. This goes in column C of the monthly section.

---

## Step 3: Format Results

### Table 1: Daily HH Rows

| Column | Header | Format |
|--------|--------|--------|
| B | Date | YYYY-MM-DD |
| C | HH Spot | 3 decimal places, $/MMBtu |

Sort in **ascending chronological order** (oldest first).

### Table 2: Monthly Citygate Rows

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM-01 |
| C | HH Avg | 2 decimal places, $/MMBtu (average of daily HH for the month) |
| D | CA | 2 decimal places, $/MCF |
| E | MA | 2 decimal places, $/MCF |
| F | IL | 2 decimal places, $/MCF |
| G | NY | 2 decimal places, $/MCF |
| H | TX | 2 decimal places, $/MCF |
| I | PA | 2 decimal places, $/MCF |

Sort in **ascending chronological order** (oldest first).

---

## Step 4: Return Results

Return BOTH tables:

> **Table 1 — Paste into Historical Data tab, columns B-C, starting at the first empty row in the daily section:**
>
> | Date | HH Spot |
> | 2026-XX-XX | X.XXX |

> **Table 2 — Paste into Historical Data tab, columns B-I, starting at the first empty row in the monthly section:**
>
> | Month | HH Avg | CA | MA | IL | NY | TX | PA |
> | 2026-XX-01 | X.XX | X.XX | X.XX | X.XX | X.XX | X.XX | X.XX |

Include all new data since the last dates in the spreadsheet. If there is no new data available, say so explicitly.

After pasting:
- **Daily section**: Blue formula columns (D: Log Return, E: Monthly Vol, F: Ann Vol) auto-compute.
- **Monthly section**: Blue formula columns (J-O: Basis = Citygate - HH for each state) auto-compute. Volatility and STEO columns also auto-compute.
- The Price Tracker tab references these values via INDEX-MATCH and will update automatically.

---

## Notes

- HH Spot prices are in **$/MMBtu**. Citygate prices are in **$/MCF** (approximately equivalent to $/MMBtu).
- Daily data covers trading days only — skip weekends and holidays.
- Monthly citygate data from EIA typically lags 2-3 months. If a recent month is not yet available, note which months are missing and provide what you can.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 7 on both sections; do not overwrite the header rows.
- The 6 citygate states map to major NG trading hubs: CA=SoCal, MA=Algonquin, IL=Chicago, NY=Transco Z6, TX=Waha, PA=Dominion South.
- Basis columns (Citygate - HH) show the regional premium or discount relative to Henry Hub. Positive = premium, negative = discount.
- If you cannot fetch a data source, note which one failed and provide the data you were able to retrieve.
