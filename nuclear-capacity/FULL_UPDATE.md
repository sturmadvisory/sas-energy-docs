# Nuclear Capacity Forecast — Full Update Instructions

You are an AI assistant helping a user update their Nuclear Capacity Forecast spreadsheet with the latest monthly EIA nuclear generation data. **Execute all steps below automatically.** Do not ask the user to do anything — fetch the data yourself, compute the results, and return a table ready to paste.

## What This Does

Downloads the latest monthly US nuclear generation data from the EIA API, then returns new rows formatted to paste directly into the Historical Data tab.

---

## Step 1: Read From the Spreadsheet

Open the attached spreadsheet and go to the **Historical Data** tab. Data starts at row 6.

1. Read column B to find the **last Month** (YYYY-MM format).
2. Read the last 3-4 rows across columns B through D to confirm data continuity and verify the most recent values:
   - B: Month (YYYY-MM)
   - C: Nuclear Gen (GWh)
   - D: Days in Month
3. Record the last month — you will only fetch data **after** this month.

---

## Step 2: Fetch Fresh Data

### A. Monthly Nuclear Generation (EIA API)

Fetch monthly US nuclear generation from the EIA electricity API:
```
https://api.eia.gov/v2/electricity/electric-power-operational-data/data/?api_key=ZF0jbbLU5I2AchOTVr3idq6yya134laoiIyNPgsc&frequency=monthly&data[0]=generation&facets[fueltypeid][]=NUC&facets[location][]=US&facets[sectorid][]=99&start={START_DATE}
```

Replace `{START_DATE}` with the first day of the month after the last row (e.g., if last row is 2025-10, use `2025-11`).

The returned `generation` value is in **thousand megawatt-hours**. Convert to **GWh** by dividing by 1,000 (since 1 GWh = 1,000 MWh = 1 thousand MWh). Check the API response units carefully — if the value is already in GWh, no conversion is needed.

### B. NRC Reactor Status (supplementary check)

Optionally check the NRC reactor status page for context on recent outages or restarts:
```
https://www.nrc.gov/reading-rm/doc-collections/event-status/reactor-status/
```

This is for validation only — do not use NRC data as a substitute for EIA generation figures. If EIA data shows an unusual month (very low generation), check NRC for explanations (major outages, refueling seasons).

---

## Step 3: Format Results

Build a table with these exact columns in this order. Each row represents one calendar month.

| Column | Header | Format |
|--------|--------|--------|
| B | Month | YYYY-MM |
| C | Nuclear Gen | 1 decimal place, GWh |
| D | Days | Integer (days in that month) |

Sort rows in **ascending chronological order** (oldest new month first, newest last).

---

## Step 4: Return Results

Return the data as a paste-ready table:

> **Paste into Historical Data tab, columns B through D, starting at the first empty row after the existing data:**
>
> | Month | Nuclear Gen | Days |
> | 2025-XX | XX,XXX.X | XX |

Include all new months since the last month in the spreadsheet. If there is no new data available, say so explicitly.

After pasting, the blue formula columns will auto-compute:
- E: Capacity Factor = `C * 1000 / (95523 * D * 24)`
- F: YoY Change GWh = `C - C[12 months prior]`
- G: YoY Change % = `F / C[12 months prior]`
- H: Gas Displacement BCF/d = `F * 1000 * 7000 * 24 / (1e12 * D)`

And the gray computed columns will auto-compute:
- I: Rolling 12-mo Avg Gen GWh
- J: Offline MW = `95523 - (C * 1000 / (D * 24))`

---

## Notes

- Nuclear generation is in **GWh** (gigawatt-hours).
- US nuclear fleet capacity: **95,523 MW** across 93 reactors (as of 2024). This is the denominator for capacity factor calculations.
- Heat rate assumption: **7,000 BTU/kWh** for gas displacement calculations. This represents the efficiency of a natural gas combined-cycle plant that would replace lost nuclear generation.
- **Positive gas displacement** means more gas burn is needed (nuclear generation dropped year-over-year, so gas plants must compensate).
- **Negative gas displacement** means less gas burn is needed (nuclear generation increased year-over-year, displacing gas-fired generation).
- Data ordering must be **ascending chronological** (oldest first) — this matches the Historical Data tab layout.
- Data starts at row 6 on the Historical Data tab; do not overwrite the header rows (1-5).
- EIA electricity data typically has a **2-month lag** (e.g., in February you may only have data through November or December).
- Column D (Days in Month) must be hardcoded: Jan=31, Feb=28 or 29, Mar=31, Apr=30, May=31, Jun=30, Jul=31, Aug=31, Sep=30, Oct=31, Nov=30, Dec=31. Check for leap years.
- Nuclear generation follows a seasonal pattern: lower in spring and fall (refueling outages), higher in summer and winter (peak demand). A month with unusually low generation may simply reflect a heavy refueling season rather than an unexpected outage.
- If the EIA API returns no new data, report that clearly. The user may be checking before the latest month is published.
- If you cannot fetch the data, note the failure. Do not substitute estimated values.
