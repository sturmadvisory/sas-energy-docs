# SAS Energy Analyst — Agent Reference

Master knowledge file for the SAS Energy Analyst GPT. This document covers everything the per-product FULL_UPDATE.md files do not: product suite overview, model methodologies, cross-product relationships, NG market fundamentals, and complete data source reference.

---

## 1. Product Suite Overview

SAS Energy Analytics is a suite of 10 Excel-based natural gas fundamental analysis models by Sturm Advisory Services LLC. Each product is a self-contained `.xlsx` workbook with gold input cells (user pastes data), blue formula cells (auto-calculate), and gray total cells.

| # | Product | Update Freq | Headline Metric | Formulas | Tab Layout |
|---|---------|-------------|-----------------|----------|------------|
| 1 | NG Storage Forecast | Weekly (Thu) | Storage surplus/deficit vs 5-yr avg (BCF) | 9,132 | 8-tab |
| 2 | Degree Day Weather | Weekly | CONUS HDD/CDD and implied gas demand (BCF/d) | 5,730 | 8-tab |
| 3 | Gas Burn Power Stack | Monthly | NG consumption by sector (MMCF) | 5,755 | 8-tab |
| 4 | NG Production & Nowcast | Monthly | US marketed production ex-Alaska (BCF/d) | 18,344 | 10-tab |
| 5 | NG Balance Report | Monthly | Total supply vs total demand balance (BCF/d) | 4,836 | 8-tab |
| 6 | Rig Count Forecast | Weekly (Fri) | US gas rig count by basin | 4,850 | 8-tab |
| 7 | Nuclear Capacity Forecast | Monthly | Nuclear generation and capacity factor (GWh) | 2,035 | 8-tab |
| 8 | Price Scenario Analysis | Daily | HH spot, NYMEX curve, basis, volatility | 7,559 | 8-tab |
| 9 | Trading PnL Tracker | Daily | Cumulative P&L on NYMEX NG positions | 252 | 8-tab |
| 10 | Portfolio Risk Model | Daily | VaR, drawdown, correlation across contracts | 2,715 | 8-tab |

### Pricing
- Individual products: $49 each
- Complete Bundle (all 10): $279
- Available on: Etsy, Amazon, Google

---

## 2. Standard Tab Architecture

### 8-Tab Layout (Products 1-3, 5-10)
1. **Instructions** — Product overview, tab guide, first-5-minutes walkthrough
2. **AI Workflow** — Copy-paste prompts (Full Update, Quick Update) + link to FULL_UPDATE.md
3. **Dashboard** — Charts and summary KPIs (all formulas, no user input)
4. **Main Data Tab** — Primary analysis grid with gold inputs and blue formulas
5. **Historical Data** — Gold input tab where users paste raw data from APIs
6. **Model Reference** — Methodology documentation, formula explanations, data source details
7. **Product Catalog** — Overview of all 10 products with AI automation levels
8. **About Author** — Fletcher Sturm bio and Sturm Advisory information

### 10-Tab Layout (Product 4 only — merged production + nowcast)
1. Instructions | 2. AI Workflow | 3. Dashboard
4. **Production Tracker** — 60-month production analysis (refs Production History)
5. **Weekly Estimates** — Mass balance nowcast formulas (refs Demand & Storage Data)
6. **Production History** — Gold input: monthly production, basins, states
7. **Demand & Storage Data** — Gold input: monthly demand + weekly storage
8. **Model Reference** — 12 sections (6 production + 6 nowcast)
9. Product Catalog | 10. About Author

### Common Conventions
- Data starts at **row 6** (rows 1-5 are headers/labels)
- Gold cells (`#FFF2CC`) = user input
- Blue cells (`#D6E4F0`) = formulas (never overwrite)
- Gray cells (`#F2F2F2`) = computed totals/subtotals
- Number format: right-justified, black font, red parentheses for negatives
- Change format: dark green with `+` for positive, red parentheses for negative

---

## 3. Per-Product Detail

### Product 1: NG Storage Forecast

**Purpose**: Track the EIA Weekly Natural Gas Storage Report — the single most market-moving data release in natural gas, published every Thursday at 10:30 AM ET.

**Main Tab (Storage Tracker)**: 60 weeks of storage data with formulas for WoW change, YoY comparison, 5-year average, surplus/deficit, and implied injection/withdrawal rates.

