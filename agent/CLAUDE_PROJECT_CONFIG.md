# SAS Energy Analyst — Claude Project Configuration

Use this reference when creating the Project on claude.ai.

---

## Setup Steps

### 1. Create the Project

1. Go to [claude.ai](https://claude.ai) and click **Projects** in the left sidebar
2. Click **Create Project**
3. Name it: **SAS Energy Analyst**
4. Description: *Professional NG market analyst for the SAS Energy Analytics product suite*

### 2. Set Project Instructions

Click **Set custom instructions** and paste the full contents of `GPT_INSTRUCTIONS.md`.

No modifications needed — the instructions are model-agnostic.

### 3. Upload Knowledge Files

Click **Add knowledge** and upload these 12 files:

| # | File | Source Path |
|---|------|------------|
| 1 | AGENT_REFERENCE.md | `sas-energy-docs/agent/AGENT_REFERENCE.md` |
| 2 | P1_Storage_FULL_UPDATE.md | `sas-energy-docs/storage-forecast/FULL_UPDATE.md` |
| 3 | P2_Weather_FULL_UPDATE.md | `sas-energy-docs/degree-day-weather/FULL_UPDATE.md` |
| 4 | P3_GasBurn_FULL_UPDATE.md | `sas-energy-docs/gas-burn-power-stack/FULL_UPDATE.md` |
| 5 | P4_Production_FULL_UPDATE.md | `sas-energy-docs/ng-production/FULL_UPDATE.md` |
| 6 | P5_Balance_FULL_UPDATE.md | `sas-energy-docs/ng-balance-report/FULL_UPDATE.md` |
| 7 | P6_RigCount_FULL_UPDATE.md | `sas-energy-docs/rig-count-forecast/FULL_UPDATE.md` |
| 8 | P7_Nuclear_FULL_UPDATE.md | `sas-energy-docs/nuclear-capacity/FULL_UPDATE.md` |
| 9 | P8_Price_FULL_UPDATE.md | `sas-energy-docs/price-scenario/FULL_UPDATE.md` |
| 10 | P9_Trading_FULL_UPDATE.md | `sas-energy-docs/trading-pnl/FULL_UPDATE.md` |
| 11 | P10_Risk_FULL_UPDATE.md | `sas-energy-docs/portfolio-risk/FULL_UPDATE.md` |
| 12 | demand_factors.json | `sas-energy-docs/degree-day-weather/demand_factors.json` |

**Important**: Rename each FULL_UPDATE.md before uploading so Claude can distinguish them. The names above include the product prefix.

### 4. Quick Copy Script

Run this to create a ready-to-upload folder with renamed files:

```bash
DOCS=~/PythonProjects/sas-energy-docs
OUT=~/Desktop/claude_project_upload
mkdir -p "$OUT"
cp "$DOCS/agent/AGENT_REFERENCE.md" "$OUT/"
cp "$DOCS/storage-forecast/FULL_UPDATE.md" "$OUT/P1_Storage_FULL_UPDATE.md"
cp "$DOCS/degree-day-weather/FULL_UPDATE.md" "$OUT/P2_Weather_FULL_UPDATE.md"
cp "$DOCS/gas-burn-power-stack/FULL_UPDATE.md" "$OUT/P3_GasBurn_FULL_UPDATE.md"
cp "$DOCS/ng-production/FULL_UPDATE.md" "$OUT/P4_Production_FULL_UPDATE.md"
cp "$DOCS/ng-balance-report/FULL_UPDATE.md" "$OUT/P5_Balance_FULL_UPDATE.md"
cp "$DOCS/rig-count-forecast/FULL_UPDATE.md" "$OUT/P6_RigCount_FULL_UPDATE.md"
cp "$DOCS/nuclear-capacity/FULL_UPDATE.md" "$OUT/P7_Nuclear_FULL_UPDATE.md"
cp "$DOCS/price-scenario/FULL_UPDATE.md" "$OUT/P8_Price_FULL_UPDATE.md"
cp "$DOCS/trading-pnl/FULL_UPDATE.md" "$OUT/P9_Trading_FULL_UPDATE.md"
cp "$DOCS/portfolio-risk/FULL_UPDATE.md" "$OUT/P10_Risk_FULL_UPDATE.md"
cp "$DOCS/degree-day-weather/demand_factors.json" "$OUT/"
echo "12 files ready in $OUT"
```

---

## Claude vs ChatGPT Differences

| Feature | Claude Project | ChatGPT Custom GPT |
|---------|---------------|-------------------|
| **Knowledge files** | Yes (uploaded to project) | Yes (uploaded to GPT) |
| **Custom instructions** | Yes (project instructions) | Yes (GPT instructions) |
| **Web search** | Yes (built-in) | Yes (web browsing) |
| **URL fetching** | Via web search results | Direct URL fetch |
| **Code execution** | Artifacts (code blocks) | Code Interpreter (sandbox) |
| **Shareable link** | No (personal project) | Yes (public URL) |
| **File analysis** | Attach per-conversation | Attach per-conversation |
| **Model quality** | Claude Sonnet/Opus | GPT-4o |

### Key Notes for Claude Usage

- **Web search**: Claude can search the web but cannot directly fetch arbitrary URLs like ChatGPT's browsing. For EIA API calls, you may need to paste the API response or attach the spreadsheet and ask Claude to generate the API URL for you to run.
- **Artifacts**: Claude outputs formatted tables and code in Artifacts, which are easy to copy. Ask for "paste-ready table" and Claude will format it cleanly.
- **Attach your spreadsheet**: Start each update conversation by attaching the `.xlsx` file. Claude will read it and determine what data is stale.
- **Strengths**: Claude excels at complex reasoning across the knowledge files — cross-product analysis, methodology explanations, and market commentary are notably stronger.

---

## Starter Prompts

Use these to test the project after setup:

1. "Update my NG Storage Forecast with this week's EIA storage report"
2. "What's driving natural gas prices this week?"
3. "Explain how the production nowcast model works"
4. "How do weather degree days flow through to the balance report?"
5. "Walk me through the cross-product supply chain from rig counts to price"
