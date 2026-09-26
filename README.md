# Respiratory Longitudinal Analysis

**Longitudinal analysis of repeated clinical measurements in R, combining multiple-testing correction with Generalized Estimating Equations (GEE).**

This project investigates whether an active treatment is associated with a better respiratory status than placebo across four follow-up visits.

The analysis addresses two statistical problems that require different solutions:

1. **Multiple comparisons:** four visit-specific treatment comparisons increase the risk of false-positive findings.
2. **Repeated measurements:** four observations from the same patient are correlated and cannot be treated as independent.

The project therefore combines **multiple-testing correction** for the visit-specific analyses with a **GEE model** for the longitudinal analysis.

**Tools:** R · Quarto · geepack · dplyr · ggplot2  
**Methods:** Multiple Testing · Bonferroni · Holm · Benjamini-Hochberg · Benjamini-Yekutieli · GEE · Sensitivity Analysis

---

## Project Overview

### Research Question

**Is active treatment associated with a better respiratory status than placebo across four follow-up visits?**

The outcome is binary:

- `1` = good respiratory status
- `0` = poor respiratory status

The dataset contains repeated measurements from the same individuals.

This means that a valid analysis must consider both:

- differences between treatment groups,
- and dependence between observations from the same patient.

---

## Key Results

### Study Structure

- **111 patients**
- **4 measurements per patient**
- **444 observations**
- Active treatment vs. placebo
- Binary respiratory outcome

### Visit-Specific Differences

The active-treatment group showed a higher observed proportion of good respiratory status at all four visits.

| Visit | Difference: Active Treatment − Placebo |
|---|---:|
| Visit 1 | +19.4 percentage points |
| Visit 2 | +31.8 percentage points |
| Visit 3 | +26.6 percentage points |
| Visit 4 | +17.3 percentage points |

The largest observed difference occurred at **Visit 2**.

---

## Why This Analysis Is Not Just Four Separate Tests

There are two different statistical problems in these data.

```text
Problem 1
Four treatment comparisons
        ↓
Higher risk of false-positive findings
        ↓
Multiple-testing correction


Problem 2
Four observations from each patient
        ↓
Measurements within a patient are correlated
        ↓
Longitudinal GEE model
```

These problems are related, but they are **not the same**.

Multiple-testing correction controls error caused by testing several hypotheses.

It does **not** correct a model that incorrectly treats repeated measurements from the same patient as independent.

For that reason, the visit-specific tests are used as an exploratory analysis, while the **GEE model provides the main longitudinal analysis**.

---

# Analysis Workflow

```text
Load respiratory dataset
        ↓
Understand patient and visit structure
        ↓
Create unique patient identifier
        ↓
Validate repeated measurements
        ↓
Describe respiratory outcomes by treatment and visit
        ↓
Perform four visit-specific treatment comparisons
        ↓
Correct for multiple testing
        ↓
Build longitudinal GEE model
        ↓
Assess treatment × visit interaction
        ↓
Select parsimonious model
        ↓
Adjust for age and sex
        ↓
Check age-model specification
        ↓
Sensitivity analysis
        ↓
Final model and interpretation
```

---

## 1. Data and Study Structure

The project uses the publicly available `respiratory` dataset from the R package `geepack`.

The dataset originates from a randomized clinical study involving patients from two clinical centres.

Participants were assigned to:

- **active treatment**
- **placebo**

Respiratory status was recorded at four follow-up visits.

---

## 2. Identifying the Correct Unit of Analysis

An important issue appears immediately when inspecting the data.

The variable `id` is **not globally unique** because identical ID values can occur in different clinical centres.

I therefore construct a unique patient identifier by combining:

```text
center + id
```

This produces **111 unique patients**.

Each patient has exactly four observations:

```text
111 patients × 4 visits = 444 observations
```

The 444 rows must therefore **not** be interpreted as 444 independent patients.

Measurements from the same person may be correlated.

Ignoring this dependence can produce:

- underestimated standard errors,
- confidence intervals that are too narrow,
- p-values that are too small,
- and overly optimistic conclusions.

---

## 3. Data Validation

Before statistical modelling, the dataset is checked to ensure that:

- patients can be uniquely identified,
- every patient has four recorded visits,
- visit structure is consistent,
- treatment and outcome variables have valid values,
- the binary outcome is correctly coded.

