---
id: churn.business.uplift-vs-propensity
title: '"Who will churn" and "who should receive the offer" are different models'
stage: [business, evaluation]
family: [classification]
domain: [churn]
type: question
applies_when: "a retention action with a cost is planned"
enforce: gap_detector
confidence: baseline
sources:
  - "Ascarza — Retention futility: targeting high-risk customers might be ineffective, JMR (2018)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Targeting the highest-risk customers often wastes offers on lost causes and on people who would have stayed anyway. If historical campaign data with a control group exists, build an uplift model; otherwise ship propensity with a plan to run a holdout and collect uplift data.

**Ask the user.** "What will you do with the list, and has that action been tried before with a control group?"
