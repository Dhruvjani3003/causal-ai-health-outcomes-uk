# Causal AI and Health Outcomes — England

Causal discovery and causal inference applied to open NHS/ONS/IMD data, investigating what drives infant mortality disparities across deprivation levels and ethnic groups in England.

## Background

Infant mortality in England varies by both deprivation and ethnic group. ONS data shows babies in more deprived areas face higher mortality risk, and Black and Asian babies have consistently higher infant mortality rates than White babies nationally. This project investigates whether these factors are just correlated with mortality, or plausibly *cause* it, and by how much — using causal discovery and causal inference rather than correlation alone.

## Data sources

- **NHS Maternity Statistics 2024-25** (digital.nhs.uk) — birth outcomes and delivery data by region
- **ONS: Births and infant mortality by ethnicity in England and Wales, 2007-2019** (ons.gov.uk) — Table 13 specifically, which combines ethnicity, IMD decile, and infant mortality rate in one pre-joined table
- **Index of Multiple Deprivation 2025** (IoD2025) — LSOA-level deprivation scores, including the Health Deprivation and Disability sub-domain
- **ONS Health Index underlying data, England** — explored as supplementary context

Raw files required inspection before use — headers are offset several rows into each sheet, some values are ONS-suppressed for reliability ("u" flags on small-number estimates), and the IMD decile column contained a summary "Total" row that had to be removed. Cleaning steps (header alignment, missing-value handling, column standardisation) are in `notebooks/CHILD.ipynb`.

## Method

1. **Data cleaning** — loaded ONS Table 13, reshaped from wide to long format (one row per year/IMD decile/ethnic group), removed summary and missing rows
2. **Exploratory analysis** — mortality by ethnicity and deprivation, scatter and box plots, correlation matrix, Spearman rank correlation
3. **Causal discovery** — PC algorithm (`pgmpy`) to recover a causal graph between deprivation, ethnicity, and mortality; one wrong-direction edge was found and corrected using background knowledge (ethnicity/deprivation cannot be caused by an outcome measured after birth)
4. **Causal inference** — `DoWhy` backdoor adjustment (linear regression estimator) to quantify each causal effect, with random-common-cause refutation tests to check robustness

## Key findings

1. **Deprivation causally increases infant mortality**, controlling for ethnicity: each 1-decile move away from deprivation is associated with a -0.136 point change in mortality rate (robust to refutation test, p=0.92).
2. **Ethnicity has a direct effect on infant mortality independent of deprivation.** Compared to White babies, controlling for deprivation: Black babies show +3.73 points higher mortality (robust to refutation, p=1.0), Asian +1.59, Any Other ethnic group +1.13, Mixed +0.60.
3. **These two effects are additive, not competing.** Deprivation explains part of the disparity, but a substantial ethnicity-specific gap remains unexplained by deprivation alone — consistent with the national ONS pattern this project set out to investigate.

## Limitations

- Observational data, not a randomized experiment — "causal" here means consistent with a causal story once known confounders are controlled for, not proof beyond doubt
- ~15-20% of rows were excluded due to missing IMD decile values or ONS-suppressed "unreliable" rate estimates, disproportionately affecting smaller ethnic groups
- Data is aggregated at year/decile/ethnicity level, not individual birth records
- Known confounders (maternal age, smoking, antenatal care access, birthweight) are not included in this model
- Data covers 2007-2019, pre-dating COVID-19

## Policy implications

Reducing infant mortality requires addressing both deprivation and ethnicity-specific factors separately — tackling poverty alone would not close the ethnic disparity gap found in this analysis.

## Repo structure

```
causal-ai-health-outcomes-uk/
├── data/
│   ├── raw/          — downloaded NHS/ONS/IMD files
│   └── processed/    — cleaned master dataset
├── notebooks/
│   └── CHILD.ipynb   — full pipeline: cleaning, EDA, causal discovery, causal inference
├── outputs/
│   └── figures/       — saved charts and causal DAG
├── requirements.txt
└── README.md
```