All 111 patients have observations at all four scheduled visits.

The analysis therefore contains:

**111 patients and 444 longitudinal observations.**

Data validation confirms the technical structure of the dataset.

It does not automatically prove the assumptions of the later statistical models.

---

## 4. Descriptive Analysis

The first step is to examine the observed proportion of patients with a good respiratory status at each visit.

The active-treatment group shows a higher proportion of favourable outcomes at every visit.

The observed differences are:

- **Visit 1:** +19.4 percentage points
- **Visit 2:** +31.8 percentage points
- **Visit 3:** +26.6 percentage points
- **Visit 4:** +17.3 percentage points

This provides an initial indication of a treatment-group difference.

However, descriptive differences alone do not establish statistical evidence.

---

# 5. Visit-Specific Statistical Tests

For each visit, the active-treatment group is compared with the placebo group separately.

The null hypothesis at each visit is:

> The probability of a good respiratory status is the same in both treatment groups.

The resulting unadjusted p-values are:

| Visit | Raw p-value |
|---|---:|
| Visit 1 | 0.0381 |
| Visit 2 | 0.000787 |
| Visit 3 | 0.00445 |
| Visit 4 | 0.0690 |

If each p-value were considered separately at the 5% level, Visits **1, 2 and 3** would appear statistically significant.

But four hypotheses were tested simultaneously.

This creates a **multiple-testing problem**.

---

# 6. Multiple-Testing Correction

To assess how stable the visit-specific conclusions are, the same four raw p-values are adjusted using four methods.

### Bonferroni

Controls the **Familywise Error Rate (FWER)**.

It provides strict protection against making at least one false-positive conclusion within the complete family of tests.

### Holm

Also controls the FWER, but is generally less conservative than the simple Bonferroni correction.

### Benjamini-Hochberg

Controls the **False Discovery Rate (FDR)**.

Instead of controlling the probability of any false-positive result, it controls the expected proportion of false discoveries among rejected hypotheses.

### Benjamini-Yekutieli

Also controls the FDR and remains valid under more general forms of dependence between test statistics, at the cost of being more conservative.

---

## Corrected Results

| Visit | Raw p | Bonferroni | Holm | Benjamini-Hochberg | Benjamini-Yekutieli |
|---|---:|---:|---:|---:|---:|
| Visit 1 | 0.0381 | 0.1524 | 0.0762 | 0.0508 | 0.1058 |
| Visit 2 | 0.000787 | 0.00315 | 0.00315 | 0.00315 | 0.00656 |
| Visit 3 | 0.00445 | 0.0178 | 0.01335 | 0.00890 | 0.01854 |
| Visit 4 | 0.0690 | 0.2760 | 0.0762 | 0.0690 | 0.1438 |

At a 5% significance level:

| Correction Method | Significant Visits |
|---|---|
| Bonferroni | 2 and 3 |
| Holm | 2 and 3 |
| Benjamini-Hochberg | 2 and 3 |
| Benjamini-Yekutieli | 2 and 3 |

The most stable visit-specific evidence therefore occurs at **Visits 2 and 3**.

Visit 1 illustrates why multiple-testing correction matters:

```text
Raw p-value        = 0.0381
BH-adjusted p      = 0.0508
Holm-adjusted p    = 0.0762
Bonferroni p       = 0.1524
```

A result that initially appears significant can therefore change once the complete family of hypotheses is considered.

---

## What Multiple Testing Does — and Does Not — Solve

The corrected visit-specific tests help control false-positive findings.

However, they still analyse each visit separately.

They do **not** provide a joint model of the four repeated measurements from each patient.

The next analytical question is therefore:

> **What is the overall association between treatment and respiratory status when all repeated observations are analysed together?**

This requires a longitudinal model.

---

# 7. Longitudinal Analysis with GEE

A **Generalized Estimating Equation (GEE)** model is used because:

- the outcome is binary,
- each patient is measured repeatedly,
- observations within the same patient may be correlated.

Patients are treated as clusters using the unique `person_id`.

A logistic GEE model estimates population-average associations while accounting for within-patient dependence.

---

## Initial Longitudinal Model

The initial model allows the treatment association to vary between visits using a:

**treatment × visit interaction**

