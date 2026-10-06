---
id: forecasting.eda.signal-vs-noise
title: Separate signal from noise before choosing any model
stage: [eda, modeling]
family: [forecasting]
domain: [any]
type: principle
applies_when: "any time series target"
enforce: reviewer
confidence: baseline
sources:
  - "Hyndman & Athanasopoulos — Forecasting: Principles and Practice, 3rd ed. (ch. 3 decomposition, ch. 5 benchmarks)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Decompose every series (STL) into trend, seasonality, remainder and quantify how much variance each explains before modeling. Series whose remainder dominates are mostly noise and will not reward complex models.

**Why.** The most common forecasting failure is fitting noise: a model that tracks the training history closely but cannot beat a seasonal naive on a held-out window. Measuring seasonal strength and trend strength (Hyndman's F_s, F_t) tells you up front what there is to learn.

**How to apply.** (1) STL per series or per aggregate; (2) compute strength of trend/seasonality; (3) compare the remainder variance to the signal variance; (4) check remainder autocorrelation (Ljung-Box) to see whether structure remains; (5) report the share of series classified as "mostly noise" so the user understands the ceiling.

**Pitfalls.** Decomposing at the wrong frequency hides seasonality; low-volume daily series look like noise but are predictable weekly. Aggregating before decomposing is often the right first move.

**Ask the user.** "Do you expect weekly or yearly patterns? Were there events that would show up as spikes (promotions, stock-outs, price changes)?"
