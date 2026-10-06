---
id: forecasting.business.hierarchy-and-grain
title: Forecast at the grain of the decision and reconcile across the hierarchy
stage: [business, modeling]
family: [forecasting]
domain: [demand]
type: question
applies_when: "data has multiple entities (SKU, store, region, warehouse)"
enforce: gap_detector
confidence: baseline
sources:
  - "Hyndman & Athanasopoulos — FPP3, ch. 11 forecasting hierarchical and grouped series"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** The grain of the forecast is a business decision (who orders, at what level), not a data property. When several levels matter, forecast at the levels and reconcile (MinT or simple top-down/bottom-up) so totals add up.

**Why.** Bottom-level series are noisy; top-level series hide mix. A plan that does not add up across warehouses gets rejected by finance.

**Ask the user.** "Who consumes the forecast and at what level: per warehouse, per SKU, total? Do the numbers need to add up across levels?"
