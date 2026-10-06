---
id: data.features.leakage-patterns
title: "Hunt for leakage: features from the future, from the label, or from the ID"
stage: [eda, feature_engineering, modeling]
family: [any]
domain: [any]
type: checklist
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Kaufman, Rosset, Perlich — Leakage in data mining: formulation, detection, and avoidance (2012)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Reviewer checklist: (1) every feature has a point-in-time justification; (2) no feature is a transformation of the target or produced by the labeling process; (3) IDs, row order, file names, and timestamps are not features unless proven legitimate; (4) a model with suspiciously high validation performance (far above the baseline and the literature) is treated as leaky until a feature ablation explains the gain; (5) target encoding and imputation are fitted inside the training fold only.
