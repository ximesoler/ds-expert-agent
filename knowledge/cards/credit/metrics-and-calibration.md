---
id: credit.evaluation.gini-ks-calibration
title: Credit scoring reports Gini/KS for ranking and calibration by score band for pricing
stage: [evaluation]
family: [classification]
domain: [credit]
type: metric
applies_when: "probability of default, credit scoring, or lending decisions"
enforce: evaluator
confidence: baseline
sources:
  - "Siddiqi — Credit Risk Scorecards (2006); Basel II/III model validation practice"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Ranking power is reported as Gini (2·AUC−1) and KS; calibration as observed vs predicted default rate per score band (Hosmer-Lemeshow style table). Stability of the score distribution (PSI) between development and recent samples is part of the acceptance criteria.
