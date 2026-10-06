---
id: data.deployment.train-serve-skew
title: Features must be computable at inference exactly as in training; prove it with a replay test
stage: [feature_engineering, deployment]
family: [any]
domain: [any]
type: checklist
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Breck, Cai, Nielsen, Salib, Sculley — The ML Test Score (2017)"
  - "Sculley et al. — Hidden technical debt in machine learning systems (2015)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** The same code path builds features for training and serving. The deployment stage includes a test that scores a historical batch through the API and reproduces the training-time predictions within tolerance.