**Historical Data columns**: Report Date | Total L48 | East | Midwest | South Central | SC NonSalt | SC Salt | Mountain | Pacific | HH Price

**Regression Model**: Predicts the weekly storage surprise (actual vs consensus).
- 7 features + intercept, R² = 0.584, RMSE = 25.0 BCF
- Features include: prior week surprise, HDD deviation, cooling season flag, price momentum
- Full-season forecast projects storage through end of injection/withdrawal season

**Key EIA Series** (weekly storage, `/v2/natural-gas/stor/wkly/data/`):
- Total L48: `NG.NW2_EPG0_SWO_R48_BCF.W`
- East: `NG.NW2_EPG0_SWO_R31_BCF.W`
- Midwest: `NG.NW2_EPG0_SWO_R32_BCF.W`
- South Central: `NG.NW2_EPG0_SWO_R33_BCF.W`
- SC NonSalt: `NG.NW2_EPG0_SNO_R33_BCF.W`
- SC Salt: `NG.NW2_EPG0_SSO_R33_BCF.W`
- Mountain: `NG.NW2_EPG0_SWO_R34_BCF.W`
- Pacific: `NG.NW2_EPG0_SWO_R35_BCF.W`
- HH Spot: `RNGWHHD` (daily, average to weekly)

**Quick Update Prompt**: "Download only this week's EIA storage report (series NG.NW2_EPG0_SWO_R48_BCF.W and regional). Give me one row: Report Date, Total L48, East, Midwest, South Central, SC NonSalt, SC Salt, Mountain, Pacific, HH Price."

---

### Product 2: Degree Day Weather

**Purpose**: Track population-weighted heating and cooling degree days across 9 US Census Divisions and CONUS, with implied gas demand factors.

**Main Tab (Weather Tracker)**: Weekly HDD/CDD by division with deviation from 30-year normals, year-over-year comparisons, and demand impact estimates.

**Historical Data columns**: Week End Date | CONUS HDD | CONUS CDD | HDD Normal | CDD Normal | HH Price | Storage | Div 1-9 HDD

**NWP Forecast Model**: 199-city GFS operational forecast (00Z), 7-day horizon, weighted by Census ACS gas-heated households.
- Backtest: GEFSv12 reforecast vs nClimGrid, 2019 full year
- HDD Day 1 MAE: 1.7°F

**Demand Factors**: Monthly BCF/HDD and BCF/CDD conversion factors stored in `demand_factors.json`. Allow translation from degree days to implied gas demand in BCF/d.

**Key Data Sources**:
- NOAA CPC daily HDD/CDD: `https://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/{YYYY}/StatesCONUS.{Heating|Cooling}.txt`
- 30-year normals: `https://ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/normals/StatesCONUS.{Heating|Cooling}.txt`
- EIA HH price: `RNGWHHD` | Storage: `NG.NW2_EPG0_SWO_R48_BCF.W`

**Census Division Mapping**:
1. New England (CT, ME, MA, NH, RI, VT)
2. Mid-Atlantic (NJ, NY, PA)
3. E.N. Central (IL, IN, MI, OH, WI)
4. W.N. Central (IA, KS, MN, MO, NE, ND, SD)
5. S. Atlantic (DE, DC, FL, GA, MD, NC, SC, VA, WV)
6. E.S. Central (AL, KY, MS, TN)
7. W.S. Central (AR, LA, OK, TX)
8. Mountain (AZ, CO, ID, MT, NV, NM, UT, WY)
9. Pacific (CA, OR, WA)

**Quick Update Prompt**: "Download this week's NOAA population-weighted degree days from the CPC FTP server. Give me the last 7 days of daily HDD and CDD for all 9 Census Divisions plus CONUS total."

---

### Product 3: Gas Burn Power Stack

**Purpose**: Track natural gas consumption by sector (Electric Power, Residential, Commercial, Industrial) and NG-fired electricity generation.

**Main Tab (Gas Burn Tracker)**: Monthly consumption by sector with MoM and YoY changes, sector share analysis, and generation efficiency metrics.

**Historical Data columns**: Month | Elec Power (MMCF) | Residential (MMCF) | Commercial (MMCF) | Industrial (MMCF) | HH Price | NG Gen (000 MWh)

**Key EIA Series** (`/v2/natural-gas/cons/sum/data/`):
- Electric Power: `N3045US2`
- Residential: `N3010US2`
- Commercial: `N3020US2`
- Industrial: `N3035US2`
- NG Generation: `/v2/electricity/electric-power-operational-data/data/` (fueltypeid=NG, sectorid=99)

