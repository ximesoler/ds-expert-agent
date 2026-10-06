---
id: modeling.tabular.gbm-default
title: Gradient-boosted trees are the default for tabular data; deep learning must earn its place
stage: [modeling]
family: [classification, regression]
domain: [any]
type: heuristic
applies_when: "tabular features, < ~10M rows"
enforce: advisory
confidence: baseline
sources:
  - "Grinsztajn, Oyallon, Varoquaux — Why do tree-based models still outperform deep learning on tabular data? (2022)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Start with LightGBM/XGBoost with sensible defaults and early stopping; tune only after features are stable; use linear models when interpretability or extrapolation matters. Report the spread across CV folds and seeds so small differences are not sold as improvements.
