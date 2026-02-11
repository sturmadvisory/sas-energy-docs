# Trading P&L Tracker — Full Update Instructions

You are an AI assistant helping a user update their Trading P&L Tracker spreadsheet with the latest NYMEX natural gas futures settlement prices. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest daily NYMEX NG futures settlement prices for all listed contracts from CME, then returns new columns formatted to paste directly into the HHub Price Data tab. This product uses a grid layout (contracts as rows, dates as columns) rather than standard time-series rows.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **HHub Price Data** tab (not Historical Data — this product uses a different tab name). Data starts at row 7.

1. Read **column B** (rows 7 onward) to get the full list of futures contract labels (e.g., NGH26, NGJ26, NGK26, ... NGZ28). Record every contract in order.
2. Read **row 6** across all populated columns (C onward) to find all existing trading dates. Identify the **last trading date** (rightmost populated column header).
3. Count the total number of existing date columns to confirm where the next empty column starts.
4. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

Fetch NYMEX NG futures settlement prices for **all contracts listed in column B** for every trading day after the last date in the spreadsheet through today.

### Primary Source: CME Group

```
https://www.cmegroup.com/markets/energy/natural-gas/natural-gas.settlements.html
```

Navigate to the settlements page and retrieve the daily settlement price for each contract. CME publishes final settlements after the close of each trading day.

### Alternative Source: Yahoo Finance

If CME data is unavailable, use Yahoo Finance historical data for NYMEX NG continuous front-month contracts as a cross-reference.

### Contract Month Codes

Contracts follow CME convention for the month letter:
- F = January, G = February, H = March, J = April, K = May, M = June
- N = July, Q = August, U = September, V = October, X = November, Z = December

The two-digit suffix is the year (e.g., H26 = March 2026, Z28 = December 2028).

---

## Step 3: Format Results

Build a grid table where:
- The first column contains contract labels matching column B exactly
- Each additional column represents one new trading date
- Column headers are dates in YYYY-MM-DD format
- Cell values are settlement prices

| Contract | 2026-XX-XX | 2026-XX-XX | ... |
|----------|-----------|-----------|-----|
| NGH26    | X.XXX     | X.XXX     | ... |
| NGJ26    | X.XXX     | X.XXX     | ... |
| NGK26    | X.XXX     | X.XXX     | ... |
| ...      | ...       | ...       | ... |

Sort columns in **ascending chronological order** (oldest new date on the left, newest on the right).

---

## Step 4: Return Results

Return the data as a paste-ready grid:

> **Paste into HHub Price Data tab, starting at the next empty column after the last date, with dates in row 6 and prices in rows 7 onward:**
>
> | Contract | [Date1] | [Date2] | ... |
> | NGH26    | 3.456   | 3.478   | ... |
> | NGJ26    | 3.512   | 3.534   | ... |

Include all new trading days since the last date in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the **Trade Blotter** tab uses VLOOKUP to find settlement prices from the HHub Price Data tab for mark-to-market P&L calculations. Ensure contract labels match exactly.

---

## Notes

- Settlement prices in **$/MMBtu** with 3 decimal places.
- Trading days only — skip weekends and holidays.
- Date columns must be in **ascending chronological order** (left to right).
- Contract labels must match column B exactly (e.g., "NGH26" not "NG H26" or "NGH2026").
- Expired contracts will show no new data — leave those cells blank if the contract has already settled/expired.
- If a contract is not yet listed on CME (far-dated), leave those cells blank.
- The grid typically contains ~34 contracts spanning roughly 3 years forward.
- Row 6 contains the date headers; rows 7 onward contain settlement prices aligned to the contracts in column B.
- If you cannot fetch a data source, note which one failed. Partial data (some dates but not all) is still useful — return what you have.
- The Trade Blotter tab depends on this price grid for live P&L, so missing dates will cause #N/A errors in the blotter.
