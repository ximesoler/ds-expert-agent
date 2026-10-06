---
id: data.eda.missingness-mechanism
title: Missingness is information; test whether it depends on the target before imputing
stage: [eda, feature_engineering]
family: [any]
domain: [any]
type: heuristic
applies_when: "columns with > 1% missing"
enforce: advisory
confidence: baseline
sources:
  - "Rubin — Inference and missing data (1976); Little & Rubin — Statistical Analysis with Missing Data"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Compare the target rate for missing vs present per column. If they differ, add a missing indicator and treat the pattern as a feature. Impute inside the training fold; trees can take NaN directly (LightGBM) so prefer that over mean imputation.

**Ask the user.** "Why would this field be empty: not collected for some segments, not applicable, or system error?"
