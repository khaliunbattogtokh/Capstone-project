# Did ChatGPT Reshape Entry-Level Job Postings?

**ECON696 Capstone — Khaliun Battogtokh, University of San Francisco, May 2026**

This repository contains the code, data, and figures for a difference-in-differences analysis examining whether ChatGPT's launch in November 2022 reduced entry-level job postings in AI-exposed occupations.

## Repository Structure

```
├── Code/
│   ├── ai_comments.ipynb          # Primary notebook — full end-to-end pipeline
│   ├── postings_jolts.ipynb       # Robustness: JOLTS validation, demand shocks
│   └── archive/                   # Old notebooks (do not use)
├── fig/                           # All output figures (PNG)
├── papers/                        # Reference papers
├── ai_exposure.csv                # AI exposure scores by SOC code (Eloundou et al. 2023)
├── occupation_automation_augmentation_data.csv  # Brynjolfsson et al. (2025) automation/augmentation index
└── README.md
```

## Data Requirements

### Included in this repo
- **`ai_exposure.csv`** — GPT-4 task exposure scores by O*NET SOC code. Primary measure: `gpt4_beta`.
- **`occupation_automation_augmentation_data.csv`** — Brynjolfsson, Chandar & Chen (2025) automation/augmentation index. SOC column: `O*NET-SOC Code` (trimmed to 7 chars).

### Requires institutional access
- **Lightcast job postings** (via Snowflake) — `emsi.us.postings`, 2014–2024. Set environment variables before running:
  ```bash
  export SNOWFLAKE_USER=your_username
  export SNOWFLAKE_PASSWORD=your_password
  ```
  Snowflake account: `avb99459.us-east-1`, warehouse: `OPPORTUNITYATWORK_WH`

- **BLS JOLTS** — Total nonfarm job openings (NSA). Download from [BLS](https://www.bls.gov/jlt/).
- **BLS OEWS** — National occupational employment & wages, 2019–2024. Download from [BLS](https://www.bls.gov/oes/tables.htm).

## Running the Analysis

Open and run `Code/ai_comments.ipynb` in order. The notebook is self-contained and covers:

1. Load AI exposure data + query Snowflake for postings by years-of-experience bucket
2. Validate against BLS JOLTS
3. Descriptive charts (software developers by YOE, finance analysts by YOE, overall postings, automating vs. augmenting occupations)
4. Difference-in-differences regression pipeline (two-way FE, WLS, clustered SEs)
5. Event study (monthly and annual)

### Required packages
```
pandas numpy matplotlib seaborn statsmodels snowflake-connector-python openpyxl
```

## Key Results

Two-way FE DiD, WLS weighted by 2021 posting shares, SEs clustered by occupation:

| Specification | Treatment | Coef. | SE | p-value |
|---|---|---|---|---|
| Main DiD | `gpt4_beta` | −1.729 | 0.898 | 0.054 |
| + rate sensitivity control | `gpt4_beta` | −1.926 | 0.824 | 0.019 |
| Main DiD | `observed_exposure` | −2.243 | 0.921 | 0.015 |
| + rate sensitivity control | `observed_exposure` | −2.426 | 0.889 | 0.006 |

## References

- Eloundou, T., Manning, S., Mishkin, P., & Rock, D. (2023). GPTs are GPTs: An early look at the labor market impact potential of large language models.
- Brynjolfsson, E., Chandar, P., & Chen, W. (2025). Are the Canaries in the Coal Mine? Generative AI and Entry-Level White-Collar Work.
- Anthropic (2025). Anthropic Economic Index.
