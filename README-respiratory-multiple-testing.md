# Multiple Testing and Longitudinal Respiratory Data

**An exploratory, reproducible R/Quarto analysis of repeated binary respiratory-status measurements.** The project demonstrates why visit-by-visit hypothesis tests need a stated multiplicity strategy, and why repeated observations from the same person call for a longitudinal model.

> **Read the analysis:** [Open the full Quarto report](LINK_TO_RENDERED_REPORT)

## Questions

1. Which visit-specific treatment comparisons remain statistically significant under different familywise-error and false-discovery-rate procedures?
2. What is the treatment association across visits when within-person dependence is modelled using generalized estimating equations (GEE)?

## Data at a glance

The report analyzes the `respiratory` dataset distributed with the R package [`geepack`](https://cran.r-project.org/package=geepack): **111 participants**, each observed at **four visits**, for **444 repeated measurements**. The binary outcome is coded as 1 for good respiratory status and 0 otherwise. The treatment groups are active treatment and placebo.

## Main results

### Visit-specific tests

The analysis compares four visit-specific group tests and adjusts their *p*-values using four procedures:

| Procedure | Error criterion | Visits significant at 5% in this analysis |
|---|---|---|
| Bonferroni | Familywise Error Rate (FWER) | 2 and 3 |
| Holm | FWER | 2 and 3 |
| Benjamini–Hochberg | False Discovery Rate (FDR) | 1 and 2 |
| Benjamini–Yekutieli | FDR; valid under general dependence | 2 and 3 |

The procedures do not answer the same error-control question, so their different visit-level conclusions are informative rather than interchangeable.

### Longitudinal model

The report fits logistic GEE models to account for repeated measurements within participants. In the adjusted model, the estimated odds ratio for placebo versus active treatment at visit 1 is **0.440** (95% CI: 0.199–0.974; *p* = 0.043), adjusted for age and sex. The treatment-by-visit interaction terms do not provide statistically sufficient evidence that the treatment association changes over visits. The report also fits a simplified model without the interaction; in that model the treatment association remains statistically detectable (OR **0.368**, *p* = **0.00186**).

The final model uses a nonlinear age term: its QIC is approximately **580**, compared with approximately **602** for the linear-age model. The estimated treatment odds ratio remains similar (**0.352** versus **0.368**), indicating that this estimate is not materially changed by that age specification in these data.

These are exploratory group-level associations in a small dataset, not individual predictions or clinical recommendations.

## Analysis workflow

- Validate outcome coding, treatment groups, visit structure, and participant identifiers; combine clinic and participant IDs because IDs are not globally unique across clinics.
- Describe the outcome by treatment and visit.
- Conduct four visit-specific group comparisons and report raw and adjusted *p*-values.
- Explain the mathematical distinction between FWER control (Bonferroni, Holm) and FDR control (Benjamini–Hochberg, Benjamini–Yekutieli).
- Fit unadjusted and age-/sex-adjusted logistic GEE models, accounting for participant-level repeated measurements.
- Compare working correlation structures and model specifications; report odds ratios, confidence intervals, and model-based probabilities.
- Check model diagnostics and conduct a sensitivity analysis comparing linear and nonlinear age effects using QIC.

## Reproduce the report

The report is authored in Quarto and uses R. To reproduce it from the repository root:

1. Install R and [Quarto](https://quarto.org/docs/get-started/).
2. Install the packages used by the source document (including `geepack`, `dplyr`, and `ggplot2`; check the `.qmd` for the complete list).
3. Render the source file:

   ```bash
   quarto render PATH_TO_SOURCE_FILE.qmd
   ```

Replace `PATH_TO_SOURCE_FILE.qmd` with the actual committed Quarto filename. The rendered HTML and its supporting `*_files/` directory should be committed together if the report is meant to open directly on GitHub Pages or another host.

## Interpretation and limitations

- The dataset contains 111 participants, so estimates—especially adjusted estimates—have meaningful uncertainty.
- The sex distribution differs between the treatment groups; age and sex are included in the adjusted model, but adjustment does not guarantee elimination of confounding.
- Visit-specific tests and the joint GEE model answer different questions. Multiplicity corrections do not account for within-person dependence; GEE addresses the repeated-measure structure.
- QIC supports comparison among the candidate GEE models considered; it does not establish that a model is true.
- Findings are exploratory group comparisons and should not be interpreted as individual treatment advice.

## Tools

R · Quarto · `geepack` · `dplyr` · `ggplot2` · multiple-testing correction · logistic GEE · longitudinal data analysis · model diagnostics

## Repository contents

Update this list to match the files committed to the repository:

- `PATH_TO_SOURCE_FILE.qmd` — Quarto source for the analysis
- `PATH_TO_RENDERED_REPORT.html` — rendered report (if included)
- `PATH_TO_RENDERED_REPORT_files/` — supporting assets generated by Quarto (if included)
