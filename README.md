# BMI Disparities Across Ethnic Groups in U.S. Adults
### A Mixed Effects Regression Analysis — NHANES 2013–2018

> **Research question:** Do ethnic groups show systematically different BMI levels after controlling for age, sex, income, and time — and what does that tell us about where public health interventions should focus?

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Sj9GjEB3Miz4QEKnJYvy17Lu2460eZsg)

---

## Overview

This project applies a **Linear Mixed Effects Model** to three cycles of NHANES (National Health and Nutrition Examination Survey) data (2013–2014, 2015–2016, 2017–2018) to quantify how much of the variation in adult BMI is explained by ethnic group membership versus individual-level factors such as age, sex, and income.

The analysis follows a rigorous data science workflow: multi-source data merging, four-dimension quality audit, exploratory analysis, formal statistical modeling, and clinically grounded recommendations.

**Key finding:** Ethnic group membership explains only 8.3% of BMI variation (ICC = 0.083). The remaining 91.7% is individual-level — meaning one-size-fits-all group interventions are insufficient, and income-targeted strategies offer more actionable leverage.

---

## Dataset

**Source:** [National Health and Nutrition Examination Survey (NHANES)](https://www.cdc.gov/nchs/nhanes/index.htm) — U.S. Centers for Disease Control and Prevention

| Property | Value |
|---|---|
| Survey cycles | 2013–2014, 2015–2016, 2017–2018 |
| Raw participants | 28,061 |
| After quality cleaning | 16,940 adults (age 18–80) |
| Final modeling sample | 15,767 (after removing refused/unknown income) |
| Files used | DEMO_H/I/J · BMX_H/I/J |
| Join key | `SEQN` (unique participant ID) |

**Variables:**

| Variable | Type | Description |
|---|---|---|
| `bmi` | Continuous (outcome) | Body Mass Index (kg/m²) |
| `age` | Continuous | Age at screening in years |
| `sex` | Binary | Male / Female |
| `ethnicity` | Categorical (6 groups) | Random effect grouping variable |
| `income` | Ordinal (1–12) | Annual household income category |
| `cycle` | Ordinal (0, 1, 2) | Survey cycle as timepoint |

---

## Methodology

### Data Quality Audit

Four-dimension quality check performed before any analysis:

| Dimension | Finding | Action |
|---|---|---|
| **Completeness** | BMI + height missing in same 2,209 rows; waist 12.4% missing | Dropped missing core measures; flagged waist |
| **Validity** | 8,873 minors; 8 weight errors; 3 extreme BMI outliers | Dropped all |
| **Consistency** | BMI recalculated from weight/height — 0 discrepancies | No action needed |
| **Duplicates** | 0 fully duplicate rows; 0 duplicate SEQN within cycle | No action needed |

### Model Specification

$$Y_{ij} = \beta_0 + \beta_1 \text{age}_{ij} + \beta_2 \text{age}^2_{ij} + \beta_3 \text{sex}_{ij} + \beta_4 \text{income}_{ij} + \beta_5 \text{cycle}_{ij} + u_j + \varepsilon_{ij}$$

| Symbol | Definition |
|---|---|
| $Y_{ij}$ | BMI of participant $i$ in ethnic group $j$ |
| $\beta_0$ | Population intercept — average BMI at reference values |
| $\beta_1, \beta_2$ | Linear and quadratic age effects |
| $\beta_3$ | Sex effect (Female vs Male reference) |
| $\beta_4$ | Income effect (per ordinal category) |
| $\beta_5$ | Time trend across survey cycles |
| $u_j \sim \mathcal{N}(0, \sigma^2_u)$ | Ethnic group random intercept |
| $\varepsilon_{ij} \sim \mathcal{N}(0, \sigma^2_\varepsilon)$ | Individual residual |

Fitted using **REML** in R (`lme4` + `lmerTest`). P-values computed via **Satterthwaite's approximation**.

---

## Results

### Fixed Effects — All significant at p < 0.01

| Predictor | Estimate | Interpretation |
|---|---|---|
| β₀ = 30.01 | Intercept | Population average BMI sits at the clinical obesity threshold |
| β₁ = +0.021 | Age (linear) | BMI rises with age |
| β₂ = -0.004 | Age (squared) | Rise decelerates and reverses after ~age 55 |
| β₃ = +0.974 | Sex: Female | Females ~1 BMI unit higher than males after controls |
| β₄ = -0.055 | Income | Each income step higher → 0.055 lower BMI |
| β₅ = +0.396 | Cycle | BMI rising ~0.4 units per 2-year cycle at population level |

### Random Effects — Ethnic Group Deviations

| Ethnic Group | Deviation from Mean |
|---|---|
| Non-Hispanic Asian | **-4.08** |
| Other Hispanic | +0.12 |
| Non-Hispanic White | +0.28 |
| Other/Multiracial | +1.02 |
| Non-Hispanic Black | +1.25 |
| Mexican American | **+1.42** |

### Variance Decomposition

| Component | Value | Meaning |
|---|---|---|
| Between-group variance σ²u | 4.30 | Variation explained by ethnicity |
| Residual variance σ²ε | 47.70 | Individual-level variation |
| **ICC** | **0.083 (8.3%)** | Ethnicity explains 8.3% of BMI variation |

---

## Key Findings

**1. Population BMI is rising regardless of demographics**
The +0.396 cycle effect means average adult BMI increased ~0.8 units between 2013 and 2018 — a public health signal independent of group membership.

**2. Income is the most actionable predictor**
Higher income is consistently associated with lower BMI across all groups. Unlike age or sex, income is a modifiable target for policy intervention — housing, food access, and employment programs could have measurable BMI effects.

**3. Non-Hispanic Asian BMI requires different clinical thresholds**
A 4-unit deviation below the population mean is large. The WHO (World Health Organization)has recommended lower BMI thresholds for Asian populations (overweight ≥ 23 instead of ≥ 25), and this finding supports that approach.

**4. Mexican American and Non-Hispanic Black groups carry unexplained BMI burden**
After controlling for age, sex, income, and time, these groups still show the highest deviations. This points to unmeasured structural factors — neighborhood food environments, occupational physical demands, healthcare access — that merit further investigation.

**5. Individual variation dominates group effects (ICC = 8.3%)**
The most important methodological finding: 91.7% of BMI variation is within groups, not between them. This means targeting interventions at ethnic groups as monolithic categories will miss most of the population at risk.

---

## Clinical Recommendations

Based on the model findings:

**Short term**
- Prioritize income-based screening — patients in lower income brackets show systematically higher BMI regardless of ethnicity
- Apply lower BMI thresholds for Non-Hispanic Asian patients in clinical assessments

**Medium term**
- Design interventions at the individual level, not the group level — within-group heterogeneity is too large for group-based programs to be effective
- Monitor the rising cycle trend (+0.4 per 2 years) — if it continues at this rate, average adult BMI will cross 31 by 2025

**Long term**
- Investigate the structural factors behind the residual Mexican American and Non-Hispanic Black deviations — income alone does not fully explain them
- Future studies should incorporate dietary data, physical activity measures, and neighborhood-level variables to reduce the unexplained 91.7% individual variance

---

## Limitations

- **Cross-sectional design:** NHANES surveys different people each cycle — individual trajectories cannot be tracked. Results describe population-level associations, not causal pathways.
- **Survey weights not applied:** NHANES oversamples minority groups. Applying `WTMEC2YR` weights would adjust estimates for national representativeness — a recommended extension.
- **BMI as outcome:** BMI does not distinguish muscle from fat and performs differently across ethnic groups. Body fat percentage or waist-to-height ratio may be more appropriate in future work.
- **Income is self-reported and categorical:** Measurement error in income is likely and the ordinal encoding assumes equal spacing between categories.
- **6 ethnic groups as random effects:** With only 6 groups, estimation of σ²u is imprecise. Results should be interpreted as exploratory rather than confirmatory.

---

## Future Extensions

This project establishes a foundation that can be extended in several directions:

**1. Apply NHANES survey weights**
Adding `WTMEC2YR` as an analytical weight would make findings nationally representative — a one-cell change in R with significant interpretive value.

**2. Add dietary and activity covariates**
NHANES also contains dietary recall (DR1, DR2) and physical activity modules. Adding these as fixed effects would reduce residual variance and sharpen the income and ethnicity estimates.

**3. Extend to additional NHANES cycles**
Cycles from 2005 onwards are publicly available. Adding more timepoints would allow detection of longer-term trends and reduce uncertainty in the cycle coefficient.

**4. Waist circumference as secondary outcome**
Re-run the same model with `waist_cm` as outcome — waist is a stronger predictor of cardiovascular risk than BMI and would add a second clinically relevant dimension to the analysis.

**5. Age-stratified models**
The current model spans ages 18–80. Fitting separate models for 18–40, 40–60, and 60–80 would test whether income and ethnicity effects are consistent across the life course or concentrated in specific age windows.

**6. Interaction terms**
Testing `income × ethnicity` and `income × cycle` interactions would reveal whether the income-BMI relationship has strengthened or weakened over time, and whether it operates differently across groups.

---

## Tech Stack

- **Python** — pandas, matplotlib, seaborn, numpy, scipy
- **R** — lme4, lmerTest
- **Environment** — Google Colab
- **Data** — NHANES public use files (CDC, no IRB required)

---

## Repository Structure

```
health-longitudinal-analysis/
├── health_project.ipynb     ← full analysis notebook
├── DEMO_H_2013_2014.csv
├── DEMO_I_2015_2016.csv
├── DEMO_J_2017_2018.csv
├── BMX_H_2013_2014.csv
├── BMX_I_2015_2016.csv
├── BMX_J_2017_2018.csv
└── README.md
```

---

## How to Run

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Sj9GjEB3Miz4QEKnJYvy17Lu2460eZsg)

Click the badge above — all data loads automatically from GitHub. Run all cells sequentially by clicking "Run all".

### Run locally

```bash
git clone https://github.com/sebasDobleU/health-longitudinal-analysis
cd health-longitudinal-analysis
pip install pandas matplotlib seaborn scipy numpy
jupyter notebook health_project.ipynb
```

---

## Author

**Sebastian White Restrepo**
Physical Engineering Student · Universidad EAFIT · Medellín, Colombia
Open to freelance data science and ML projects

[GitHub](https://github.com/sebasDobleU) · sebasw7109@gmail.com
