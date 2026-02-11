# Portfolio Risk Model — Full Update Instructions

You are an AI assistant helping a user update their Portfolio Risk Model spreadsheet with the latest daily natural gas price data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest daily Henry Hub spot and NYMEX futures settlement prices (C1-C4) from the EIA API, then returns new rows formatted to paste directly into the Historical Prices tab. The Risk Dashboard uses these prices and their log returns for VaR, drawdown, and correlation calculations.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Prices** tab (not Historical Data — this product uses a different tab name). Data starts at row 7.

1. Read column B to find the **last Date** (YYYY-MM-DD, trading days only).
2. Read the last 3-4 rows across columns B through G to confirm data continuity and verify the most recent values:
   - B: Date (YYYY-MM-DD)
   - C: HH Spot ($/MMBtu)
   - D: C1 ($/MMBtu)
   - E: C2 ($/MMBtu)
   - F: C3 ($/MMBtu)
   - G: C4 ($/MMBtu)
3. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

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

### Alternative Source: CME Settlements

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
| D | C1 | 3 decimal places, $/MMBtu |
| E | C2 | 3 decimal places, $/MMBtu |
| F | C3 | 3 decimal places, $/MMBtu |
| G | C4 | 3 decimal places, $/MMBtu |

Sort rows in **ascending chronological order** (oldest new day first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Prices tab, columns B through G, starting at the first empty row after the existing data:**
>
> | Date | HH Spot | C1 | C2 | C3 | C4 |
> | 2026-XX-XX | X.XXX | X.XXX | X.XXX | X.XXX | X.XXX |

Include all new trading days since the last date in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns (H-L: log returns for each price series) will auto-compute from the new gold-column values:
- H: HH Ret = LN(C/C[-1])
- I: C1 Ret = LN(D/D[-1])
- J: C2 Ret = LN(E/E[-1])
- K: C3 Ret = LN(F/F[-1])
- L: C4 Ret = LN(G/G[-1])

The Risk Dashboard tab references these prices and returns for VaR, drawdown, and correlation matrix calculations.

---

## Notes

- All prices are in **$/MMBtu** (dollars per million British thermal units).
- Daily data covers trading days only — skip weekends and holidays.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Prices tab layout.
- Data starts at row 7 on the Historical Prices tab; do not overwrite the header rows (1-6).
- All five price series (HH Spot, C1-C4) should have values for each trading day. If HH Spot is missing for a day but NYMEX contracts are available, note the gap — the log return formula will produce an error for that row.
- Log returns require the prior day's price to compute. The first new row's return formula will reference the last existing row's price, so there must be no gap in dates.
- If you cannot fetch a data source, note which one failed and provide the data you were able to retrieve.
- The Risk Dashboard depends on complete, continuous daily data for accurate VaR and correlation calculations. Gaps will distort the risk metrics.
