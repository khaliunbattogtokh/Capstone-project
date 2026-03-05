# ECON696 – Independent Study

## Research Question
Did AI (specifically GPT-4) reduce entry-level hiring in highly AI-exposed occupations after ChatGPT launched in October 2022?

## Project Structure

```
ECON696/
├── Code/
│   ├── AI_exposure_occs.ipynb         # Main analysis: entry-level share × AI exposure
│   ├── AI_exposure_by_job_sep12.ipynb # AI exposure measure construction from job postings
│   └── AI exposure by job, occ.ipynb  # Earlier exploratory version
├── Output/
│   └── ai_exposed_occs.xlsx           # Occupation-level AI exposure output
├── papers/                            # Reference papers
├── writeup/                           # Paper draft
├── ai_exposure.csv                    # AI exposure scores by O*NET SOC code
└── occupation_automation_augmentation_data.csv
```

## Data Sources

- **`ai_exposure.csv`** — Occupation-level AI exposure scores (GPT-4 alpha/beta/gamma, human raters, automation) keyed by O*NET SOC code. `gpt4_beta` is the primary exposure measure.
- **Snowflake (EMSI job postings)** — `emsi.us.postings`, 2014–2024. Requires `SNOWFLAKE_USER` and `SNOWFLAKE_PASSWORD` env vars. Account: `avb99459.us-east-1`, warehouse: `OPPORTUNITYATWORK_WH`.
- **Google Drive** — Working directory is mounted at `~/Library/CloudStorage/GoogleDrive-{user}@opportunityatwork.org/Shared drives/Insights`.

## Key Variables

- **`entry_level_pct`** — Share of job postings that are entry-level (≤1 year experience OR Junior seniority with no YOE listed, and ≤Bachelor's degree)
- **`gpt4_beta`** — GPT-4 AI exposure score for the occupation (0–1)
- **`entry_sa`** — Seasonally adjusted entry-level share (month dummies removed)
- **`exposure_group`** — Tertile bin of `gpt4_beta`: Low / Medium / High
- **`w_2021`** — Occupation weight = share of total 2021 postings (used for weighted averages)
- **Event date** — October 2022 (ChatGPT launch), indexed to 100

## Analysis Pipeline

1. Build AI exposure measure from EMSI skills data (`AI_exposure_by_job_sep12.ipynb`)
2. Query monthly entry-level posting shares by occupation from Snowflake
3. Merge on occupation code (SOC 2021 5-digit)
4. Seasonally adjust entry-level share (OLS on month dummies)
5. Compute 2021 occupation weights
6. Bin occupations into Low/Medium/High AI exposure tertiles
7. Compute weighted monthly averages per group, index to Oct 2022 = 100
8. Event-study regression: `index ~ C(event_time) + C(event_time):C(exposure_group)` (HC1 SEs)
9. DiD regression: `indexed_share ~ post * gpt4_beta + C(OCC_CODE) + C(ym_str)`, WLS weighted by `w_2021_pct`, SEs clustered by occupation

## Environment Setup

```bash
export SNOWFLAKE_USER=your_username
export SNOWFLAKE_PASSWORD=your_password
```

Required packages: `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, `snowflake-connector-python`, `openpyxl`, `xlsxwriter`, `pyreadstat`
