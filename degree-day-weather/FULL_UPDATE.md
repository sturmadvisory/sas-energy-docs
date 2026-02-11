# Degree Day Weather Model — Full Update Instructions

You are an AI assistant helping a user update their Degree Day Weather Model spreadsheet with the latest weekly heating and cooling degree day data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest daily HDD and CDD data from NOAA CPC for CONUS and all 9 Census Divisions, aggregates them into weekly totals, fetches Henry Hub prices and EIA storage levels for the same weeks, and returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Week End Date** (YYYY-MM-DD).
2. Read the last 3-4 rows across columns B through Q to confirm data continuity and verify the most recent values:
   - B: Week End Date
   - C: CONUS HDD
   - D: CONUS CDD
   - E: HDD Normal (30-yr climatology)
   - F: CDD Normal (30-yr climatology)
   - G: HH Price ($/MMBtu)
   - H: Storage (BCF)
   - I-Q: Regional HDD by Census Division (Div 1 through Div 9)
3. Record the last date — you will only fetch data **after** this date.
4. Determine the week-ending convention used (typically Saturday or Sunday) by checking the existing date pattern.

---

## Step 2: Fetch Fresh Data

### A. Daily HDD and CDD from NOAA CPC

Download the current year's daily population-weighted degree day files:

**Heating Degree Days:**
```
ftp://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/{YYYY}/StatesCONUS.Heating.txt
```

**Cooling Degree Days:**
```
ftp://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/{YYYY}/StatesCONUS.Cooling.txt
```

Replace `{YYYY}` with the current year (and prior year if needed to cover the gap). These files contain daily HDD/CDD values for each state and CONUS total.

If the current year's file is unavailable via FTP, try the HTTP mirror:
```
https://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/{YYYY}/StatesCONUS.Heating.txt
https://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/{YYYY}/StatesCONUS.Cooling.txt
```

### B. 30-Year Normals

Download the climatological normals for comparison:
```
ftp://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/normals/StatesCONUS.Heating.txt
ftp://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/normals/StatesCONUS.Cooling.txt
```

### C. Aggregate Daily to Weekly

For each new week after the last date in the spreadsheet:
1. Sum daily CONUS HDD values for the 7-day period ending on the week end date. This gives column C.
2. Sum daily CONUS CDD values for the same 7-day period. This gives column D.
3. Sum daily normal HDD values for the same 7-day period. This gives column E.
4. Sum daily normal CDD values for the same 7-day period. This gives column F.
5. For columns I through Q, aggregate daily HDD by Census Division:

| Column | Census Division | States |
|--------|----------------|--------|
| I | Div 1 — New England | CT, ME, MA, NH, RI, VT |
| J | Div 2 — Mid-Atlantic | NJ, NY, PA |
| K | Div 3 — E.N. Central | IL, IN, MI, OH, WI |
| L | Div 4 — W.N. Central | IA, KS, MN, MO, NE, ND, SD |
| M | Div 5 — S. Atlantic | DE, DC, FL, GA, MD, NC, SC, VA, WV |
| N | Div 6 — E.S. Central | AL, KY, MS, TN |
| O | Div 7 — W.S. Central | AR, LA, OK, TX |
| P | Div 8 — Mountain | AZ, CO, ID, MT, NV, NM, UT, WY |
| Q | Div 9 — Pacific | CA, OR, WA |

Sum the state-level daily HDD values for each division, then sum across the 7-day week.

### D. Henry Hub Price (weekly average)

Fetch daily Henry Hub spot prices from EIA:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_DATE}
```

Average the daily prices for each week to produce column G.

### E. Weekly Storage Level

Fetch the EIA weekly storage total:
```
https://api.eia.gov/v2/natural-gas/stor/wkly/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=weekly&data[0]=value&facets[series][]=NG.NW2_EPG0_SWO_R48_BCF.W&start={LAST_DATE}
```

Match each storage report to the corresponding weather week for column H.

---

## Step 3: Format Results

Build a table with these exact columns in this order:

| Column | Header | Format |
|--------|--------|--------|
| B | Week End Date | YYYY-MM-DD |
| C | CONUS HDD | Integer |
| D | CONUS CDD | Integer |
| E | HDD Normal | Integer |
| F | CDD Normal | Integer |
| G | HH Price | 2 decimal places, $/MMBtu |
| H | Storage | Integer, BCF |
| I | Div 1 HDD | Integer |
| J | Div 2 HDD | Integer |
| K | Div 3 HDD | Integer |
| L | Div 4 HDD | Integer |
| M | Div 5 HDD | Integer |
| N | Div 6 HDD | Integer |
| O | Div 7 HDD | Integer |
| P | Div 8 HDD | Integer |
| Q | Div 9 HDD | Integer |

Sort rows in **ascending chronological order** (oldest new week first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through Q, starting at the first empty row after the existing data:**
>
> | Week End Date | CONUS HDD | CONUS CDD | HDD Normal | CDD Normal | HH Price | Storage | Div1 | Div2 | Div3 | Div4 | Div5 | Div6 | Div7 | Div8 | Div9 |
> | 2026-XX-XX | XXX | XX | XXX | XX | X.XX | X,XXX | XX | XX | XX | XX | XX | XX | XX | XX | XX |

Include all new complete weeks since the last date in the spreadsheet. If the current week is incomplete (fewer than 7 days of data available), exclude it and note the partial data situation.

After pasting, the blue formula columns will auto-compute deviation from normal, YoY comparisons, and other derived metrics.

---

## Notes

- **HDD** = max(0, 65F - Average Temp). **CDD** = max(0, Average Temp - 65F). Both are population-weighted.
- Census Divisions: 1=New England, 2=Mid-Atlantic, 3=E.N.Central, 4=W.N.Central, 5=S.Atlantic, 6=E.S.Central, 7=W.S.Central, 8=Mountain, 9=Pacific.
- Weekly values are the **sum of daily values** for the 7-day period ending on the week end date.
- Data ordering must be **ascending chronological** (oldest first).
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- NOAA CPC data typically has a 1-2 day lag. If the most recent days are missing, only include complete weeks.
- If FTP is unreachable, try the HTTP mirror at `https://ftp.cpc.ncep.noaa.gov/`.
- If you cannot fetch a specific data source (NOAA, EIA price, or EIA storage), note which one failed and leave that column blank for the user to fill manually.
- The NOAA file uses state abbreviations — map each state to its Census Division before summing.
