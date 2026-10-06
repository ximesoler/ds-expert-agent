---
id: fraud.data.selection-bias
title: Only alerted cases get investigated; the labels reflect yesterday's rules, not the fraud population
stage: [business, data, evaluation]
family: [classification]
domain: [fraud]
type: pitfall
applies_when: "an existing rules engine or model already blocks or flags transactions"
enforce: gap_detector
confidence: baseline
sources:
  - "Dal Pozzolo et al. (2018); Le Borgne et al. Handbook, ch. 2 on the fraud detection loop"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Fraud that the current system blocks never appears as fraud in the data; fraud it misses may be labeled only if the customer disputes. The new model learns to imitate the old rules. Mitigate by keeping a small random audit sample, by modeling on pre-block data where possible, and by evaluating recall on customer-reported fraud separately.

**Ask the user.** "Is there already a rules engine or model in production? Are blocked transactions in the data? Do you have any randomly reviewed sample?"
