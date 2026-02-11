# NG Weekly Storage Forecast — Full Update Instructions

You are an AI assistant helping a user update their Natural Gas Storage Forecast spreadsheet with the latest weekly EIA storage data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest weekly underground natural gas storage levels by region from the EIA API and the Henry Hub spot price, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Report Date** (YYYY-MM-DD, Friday week-ending dates).
2. Read the last 3-4 rows across columns B through K to confirm data continuity and verify the most recent values:
   - B: Report Date
   - C: Total L48 (BCF)
   - D: East (BCF)
   - E: Midwest (BCF)
   - F: South Central (BCF)
   - G: SC NonSalt (BCF)
   - H: SC Salt (BCF)
   - I: Mountain (BCF)
   - J: Pacific (BCF)
   - K: HH Cash ($/MMBtu)
3. Record the last date — you will only fetch data **after** this date.

---

## Step 2: Fetch Fresh Data

### A. Weekly Storage by Region (8 series)

Use the EIA API v2 to fetch each series for all weeks after the last date in the spreadsheet. Replace `{LAST_DATE}` with the date from Step 1 in YYYY-MM-DD format.

| Column | Region | Series ID |
|--------|--------|-----------|
| C | Total Lower 48 | NG.NW2_EPG0_SWO_R48_BCF.W |
| D | East | NG.NW2_EPG0_SWO_R31_BCF.W |
| E | Midwest | NG.NW2_EPG0_SWO_R32_BCF.W |
| F | South Central | NG.NW2_EPG0_SWO_R33_BCF.W |
| G | SC Non-Salt | NG.NW2_EPG0_SNO_R33_BCF.W |
| H | SC Salt | NG.NW2_EPG0_SSO_R33_BCF.W |
| I | Mountain | NG.NW2_EPG0_SWO_R34_BCF.W |
| J | Pacific | NG.NW2_EPG0_SWO_R35_BCF.W |

**API endpoint for each series:**
```
https://api.eia.gov/v2/natural-gas/stor/wkly/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=weekly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_DATE}
```

Alternatively, visit `https://www.eia.gov/naturalgas/storage/` and scrape the latest report if the API is unavailable.

### B. Henry Hub Spot Price (daily, average to weekly)

Fetch daily Henry Hub spot prices:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_DATE}
```

For each report week (Friday week-ending date), compute the **average of daily prices** from the preceding Monday through Friday. If some days are missing (holidays), average the available days.

---

## Step 3: Format Results

Align the storage data to the report date (Friday week-ending date as reported by EIA). Each row represents one weekly report.

Build a table with these exact columns in this order:

| Column | Header | Format |
|--------|--------|--------|
| B | Report Date | YYYY-MM-DD (Friday) |
| C | Total L48 | Integer, BCF |
| D | East | Integer, BCF |
| E | Midwest | Integer, BCF |
| F | South Central | Integer, BCF |
| G | SC NonSalt | Integer, BCF |
| H | SC Salt | Integer, BCF |
| I | Mountain | Integer, BCF |
| J | Pacific | Integer, BCF |
| K | HH Cash | 2 decimal places, $/MMBtu |

Sort rows in **ascending chronological order** (oldest new week first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through K, starting at the first empty row after the existing data:**
>
> | Report Date | Total L48 | East | Midwest | South Central | SC NonSalt | SC Salt | Mountain | Pacific | HH Cash |
> | 2026-XX-XX | X,XXX | XXX | XXX | XXX | XXX | XXX | XXX | XXX | X.XX |

Include all new weeks since the last date in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns (L-P: WoW Change, 5-Yr Avg Chg, Surprise, YoY Change, YoY %) and gray columns (Q-U: Regional percentages) will auto-compute from the new gold-column values.

---

## Notes

- All storage values are in **BCF** (billion cubic feet).
- Report dates are always Fridays (week-ending). EIA publishes reports on Thursdays at 10:30 AM ET with data through the prior Friday.
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- SC NonSalt + SC Salt should sum to approximately the South Central total.
- If the EIA API returns no new data (i.e., no reports after the last date), report that clearly. The user may be checking before Thursday's release.
- If you cannot fetch a data source, note which one failed and use the most recent available value from the spreadsheet as a fallback.
- HH Cash is the weekly average of daily Henry Hub spot prices — not the futures settlement price.
