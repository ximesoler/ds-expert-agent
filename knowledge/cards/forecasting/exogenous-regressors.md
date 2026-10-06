---
id: forecasting.features.known-future-only
title: Use exogenous drivers only if their future values are known at forecast time
stage: [feature_understanding, modeling]
family: [forecasting]
domain: [any]
type: pitfall
applies_when: "candidate regressors present (price, promo, weather, macro)"
enforce: reviewer
confidence: baseline
sources:
  - "Hyndman & Athanasopoulos — FPP3, ch. 10 dynamic regression"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Split regressors into known-future (calendar, holidays, planned promotions, scheduled prices) and unknown-future (weather, competitor actions, actual traffic). The second group can only enter through lags or through a separate forecast of the regressor itself.

**Why.** The classic leak: a model that uses same-day weather looks brilliant in backtest and cannot run in production.

**How to apply.** Add a "known at forecast time?" column to the feature inventory; the reviewer fails any feature without a yes.

**Ask the user.** "Which of these will you know in advance: promotions calendar, price changes, store openings, marketing spend?"
