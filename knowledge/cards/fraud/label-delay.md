---
id: fraud.data.label-delay
title: Fraud labels arrive late; split in time and leave a maturity gap
stage: [business, data, modeling]
family: [classification]
domain: [fraud]
type: question
applies_when: "labels come from chargebacks, disputes, or investigations"
enforce: gap_detector
confidence: baseline
sources:
  - "Dal Pozzolo, Boracchi, Caelen, Alippi, Bontempi — Credit card fraud detection: a realistic modeling and a novel learning strategy, IEEE TNNLS (2018)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** A transaction is only known to be fraud weeks later. The most recent weeks of data are mostly unlabeled-as-fraud, not legitimate. Train on data older than the label delay, validate on a later window with the same delay removed, and never use a random split.

**Ask the user.** "How is the fraud label produced and how long after the event does it become reliable? Are uninvestigated cases labeled as legitimate?"