**Quick Update Prompt**: "Download the latest month's EIA natural gas consumption by sector (Electric, Residential, Commercial, Industrial) in MMCF. Give me one row with Month and the 4 sector values."

---

### Product 4: NG Production & Nowcast

**Purpose**: Comprehensive US natural gas production tracking (60-month history by national/basin/state) combined with a weekly mass-balance nowcast of current-month production.

**Production Tracker columns**: Month | US Marketed | US Dry | HH Price | 7 DPR Basins (Mcf/d) | 10 State Production (MMCF)

**Weekly Estimates columns**: Week Ending | Residential through Vehicle (7 demand sectors) | Total Demand | Storage Change | Net Imports | Implied Production | Mass Balance Avg | Nowcast Prod

**Nowcast Methodology**:
- **Target**: US Marketed Production ex-Alaska (N9050US2 - N9050AK2, ~120 BCF/d)
- **Monthly Regression**: R² = 0.9998, MAPE = 0.13%, RMSE = 0.25 BCF/d
- **9 Features**: mass balance average, DPR rigs (6-mo lag), DPR production, STEO marketed ex-AK, HH price, storage 5-year deviation, lagged production, winter dummy
- **STEO coefficient**: ~1.008 (primary anchor — STEO marketed ex-AK explains nearly all variance)
- **Forward forecast**: 24 months beyond mass balance window using STEO as mass-balance substitute
- **Alaska average**: ~1.01 BCF/d (last 12 months, subtracted from STEO marketed total)

**Mass Balance Formula**: Implied Production = Total Demand + (Storage Change / 7) - Net Imports

**Key EIA Series**:
- US Marketed: `N9050US2` | US Dry: `N9070US2` (production endpoint)
- State production: `N9050TX2`, `N9050PA2`, `N9050LA2`, `N9050NM2`, `N9050WV2`, `N9050OK2`, `N9050CO2`, `N9050WY2`, `N9050OH2`, `N9050ND2`
- Alaska: `N9050AK2` (for ex-Alaska calculation)
- DPR basin data: `https://www.eia.gov/petroleum/drilling/xls/dpr-data.xlsx`
- STEO marketed: `NGMPPUS` via `/v2/steo/data/` (uses `facets[seriesId][]`)
- STEO HH price: `NGHHUUS` (for forward forecast months)
- Demand series: Same 7 consumption + 6 trade series as P5

**Quick Update Prompt**: "Download the latest EIA DPR data for all 7 basins, national dry/marketed production from EIA bulk, and this week's EIA storage report (NG.NW2_EPG0_SWO_R48_BCF.W). Give me one row per data source with the most recent month/week."

---

### Product 5: NG Balance Report

**Purpose**: Complete US natural gas supply/demand balance sheet — the most comprehensive single view of the NG market.

**Main Tab (Monthly Balance) columns**: Month | Days | Dry Prod | Pipe Imp CA | Pipe Imp MX | LNG Imp | Residential | Commercial | Industrial | Electric | Vehicle | Lease-Plant | Pipeline | Pipe Exp CA | Pipe Exp MX | LNG Exp | Net Wdraw | Work Gas | L48 Stor | AK Stor | HH Price

**Key Formulas**:
- Total Supply = Dry Production + Total Imports (Pipe CA + Pipe MX + LNG)
- Total Demand = SUM(Residential through Pipeline) + Total Exports (Pipe CA + Pipe MX + LNG)
- BCF/d = MMCF / 1000 / Days in Month
- Storage is point-in-time: BCF = MMCF / 1000 (not divided by days)
- Forecast: 24 months forward using STEO data

**Key EIA Series** (19 monthly series across multiple endpoints):
- Dry Prod: `N9070US2` (prod/sum) | Pipe Imp CA: `N9102CN2` (move/impc) | Pipe Imp MX: `N9102MX2` | LNG Imp: `N9103US2`
- Residential: `N3010US2` (cons/sum) | Commercial: `N3020US2` | Industrial: `N3035US2` | Electric: `N3045US2`
- Vehicle: `N3025US2` | Lease-Plant: `N9160US2` (prod/sum) | Pipeline: `N9170US2` (cons/sum)
- Pipe Exp CA: `N9132CN2` (move/expc) | Pipe Exp MX: `N9132MX2` | LNG Exp: `N9133US2`
- Net Withdrawal: `N5070US2` (stor/sum) | Working Gas: `N5020US2`
- HH Price: `RNGWHHD` (pri/fut, daily averaged to monthly)

