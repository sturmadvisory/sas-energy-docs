# SAS Energy Analyst — GPT Configuration

Use this reference when creating the Custom GPT in ChatGPT.

---

## Basic Settings

| Field | Value |
|-------|-------|
| **Name** | SAS Energy Analyst |
| **Description** | Professional natural gas market analyst for the SAS Energy Analytics product suite. Updates spreadsheets, explains models, and provides fundamental market analysis across 10 NG data products. |

---

## Instructions

Paste the contents of `GPT_INSTRUCTIONS.md` into the Instructions field.

---

## Conversation Starters

1. "Update my NG Storage Forecast with this week's EIA storage report"
2. "What's driving natural gas prices this week?"
3. "Explain how the production nowcast model works"
4. "How do weather degree days flow through to the balance report?"

---

## Capabilities

| Capability | Setting |
|------------|---------|
| **Web Browsing** | ON — needed to fetch live data from EIA, NOAA, Baker Hughes, CME |
| **Code Interpreter** | ON — needed to process API responses and format paste-ready tables |
| **DALL-E Image Generation** | OFF — not needed for data analysis |

---

## Knowledge Files

Upload these 12 files as knowledge:

| # | File | Source Path |
|---|------|------------|
| 1 | AGENT_REFERENCE.md | `sas-energy-docs/agent/AGENT_REFERENCE.md` |
| 2 | FULL_UPDATE.md (Storage) | `sas-energy-docs/storage-forecast/FULL_UPDATE.md` |
| 3 | FULL_UPDATE.md (Weather) | `sas-energy-docs/degree-day-weather/FULL_UPDATE.md` |
| 4 | FULL_UPDATE.md (Gas Burn) | `sas-energy-docs/gas-burn-power-stack/FULL_UPDATE.md` |
| 5 | FULL_UPDATE.md (Production) | `sas-energy-docs/ng-production/FULL_UPDATE.md` |
| 6 | FULL_UPDATE.md (Balance) | `sas-energy-docs/ng-balance-report/FULL_UPDATE.md` |
| 7 | FULL_UPDATE.md (Rig Count) | `sas-energy-docs/rig-count-forecast/FULL_UPDATE.md` |
| 8 | FULL_UPDATE.md (Nuclear) | `sas-energy-docs/nuclear-capacity/FULL_UPDATE.md` |
| 9 | FULL_UPDATE.md (Price) | `sas-energy-docs/price-scenario/FULL_UPDATE.md` |
| 10 | FULL_UPDATE.md (Trading) | `sas-energy-docs/trading-pnl/FULL_UPDATE.md` |
| 11 | FULL_UPDATE.md (Risk) | `sas-energy-docs/portfolio-risk/FULL_UPDATE.md` |
| 12 | demand_factors.json | `sas-energy-docs/degree-day-weather/demand_factors.json` |

**Note**: Rename each FULL_UPDATE.md before uploading so they are distinguishable:
- `P1_Storage_FULL_UPDATE.md`
- `P2_Weather_FULL_UPDATE.md`
- `P3_GasBurn_FULL_UPDATE.md`
- `P4_Production_FULL_UPDATE.md`
- `P5_Balance_FULL_UPDATE.md`
- `P6_RigCount_FULL_UPDATE.md`
- `P7_Nuclear_FULL_UPDATE.md`
- `P8_Price_FULL_UPDATE.md`
- `P9_Trading_FULL_UPDATE.md`
- `P10_Risk_FULL_UPDATE.md`

---

## Profile Picture

Use a navy/steel blue natural gas flame icon. Suggested style:
- Clean, modern, flat design
- Navy background (#1B2A4A) with steel blue (#4472C4) flame
- "SAS" text optional
- Generate externally (Canva, Figma, or DALL-E separately) and upload

---

## Actions

None required. The GPT fetches data via Web Browsing (no custom API actions needed).

---

## Additional Settings

| Setting | Value |
|---------|-------|
| **Who can use this GPT** | Anyone with a link |
| **Allow conversations to be used for model improvement** | Off (recommended for commercial product) |
