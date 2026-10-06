---
id: forecasting.business.demand-vs-sales
title: Sales are censored demand; decide how stock-outs are handled before modeling
stage: [business, data]
family: [forecasting]
domain: [demand]
type: question
applies_when: "target is sales/units sold and inventory can run out"
enforce: gap_detector
confidence: baseline
sources:
  - "Nahmias — Demand estimation in lost sales inventory systems (1994)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** When an item is out of stock, recorded sales are zero regardless of demand. Training on raw sales teaches the model that demand is low exactly when it was high. Either use stock-out flags to mask/impute those periods or state the assumption "sales == demand" explicitly in the brief.

**Ask the user.** "Does the data contain stock-out or availability information? When a product ran out, what did the sales column record?"
