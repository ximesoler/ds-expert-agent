---
id: fraud.evaluation.alert-capacity
title: Optimize precision at the analysts' daily capacity and value-weighted recall, not AUC
stage: [business, evaluation]
family: [classification]
domain: [fraud]
type: metric
applies_when: "alerts are reviewed by humans or blocking has customer friction"
enforce: evaluator
confidence: baseline
sources:
  - "Le Borgne et al. Handbook (2022), ch. 4 performance metrics: precision top-k, card precision@k"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** If the team can review 200 alerts a day, the metric is precision@200/day (per card, not per transaction, since one alert blocks a card). Pair it with fraud-amount recall (share of fraudulent money caught) because a few large frauds dominate losses. Add false-positive rate on good customers as the friction cost.

**Ask the user.** "How many alerts per day can the team handle? Is the action blocking, a step-up check, or a review? What does a false alarm cost in customer terms?"