This asks:

> Does the difference between active treatment and placebo systematically change across follow-up visits?

The interaction terms did not provide sufficient statistical evidence that the treatment association changed systematically between visits.

The simpler model also produced a preferable QIC value.

For this reason, the interaction was removed from the final model.

---

# 8. Parsimonious GEE Model

The simplified model contains:

- treatment
- visit
- age
- sex

and accounts for repeated observations within each patient using an exchangeable working correlation structure.

Conceptually:

```text
Respiratory status
        ~
Treatment
+ Visit
+ Age
+ Sex
```

with repeated observations clustered by patient.

The model estimates an **average treatment association across the four visits** rather than a separate treatment coefficient for every visit.

---

## Treatment Estimate

In the initial adjusted model with a linear age effect:

**Odds Ratio for Placebo vs. Active Treatment = 0.368**

**95% CI = 0.196–0.691**

**p = 0.00186**

This means that the placebo group had substantially lower estimated odds of a good respiratory status than the active-treatment group.

Equivalently, the active treatment was associated with considerably higher odds of a favourable respiratory outcome.

However, before accepting this as the final specification, the assumptions behind the model were examined.

---

# 9. Model Diagnostics and Sensitivity Analysis

One important modelling assumption concerns age.

The first adjusted model assumes that age has a **linear effect on the log-odds scale**.

This assumption is not guaranteed by the data.

I therefore fit a second GEE model allowing a more flexible, nonlinear age relationship using a natural spline.

The models were compared using QIC.

### Model Comparison

| Model | Approx. QIC | Treatment OR: Placebo vs Active |
|---|---:|---:|
| Linear age effect | 602 | 0.368 |
| Nonlinear age effect | 580 | 0.352 |

The nonlinear age specification has the lower QIC.

This suggests that a flexible representation of age describes the observed data better according to this criterion.

But model fit alone is not the main question.

The important sensitivity question is:

> **Does changing the age specification materially change the estimated treatment association?**

---

## Sensitivity Result

### Linear Age Model

- OR = **0.368**
- 95% CI = **0.196–0.691**
- p = **0.00186**

### Nonlinear Age Model

- OR = **0.352**
- 95% CI = **0.186–0.664**
- p = **0.00127**

The treatment estimate changes only slightly:

```text
0.368 → 0.352
```

The confidence intervals remain below 1 in both models.

The central treatment conclusion is therefore **robust to this modelling decision**.

---

# 10. Selected Final Model

For the final interpretation, the analysis uses the GEE model with:

- active treatment vs. placebo
- visit
- nonlinear age effect
- sex
- patient-level clustering
- exchangeable working correlation

The final model gives:

**Odds Ratio for Placebo vs. Active Treatment = 0.352**

**95% CI = 0.186–0.664**

**p = 0.00127**

An OR below 1 indicates lower odds of a good respiratory status in the placebo group.

Expressed relative to the active-treatment group:

```text
1 - 0.352 = 0.648
```

The estimated odds of a good respiratory status are therefore approximately **64.8% lower under placebo than under active treatment**, based on this model parameterisation.

This is an odds comparison and should not be interpreted as a 64.8 percentage-point difference in probability.

---

# Final Interpretation

The project provides a consistent analytical picture.

### Descriptive evidence

The active-treatment group had a higher observed proportion of good respiratory outcomes at all four visits.

### Visit-specific inference

Four separate comparisons initially suggested differences at Visits 1, 2 and 3.

After correcting for multiple testing, **Visits 2 and 3 remained statistically supported across all four correction methods**.

### Longitudinal evidence

The GEE model accounts for the fact that each patient contributes four correlated measurements.

The final longitudinal model finds substantially lower odds of a good respiratory status under placebo than under active treatment:

**OR = 0.352, 95% CI = 0.186–0.664, p = 0.00127**

### Robustness

Allowing age to have a nonlinear relationship with the outcome improved the QIC but changed the treatment estimate only slightly.

This indicates that the central treatment conclusion is not driven by the original assumption of a linear age effect.

---

# Why Both Analyses Are Needed

The visit-specific tests and the GEE model answer different questions.

