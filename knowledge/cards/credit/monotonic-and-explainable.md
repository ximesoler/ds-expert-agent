---
id: credit.modeling.monotonic-explainable
title: Enforce monotonic constraints and reason codes where adverse-action explanations are required
stage: [modeling, deployment]
family: [classification]
domain: [credit]
type: technique
applies_when: "regulated lending decision or a need to explain declines"
enforce: advisory
confidence: baseline
sources:
  - "Equal Credit Opportunity Act / Reg B adverse-action requirements; LightGBM/XGBoost monotone_constraints"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Use monotone constraints on features with an expected direction (more delinquencies never improves the score) so explanations are stable and defensible; produce top reason codes per decision; avoid protected attributes and their proxies, and test for disparate impact.

**Ask the user.** "Is this decision regulated? Must declines be explained to the applicant? Which variables are off-limits?"