**Quick Update Prompt**: "Download the latest month of EIA bulk natural gas data for all 16 series listed in the EIA Series Reference. Give me one row with Month and all 16 values in MMCF."

---

### Product 6: Rig Count Forecast

**Purpose**: Track Baker Hughes weekly gas-directed rig counts by basin with Henry Hub price overlay for leading-indicator analysis.

**Main Tab (Rig Count Tracker) columns**: Week Ending | Total US Gas Rigs | Permian | Eagle Ford | Haynesville | Marcellus | DJ-Niobrara | Cana Woodford | Barnett | Utica | HH Spot Price

**Key Relationship**: Gas rig counts lead production by approximately 6 months. A sustained rig count increase signals future production growth.

**Data Sources**:
- Baker Hughes: `https://bakerhughes.com/rig-count` (pivot table Excel, filter DrillFor=Gas)
- Published Fridays at 1 PM ET (skips some holiday weeks)
- HH Price: `RNGWHHD` (EIA, daily averaged to weekly)

**Quick Update Prompt**: "Download this week's Baker Hughes North America gas rig count by basin from bakerhughes.com/rig-count, plus today's Henry Hub spot price from EIA. Give me one row with Week Ending, Total Gas Rigs, and basin breakdown."

---

### Product 7: Nuclear Capacity Forecast

**Purpose**: Track US nuclear generation and capacity factor to quantify gas displacement — when nuclear output drops, gas-fired generation fills the gap.

**Main Tab (Nuclear Tracker) columns**: Month | Nuclear Gen (GWh) | Days in Month

**Key Formulas**:
- US nuclear fleet capacity: 95,523 MW (93 reactors as of 2024)
- Capacity Factor = GWh * 1000 / (95,523 * Days * 24)
- Heat rate for gas displacement: 7,000 BTU/kWh
- Gas Displacement (BCF/d) = delta_GWh * 1000 * 7,000 * 24 / (1e12 * Days)

**Key Data Source**: EIA Electricity API: `/v2/electricity/electric-power-operational-data/data/` with `fueltypeid=NUC`, `location=US`, `sectorid=99`
- Returns generation in thousand MWh (divide by 1,000 for GWh)

**Quick Update Prompt**: "Download the latest month's US nuclear generation from EIA and check NRC reactor status at nrc.gov. Give me the monthly total GWh, number of reactors at power, and any notable outages."

---

### Product 8: Price Scenario Analysis

**Purpose**: Daily tracking of Henry Hub spot and NYMEX NG futures curve (C1-C4) with basis, volatility, and scenario analysis.

**Main Tab (Price Tracker) columns**: Date | HH Spot | NYMEX C1 | C2 | C3 | C4 | Citygate

**Key Formulas**:
- Basis = HH Spot - NYMEX C1
- Daily Returns = LN(Today / Yesterday)
- Annualized Volatility = Daily StdDev * SQRT(252)

**Key EIA Series** (`/v2/natural-gas/pri/fut/data/`, daily):
- HH Spot: `RNGWHHD` | C1: `RNGC1` | C2: `RNGC2` | C3: `RNGC3` | C4: `RNGC4`
- Citygate: `N3050US3` (monthly, `/v2/natural-gas/pri/sum/data/`)

**Quick Update Prompt**: "Download today's Henry Hub spot price (NG.RNGWHHD.D) and latest NYMEX NG settlements (C1-C4) from CME. Give me one row: Date, HH Spot, C1, C2, C3, C4."

---

### Product 9: Trading PnL Tracker

**Purpose**: Track NYMEX NG positions, settlement prices, and calculate cumulative P&L with commission accounting.

**Main Tab (Trade Blotter)**: Position grid with contract, direction, lots, entry/exit prices, P&L per trade, cumulative P&L.

**Historical Data (HHub Price Data)**: Transposed grid — rows are contract months (NGH26 through NGZ28), columns are trading dates, values are settlement prices.

**Contract Month Codes**: F=Jan, G=Feb, H=Mar, J=Apr, K=May, M=Jun, N=Jul, Q=Aug, U=Sep, V=Oct, X=Nov, Z=Dec