| Analysis | Question |
|---|---|
| Descriptive proportions | What do the observed group differences look like? |
| Visit-specific tests | At which individual visits is there evidence of a group difference? |
| Multiple-testing correction | Which visit-specific findings remain after accounting for four simultaneous tests? |
| GEE model | What is the average treatment association across all repeated measurements? |
| Sensitivity analysis | Does the treatment conclusion depend strongly on how age is modelled? |

The GEE model is therefore the **main longitudinal analysis**.

The visit-specific tests provide complementary information about individual time points.

---

# What This Project Demonstrates

This project demonstrates more than applying individual statistical tests.

## Data Analysis

- identifying the correct observational unit
- handling repeated measurements
- constructing unique patient identifiers
- validating longitudinal data structures
- analysing binary clinical outcomes
- comparing outcomes across treatment groups and visits

## Statistical Reasoning

- distinguishing repeated-measures dependence from multiple testing
- defining a family of related hypotheses
- controlling FWER and FDR
- comparing statistical conclusions under different correction procedures
- interpreting raw and adjusted p-values

## Longitudinal Modelling

- Generalized Estimating Equations
- logistic models for repeated binary outcomes
- patient-level clustering
- working correlation structures
- treatment × time interaction
- parsimonious model selection

## Model Evaluation

- QIC-based model comparison
- sensitivity analysis
- linear vs. nonlinear covariate specification
- Pearson residual diagnostics
- assessment of stability of the main treatment estimate

## Communication

- separating descriptive, visit-specific and longitudinal conclusions
- interpreting odds ratios and probabilities carefully
- communicating uncertainty
- documenting modelling decisions
- creating a reproducible Quarto workflow

---

# Limitations

This is an exploratory analysis of a relatively small clinical dataset.

Important limitations include:

- only **111 patients** are included,
- each patient contributes four observations,
- the sex distribution is imbalanced,
- the results describe population-average associations rather than individual predictions,
- GEE results depend on the chosen model specification and working correlation structure,
- QIC supports model comparison but does not prove that one model represents the true data-generating process,
- statistical association alone should not be interpreted as an unrestricted causal conclusion,
- the results should not be interpreted as individual medical treatment recommendations.

The multiple-testing corrections also apply specifically to the four visit-specific tests.

They do not replace the longitudinal GEE model.

---

# Reproducibility

The complete analysis is implemented in **R and Quarto**.

Main packages include:

```r
library(geepack)
library(dplyr)
library(tidyr)
library(ggplot2)
library(splines)
```

The analysis can be reproduced by loading the `respiratory` dataset directly from `geepack`:

```r
data("respiratory", package = "geepack")
```

No individual patient data need to be uploaded separately to the repository.

The Quarto analysis contains:

```text
Data validation
        ↓
Descriptive analysis
        ↓
Visit-specific tests
        ↓
Multiple-testing correction
        ↓
Longitudinal GEE modelling
        ↓
Model comparison
        ↓
Sensitivity analysis
        ↓
Final interpretation
```

---

# Tech Stack

**R** · **Quarto** · **geepack** · **dplyr** · **tidyr** · **ggplot2**

### Statistical Methods

- Proportion tests
- Bonferroni correction
- Holm correction
- Benjamini-Hochberg FDR
- Benjamini-Yekutieli FDR
- Generalized Estimating Equations
- Logistic longitudinal modelling
- Treatment × visit interaction
- QIC model comparison
- Natural splines
- Sensitivity analysis
- Pearson residual diagnostics

---

# Data Source

The analysis uses the publicly available `respiratory` dataset distributed with the R package `geepack`.

The dataset contains repeated binary respiratory outcomes from patients assigned to active treatment or placebo and observed over four follow-up visits.

---

# Project Purpose

This project was developed as a **learning and portfolio project in longitudinal clinical data analysis**.

The objective was not simply to identify statistically significant p-values.

The project demonstrates a structured analytical process:

> **Clinical Question → Data Structure → Multiple-Testing Problem → Repeated-Measures Problem → Longitudinal Model → Diagnostics → Sensitivity Analysis → Final Interpretation**

The main lesson is that different statistical problems require different solutions:

> **Multiple tests require error-rate control. Repeated observations require a model that accounts for within-patient dependence.**

The final conclusion therefore does not rely on a single p-value, but on a documented sequence of data validation, multiple-testing correction, longitudinal modelling, model comparison and sensitivity analysis.
