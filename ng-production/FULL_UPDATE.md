# NG Production Model — Full Update Instructions

You are an AI assistant helping a user update their Natural Gas Production Model spreadsheet with the latest monthly production data at national, basin, and state levels. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest monthly US marketed and dry gas production, Henry Hub prices, DPR basin-level production for 7 key basins, and state-level marketed production for the top 10 producing states, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Month** (YYYY-MM format).
2. Read the last 3-4 rows across columns B through V to confirm data continuity and verify the most recent values:
   - B: Month (YYYY-MM)
   - C: US Marketed (MMCF)
   - D: US Dry (MMCF)
   - E: HH Price ($/MMBtu)
   - F-L: DPR Basins — Total NG (Mcf/d): F=Anadarko, G=Appalachia, H=Bakken, I=Eagle Ford, J=Haynesville, K=Niobrara, L=Permian
   - M-V: States — Marketed (MMCF): M=TX, N=PA, O=LA, P=NM, Q=WV, R=OK, S=CO, T=WY, U=OH, V=ND
3. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. US Marketed and Dry Gas Production (2 series)

Use the EIA API v2:

| Column | Measure | Series ID |
|--------|---------|-----------|
| C | US Marketed Production | NG.N9050US2.M |
| D | US Dry Production | NG.N9070US2.M |

**API endpoint:**
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}-01
```

### B. Henry Hub Spot Price (daily, average to monthly)

Fetch daily Henry Hub spot prices from EIA:
```
https://api.eia.gov/v2/natural-gas/pri/fut/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=daily&data[0]=value&facets[series][]=RNGWHHD&start={LAST_MONTH}-01
```

Compute the **average of all trading-day prices** within each calendar month to produce column E.

### C. DPR Basin-Level Production (7 basins)

Download the Drilling Productivity Report data file:
```
https://www.eia.gov/petroleum/drilling/xls/dpr-data.xlsx
```

Open the file and navigate to the natural gas production tab. Extract monthly total NG production (Mcf/d) for each basin:

| Column | Basin |
|--------|-------|
| F | Anadarko |
| G | Appalachia |
| H | Bakken |
| I | Eagle Ford |
| J | Haynesville |
| K | Niobrara |
| L | Permian |

DPR data is reported as **Mcf/d** (thousand cubic feet per day). Use the values as-is — do not convert units.

### D. State-Level Marketed Production (10 states)

Use the EIA API v2 for each state. The series ID format is `NG.N9050{XX}2.M` where `{XX}` is the state code:

| Column | State | Series ID |
|--------|-------|-----------|
| M | Texas | NG.N9050TX2.M |
| N | Pennsylvania | NG.N9050PA2.M |
| O | Louisiana | NG.N9050LA2.M |
| P | New Mexico | NG.N9050NM2.M |
| Q | West Virginia | NG.N9050WV2.M |
| R | Oklahoma | NG.N9050OK2.M |
| S | Colorado | NG.N9050CO2.M |
| T | Wyoming | NG.N9050WY2.M |
| U | Ohio | NG.N9050OH2.M |
| V | North Dakota | NG.N9050ND2.M |

**API endpoint for each state:**
```
https://api.eia.gov/v2/natural-gas/prod/sum/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=value&facets[series][]={SERIES_ID}&start={LAST_MONTH}-01
```

---

## Step 3: Format Results

Build a table with these exact columns in this order:

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM |
| C | US Marketed | Integer, MMCF |
| D | US Dry | Integer, MMCF |
| E | HH Price | 2 decimal places, $/MMBtu |
| F | Anadarko | Integer, Mcf/d |
| G | Appalachia | Integer, Mcf/d |
| H | Bakken | Integer, Mcf/d |
| I | Eagle Ford | Integer, Mcf/d |
| J | Haynesville | Integer, Mcf/d |
| K | Niobrara | Integer, Mcf/d |
| L | Permian | Integer, Mcf/d |
| M | TX | Integer, MMCF |
| N | PA | Integer, MMCF |
| O | LA | Integer, MMCF |
| P | NM | Integer, MMCF |
| Q | WV | Integer, MMCF |
| R | OK | Integer, MMCF |
| S | CO | Integer, MMCF |
| T | WY | Integer, MMCF |
| U | OH | Integer, MMCF |
| V | ND | Integer, MMCF |

Sort rows in **ascending chronological order** (oldest new month first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through V, starting at the first empty row after the existing data:**
>
> | Month | US Marketed | US Dry | HH Price | Anadarko | Appalachia | Bakken | Eagle Ford | Haynesville | Niobrara | Permian | TX | PA | LA | NM | WV | OK | CO | WY | OH | ND |
> | 2026-XX | X,XXX,XXX | X,XXX,XXX | X.XX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | XX,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX | X,XXX,XXX |

Include all new months since the last month in the spreadsheet.

**Handle different data lags gracefully:**
- **DPR data** (columns F-L) is typically available with a ~1 month lag and includes 2 months of forecasts.
- **National production** (columns C-D) has a ~2 month lag.
- **State production** (columns M-V) has a ~3-4 month lag.

If DPR data is available for months where national/state data is not yet released, include the DPR columns and leave the lagging columns blank. Note which months are incomplete so the user can fill them in later.

After pasting, the blue formula columns will auto-compute: Marketed BCF/d, Dry BCF/d, Shrinkage %, YoY Growth %, DPR Total, and DPR BCF/d.

---

## Notes

- **National production** (US Marketed, US Dry) is in **MMCF** (million cubic feet).
- **DPR basin data** is in **Mcf/d** (thousand cubic feet per day) — do not convert to MMCF.
- **State production** is in **MMCF** (million cubic feet), same units as national.
- The DPR Excel file (`dpr-data.xlsx`) contains multiple tabs. Look for the tab labeled "Natural Gas" or similar — it contains monthly production by basin.
- National data typically lags by ~2 months, DPR by ~1 month, and state data by ~3-4 months. This means some rows may have partial data across the columns.
- Data ordering must be **ascending chronological** (oldest first).
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- If the EIA API or DPR file is unavailable, note which source failed and provide whatever data you could successfully fetch.
- HH Price is the monthly average of daily Henry Hub spot prices — not the futures settlement.
- Shrinkage % (a blue formula column) = 1 - Dry/Marketed. Typical values are 8-12%.