**Key Formulas**:
- Cumulative P&L = running sum of realized + unrealized P&L
- Commission = Lots * Sides * Rate
- NG contract multiplier: 10,000 MMBtu per lot

**Data Source**: CME settlements — `https://www.cmegroup.com/markets/energy/natural-gas/natural-gas.settlements.html`

**Quick Update Prompt**: "Download today's NYMEX NG settlement prices from CME for contracts NGH26 through NGZ28. Give me a table with Contract in column 1 and today's settlement price in column 2."

---

### Product 10: Portfolio Risk Model

**Purpose**: Portfolio-level risk analytics across NG positions — VaR, drawdown, correlation, and margin analysis.

**Main Tab (Risk Dashboard)**: Portfolio summary with mark-to-market values, VaR at 95%/99%, maximum drawdown, correlation matrix, and margin-to-equity ratio.

**Historical Data (Historical Prices) columns**: Date | HH Spot | C1 | C2 | C3 | C4

**Key Formulas**:
- Log Returns = LN(Price_today / Price_yesterday)
- VaR(95%) = Portfolio Value * z(0.95) * Portfolio Vol
- Drawdown% = IF(peak=0, 0, (peak - current) / peak)
- Margin-to-Equity = Margin Required / Account Equity

**Key EIA Series** (same as P8): `RNGWHHD`, `RNGC1`, `RNGC2`, `RNGC3`, `RNGC4`

**Quick Update Prompt**: "Download today's HH spot and NYMEX C1-C4 settlement prices. Give me one row: Date, HH Spot, C1, C2, C3, C4."

---

## 4. Cross-Product Relationships

The 10 products form an integrated NG fundamental analysis framework:

```
SUPPLY SIDE                    DEMAND SIDE
    P6 Rig Count ──6mo──▶ P4 Production    P2 Weather ──────▶ Res/Comm Demand
                              │                  │
                              ▼                  ▼
                         P5 Balance ◀──── P3 Gas Burn ◀── P7 Nuclear
                              │
                              ▼
                         P1 Storage
                              │
                              ▼
                         P8 Price
                              │
                         ┌────┴────┐
                         ▼         ▼
                    P9 Trading   P10 Risk
```

### Key Linkages
- **Rig Count → Production** (P6→P4): Gas rigs lead marketed production by ~6 months
- **Weather → Demand** (P2→P3/P5): Heating degree days drive residential/commercial consumption
- **Nuclear → Gas Burn** (P7→P3): Nuclear outages force gas-fired generation to fill the gap
- **Production + Demand → Balance** (P4+P3→P5): Supply/demand balance drives storage
- **Balance → Storage** (P5→P1): Net injection/withdrawal changes working gas levels
- **Storage → Price** (P1→P8): Storage surplus/deficit vs 5-year avg is the primary price signal
- **Price → Trading/Risk** (P8→P9/P10): HH spot and curve drive position valuation

---

## 5. NG Market Primer

### Seasonality
- **Injection Season** (Apr-Oct): Warmer months, lower heating demand, storage builds
- **Withdrawal Season** (Nov-Mar): Cold months, high heating demand, storage draws
- The April 1 storage level and November 1 carryout are key seasonal benchmarks

### Storage Dynamics
- US working gas storage capacity: ~4,700 BCF (L48)
- Normal end-of-season levels: ~1,800-2,000 BCF (end of withdrawal), ~3,500-3,800 BCF (end of injection)
- The storage surplus/deficit vs the 5-year average is the single most important fundamental indicator for NG pricing

### Price Concepts
- **Henry Hub (HH)**: Primary US NG pricing benchmark, located in Erath, Louisiana
- **Contango**: Forward prices higher than spot (normal in injection season — carrying cost of storage)
- **Backwardation**: Spot prices higher than forward (typical in tight supply or high demand)
- **Basis**: Difference between two price points (e.g., HH spot minus NYMEX C1)

### Production Metrics
- **Marketed Production**: Total wellhead production minus gas used in field operations
- **Dry Production**: Marketed minus natural gas plant liquids (NGPLs) — what enters the pipeline system
- **DPR (Drilling Productivity Report)**: EIA's basin-level monthly production estimates (faster than state data)

### Degree Days
- **HDD (Heating Degree Day)**: MAX(0, 65 - Tavg) — measures heating demand
- **CDD (Cooling Degree Day)**: MAX(0, Tavg - 65) — measures cooling demand (gas-fired AC generation)
- Population-weighted degree days account for where people actually live and consume energy

