---
id: fraud.deployment.hybrid-rules-and-model
title: Keep interpretable rules alongside the model and give investigators reason codes
stage: [modeling, deployment]
family: [classification, anomaly]
domain: [fraud]
type: technique
applies_when: "human investigators consume alerts or regulation requires explanation"
enforce: advisory
confidence: baseline
sources:
  - "Lundberg & Lee — SHAP (2017); industry practice described in the Fraud Detection Handbook ch. 7"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Rules catch known patterns instantly and are auditable; the model catches novel combinations. Serve both, and attach the top SHAP features as reason codes so an investigator can act in seconds. Add an unsupervised score (isolation forest on the behavioral features) to surface patterns the labels do not yet contain.
