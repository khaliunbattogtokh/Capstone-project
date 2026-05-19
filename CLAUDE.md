# ECON696 – Capstone: Did ChatGPT Reshape Entry-Level Job Postings?

## Research Question

Did AI (specifically GPT-4 / ChatGPT) reduce entry-level hiring in highly AI-exposed occupations after ChatGPT launched in November 2022?

## Project Structure

```
ECON696/
├── Code/
│   ├── ai_comments.ipynb              # PRIMARY notebook — full end-to-end pipeline
│   ├── postings_jolts.ipynb           # Robustness controls: JOLTS, demand shocks
│   ├── Anthropic_exposure_index.ipynb # Anthropic observed_exposure measure construction (untracked)
│   └── archive/
│       ├── AI_exposure_occs.ipynb     # OLD — 3 known bugs, stale results, do not use
│       ├── AI_exposure_by_job_sep12.ipynb
│       └── AI exposure by job, occ.ipynb
├── fig/                               # All output figures (PNGs) — tracked on GitHub
├── slide_figures/                     # Figures used in slides (local copy, also in fig/)
├── Output/
│   └── ai_exposed_occs.xlsx
├── updated code/                      # Local only — not tracked on GitHub
│   ├── postings_jolts.ipynb           # (duplicate kept locally)
│   ├── section4_regression.ipynb
│   └── data/
│       └── national_M20XX_dl.xlsx     # BLS OEWS national employment & wages
├── arman research/                    # Local only — not tracked
│   └── postings_jolts.ipynb
├── papers/                            # Reference papers
├── writeup/
│   ├── paper_draft_final_v2.qmd       # Final paper (Quarto)
│   ├── paper_draft_final_v2.html      # Rendered paper — submitted to Canvas
│   └── slides_econ.qmd / .html        # Presentation slides (presented May 2026)
├── ai_exposure.csv                    # AI exposure scores by O*NET SOC code
└── occupation_automation_augmentation_data.csv  # Brynjolfsson et al. automation/augmentation index
```

## Primary Notebook: `Code/ai_comments.ipynb`

Self-contained end-to-end pipeline (replaces the old buggy `AI_exposure_occs.ipynb`):

1. Loads AI exposure data + queries Snowflake for postings by YOE bucket
2. Validates against BLS JOLTS
3. Builds descriptive charts (SWD by YOE, Finance Analysts by YOE, overall postings, canaries)
4. Runs full DiD regression pipeline (Steps 1–7b) + robustness checks
5. Runs event study (monthly + annual)

## Data Sources

- **`ai_exposure.csv`** — Occupation-level AI exposure scores (GPT-4 alpha/beta/gamma). Key column: `occ_code` (7-char SOC). Primary measure: `gpt4_beta`.
- **`occupation_automation_augmentation_data.csv`** — Brynjolfsson, Chandar & Chen (2025) automation/augmentation index. SOC column: `O*NET-SOC Code` (trim to 7 chars with `str[:7]`).
- **Snowflake (Lightcast job postings)** — `emsi.us.postings`, 2014–2024. Account: `avb99459.us-east-1`, warehouse: `OPPORTUNITYATWORK_WH`. Requires `SNOWFLAKE_USER` and `SNOWFLAKE_PASSWORD` env vars.
- **BLS JOLTS** — Total nonfarm job openings (NSA), stored in `updated code/SeriesReport-*.xlsx`. Read with `skiprows=13, nrows=11`.
- **BLS OEWS** — National occupational employment & wages, annual 2019–2024. Stored in `updated code/data/national_M20XX_dl.xlsx`. Filter to `o_group == 'detailed'`.

## Key Variables

- **`entry_level_pct`** — Share of postings that are entry-level: (`min_years_experience ≤ 2` OR `job_seniority_name = 'Junior' AND min_years_experience IS NULL`) AND `min_edulevels_name` ≤ Bachelor's Degree.
- **`gpt4_beta`** — GPT-4 AI exposure score (0–1), time-invariant. Primary treatment measure.
- **`observed_exposure`** — Anthropic Economic Index: actual Claude usage patterns by occupation. Robustness treatment measure. Corr with `gpt4_beta`: r = 0.638, Spearman ρ = 0.699.
- **`entry_sa`** — Seasonally adjusted entry-level share (month dummies removed via WLS).
- **`w_2021`** — Occupation weight = share of total 2021 postings.
- **`rate_sensitivity`** — Pre-period OLS coefficient: occupation's monthly posting % change regressed on ΔFFR (2020–2022). Computed for 729 occupations.
- **Event reference month** — October 2022 (last clean pre-treatment month; ChatGPT launched Nov 30, 2022, following Brynjolfsson et al. 2025 convention).

## Final DiD Results (ai_comments.ipynb, May 14 2026)

**Entry-level definition:** YOE ≤ 2 OR (Junior + NULL YOE), edu ≤ Bachelor's  
**Y:** `entry_sa` (seasonally adjusted %)  
**Model:** Two-way FE (occupation + time), WLS (2021 weights), SEs clustered by occupation

| Spec | Treatment | β | SE | p | 95% CI | N obs | N occ |
|---|---|---|---|---|---|---|---|
| (1) Main DiD | `gpt4_beta` | −1.729 | 0.898 | 0.054 | [−3.49, 0.03] | 45,436 | 733 |
| (2) + rate sensitivity | `gpt4_beta` | −1.926 | 0.824 | 0.019 | [−3.54, −0.31] | 45,186 | 729 |
| (3) Main DiD | `observed_exposure` | −2.243 | 0.921 | 0.015 | [−4.05, −0.44] | 44,403 | 716 |
| (4) + rate sensitivity | `observed_exposure` | −2.426 | 0.889 | 0.006 | [−4.17, −0.68] | ~44,100 | ~712 |

**Key findings:**
- Entry-level postings fell 2.4× harder than senior in software developer postings (56% vs 81% of Oct 2022 level)
- Overall entry-level postings have not recovered; only 11+ YOE has grown
- Decline concentrated in automating occupations; augmenting occupations show different pattern
- Rate control increases β magnitude — AI-exposed occupations are less rate-sensitive (r = 0.047)
- Demand shock control leaves β unchanged (−1.729 → −1.732) — AI effect is distinct

## Submissions (May 2026)

- **Final Submission** (Canvas, 10 pts): `writeup/paper_draft_final_v2.html`
- **Reproducibility Materials** (Canvas, 5 pts): `https://github.com/khaliunbattogtokh/Capstone-project`

## Environment Setup

```bash
export SNOWFLAKE_USER=your_username
export SNOWFLAKE_PASSWORD=your_password
```

Required packages: `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, `snowflake-connector-python`, `openpyxl`