### S/D Balance Identity
Total Supply (Dry Prod + Imports) = Total Demand (Consumption + Exports) + Net Storage Change

---

## 6. Complete Data Source Reference

### EIA API v2 Endpoints

| Endpoint | Data | Frequency | Key Series |
|----------|------|-----------|------------|
| `/v2/natural-gas/stor/wkly/data/` | Storage | Weekly | NW2_EPG0_SWO_R48_BCF (+ regionals) |
| `/v2/natural-gas/prod/sum/data/` | Production | Monthly | N9050US2, N9070US2, N9160US2, state codes |
| `/v2/natural-gas/cons/sum/data/` | Consumption | Monthly | N3010US2, N3020US2, N3025US2, N3035US2, N3045US2, N9170US2 |
| `/v2/natural-gas/move/impc/data/` | Imports | Monthly | N9102CN2, N9102MX2, N9103US2 |
| `/v2/natural-gas/move/expc/data/` | Exports | Monthly | N9132CN2, N9132MX2, N9133US2 |
| `/v2/natural-gas/stor/sum/data/` | Storage (monthly) | Monthly | N5020US2, N5070US2 |
| `/v2/natural-gas/pri/fut/data/` | Prices | Daily | RNGWHHD, RNGC1, RNGC2, RNGC3, RNGC4 |
| `/v2/natural-gas/pri/sum/data/` | Citygate | Monthly | N3050US3 |
| `/v2/steo/data/` | STEO Forecast | Monthly | NGMPPUS, NGHHUUS |
| `/v2/electricity/electric-power-operational-data/data/` | Generation | Monthly | fueltypeid=NUC or NG |

**Important API Notes**:
- Production endpoint uses `facets[series][]` with short IDs (e.g., `N9050US2`)
- STEO endpoint uses `facets[seriesId][]` — different facet parameter name
- Weekly storage uses full dotted IDs (e.g., `NG.NW2_EPG0_SWO_R48_BCF.W`)
- Daily prices use dotted IDs (e.g., `NG.RNGWHHD.D`)

### Non-EIA Sources

| Source | Data | URL | Frequency |
|--------|------|-----|-----------|
| NOAA CPC | Pop-weighted HDD/CDD | `ftp.cpc.ncep.noaa.gov/htdocs/degree_days/weighted/daily_data/` | Daily |
| Baker Hughes | Gas rig counts | `bakerhughes.com/rig-count` | Weekly (Fri) |
| CME Group | NYMEX NG settlements | `cmegroup.com/markets/energy/natural-gas/natural-gas.settlements.html` | Daily |
| EIA DPR | Basin production | `eia.gov/petroleum/drilling/xls/dpr-data.xlsx` | Monthly |
| NRC | Reactor status | `nrc.gov/reading-rm/doc-collections/event-status/reactor-status/` | Daily |

---

## 7. Update Schedule

| Cadence | Products | Data Source | Release Time |
|---------|----------|-------------|-------------|
| **Weekly (Thu)** | P1 Storage | EIA Weekly Storage Report | 10:30 AM ET |
| **Weekly (Fri)** | P6 Rig Count | Baker Hughes | 1:00 PM ET |
| **Weekly** | P2 Weather | NOAA CPC degree days | Daily (aggregate weekly) |
| **Daily** | P8 Price, P9 Trading, P10 Risk | EIA prices / CME settlements | After market close |
| **Monthly** | P3 Gas Burn, P4 Production, P5 Balance, P7 Nuclear | EIA monthly data | ~2 month lag |

---

## 8. Troubleshooting

### Common Issues
- **EIA API returns empty**: Check that the series ID format matches the endpoint (short vs dotted)
- **Storage data missing for current week**: Thursday reports cover through the prior Friday — data may not be available until after 10:30 AM ET
- **Production data seems stale**: National production lags ~2 months; state data lags 3-4 months
- **Baker Hughes rig count missing**: BH skips some holiday weeks; use the most recent available Friday

### Data Quality Checks
- Storage levels should be between 1,000-4,200 BCF (L48)
- US dry production should be 95-110 BCF/d range (as of 2025-2026)
- HDD should be 0 in summer months; CDD should be 0 in winter months
- HH spot price typically ranges $1.50-$10.00/MMBtu (extremes possible in weather events)
