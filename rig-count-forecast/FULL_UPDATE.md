# Rig Count Forecast — Full Update Instructions

You are an AI assistant helping a user update their Rig Count Forecast spreadsheet with the latest weekly Baker Hughes rig count data and Henry Hub spot prices. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest weekly US natural gas rig counts by basin from Baker Hughes and the Henry Hub spot price from the EIA API, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Week Ending Date** (YYYY-MM-DD, Friday dates).
2. Read the last 3-4 rows across columns B through L to confirm data continuity and verify the most recent values:
   - B: Week Ending Date
   - C: Total US Gas Rigs
   - D: Permian
   - E: Eagle Ford
   - F: Haynesville
   - G: Marcellus
   - H: DJ-Niobrara
   - I: Cana Woodford
   - J: Barnett
   - K: Utica
   - L: HH Spot Price ($/MMBtu)
3. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

### A. Baker Hughes Rig Count (weekly, Fridays at 1 PM ET)

Fetch the latest US gas rig counts from Baker Hughes. Try these approaches in order:

1. **Primary — Baker Hughes website:** Navigate to `https://bakerhughes.com/rig-count` and locate the downloadable pivot table Excel file. In the "Pivottable" worksheet, filter by `DrillFor = Gas` and extract weekly totals for each basin.

2. **Fallback — Web search:** Search for the latest Baker Hughes rig count data. Look for the most recent Friday release. Key basins to extract:
   - Total US Gas Rigs (sum of all gas-directed rigs)
   - Permian
   - Eagle Ford
   - Haynesville
   - Marcellus
   - DJ-Niobrara
   - Cana Woodford
   - Barnett
   - Utica

3. **Minimum viable data:** If basin-level detail is unavailable, provide at minimum the **Total US Gas Rigs** count and the **HH Spot Price**. Leave basin columns blank and note the gap.

### B. Henry Hub Spot Price (daily, average to weekly)

Fetch daily Henry Hub spot prices from the EIA API:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_DATE}
```

For each report week (Friday week-ending date), compute the **average of daily prices** from the preceding Monday through Friday. If some days are missing (holidays), average the available days. Round to 2 decimal places.

---

## Step 3: Format Results

Build a table with these exact columns in this order. Each row represents one week (Friday week-ending date).

| Column | Header | Format |
|--------|--------|--------|
| B | Week Ending | YYYY-MM-DD (Friday) |
| C | Total US Gas Rigs | Integer |
| D | Permian | Integer |
| E | Eagle Ford | Integer |
| F | Haynesville | Integer |
| G | Marcellus | Integer |
| H | DJ-Niobrara | Integer |
| I | Cana Woodford | Integer |
| J | Barnett | Integer |
| K | Utica | Integer |
| L | HH Spot Price | 2 decimal places, $/MMBtu |

Sort rows in **ascending chronological order** (oldest new week first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through L, starting at the first empty row after the existing data:**
>
> | Week Ending | Total US Gas Rigs | Permian | Eagle Ford | Haynesville | Marcellus | DJ-Niobrara | Cana Woodford | Barnett | Utica | HH Spot Price |
> | 2026-XX-XX | XXX | XX | XX | XX | XX | XX | XX | XX | XX | X.XX |

Include all new weeks since the last date in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns (M-P: WoW Change, YoY Change, 13-Wk MA, 52-Wk MA) and gray columns (Q-X: Basin % of Total) will auto-compute from the new gold-column values.

---

## Notes

- Baker Hughes counts individual **drilling rigs** actively exploring or developing natural gas. This is not production volume — it is a leading indicator of future production.
- Baker Hughes publishes every **Friday at 1 PM ET**. Data reflects the count as of that Friday.
- Basin breakdown: Permian, Eagle Ford, Haynesville, Marcellus, DJ-Niobrara, Cana Woodford, Barnett, Utica. The sum of these basins may be less than Total US Gas Rigs because some rigs operate outside these major basins.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- If Baker Hughes data is temporarily unavailable (e.g., holiday weeks), note the gap. Baker Hughes skips some weeks around major holidays (Christmas, New Year's).
- HH Spot Price is the weekly average of daily Henry Hub spot prices — not the futures settlement price.
- If you cannot fetch basin-level data, provide at minimum Total US Gas Rigs and HH Spot Price. Leave basin columns blank rather than guessing.
- If you cannot fetch a data source, note which one failed and explain why.
