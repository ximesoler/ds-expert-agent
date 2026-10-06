---
id: forecasting.evaluation.calibrated-intervals
title: Inventory and capacity decisions need calibrated quantiles, not point forecasts
stage: [modeling, evaluation]
family: [forecasting]
domain: [demand]
type: metric
applies_when: "decision involves stock, capacity, staffing, or budget buffers"
enforce: evaluator
confidence: baseline
sources:
  - "Gneiting & Raftery — Strictly proper scoring rules (2007)"
  - "M5 uncertainty competition (Makridakis et al., IJF 2022)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Produce the quantiles the decision needs (e.g., q50 and the service-level quantile q95) and evaluate them with pinball loss and empirical coverage. A 95% interval that covers 80% of outcomes is a stock-out machine.

**Why.** Safety stock is a quantile of demand over lead time. The mean forecast alone cannot set it.

**How to apply.** Report coverage per horizon and per segment; penalize over- and under-coverage; translate to expected stock-out and holding cost using the user's cost inputs.

**Ask the user.** "What is the cost of a stock-out versus holding one extra unit for a month?"
