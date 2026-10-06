---
id: fraud.features.atypical-behavior
title: Fraud shows as deviation from the entity's own normal; build features relative to that baseline
stage: [eda, feature_engineering]
family: [classification, anomaly]
domain: [fraud]
type: technique
applies_when: "entity id present (card, account, user, merchant, device) with history"
enforce: advisory
confidence: baseline
sources:
  - "Le Borgne, Siblini, Lebichot, Bontempi — Reproducible Machine Learning for Credit Card Fraud Detection, Practical Handbook (2022), ch. 3 feature engineering"
  - "Bahnsen, Aouada, Stojanovic, Ottersten — Feature engineering strategies for credit card fraud detection (2016)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Raw amount or merchant category is weak; "amount vs this card's 90-day mean and std", "first time at this merchant/country/device", "number of transactions in the last 1h/24h/7d vs usual", "distance from previous location and time since (impossible travel)" are the features that carry signal.

**How to apply.** For each entity compute rolling baselines strictly from the past (point-in-time correct), then express each event as z-scores, ratios, novelty flags, and velocity counts over several windows. Add peer-group baselines (same merchant segment, same customer cohort) for cold entities.

**Pitfalls.** Rolling windows that include the current event or future events; computing baselines over the whole dataset (leak); entities with little history get extreme z-scores, so cap and add a history-length feature.
