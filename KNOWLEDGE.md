# Expert Knowledge Layer

**Goal:** give every expert agent the tacit knowledge a senior practitioner carries (what to look at, what to ask, what to never do, which metric fits which decision), start from a curated baseline, and let the owner enrich it continuously from LinkedIn posts, papers, and lessons learned in real projects.

The guiding idea: **knowledge is data, not prompt text.** It lives in versioned files with a schema, is filtered by context (stage × problem family × domain), and reaches the agents through four well-defined channels. Enriching it never means editing a system prompt.

---

## 1. Unit of knowledge: the card

One file = one card = one actionable claim. Markdown body, YAML header for filtering.

```markdown
---
id: fraud.features.atypical-behavior
title: Score deviation from the entity's own baseline, not absolute values
stage: [feature_engineering, eda] # business | data | eda | feature_understanding | feature_engineering | modeling | evaluation | deployment | monitoring
family: [classification, anomaly] # classification | regression | forecasting | anomaly | any
domain: [fraud]                   # fraud | demand | churn | credit | pricing | maintenance | marketing | healthcare | any (exclusive)
type: heuristic                   # principle | heuristic | checklist | question | metric | pitfall | technique | reference
applies_when: "entity id (customer/card/account) is present and history length > 1 event"
enforce: advisory                 # advisory | gap_detector | reviewer | evaluator
confidence: baseline              # baseline | practitioner | validated
sources:
  - "Le Borgne, Siblini, Lebichot, Bontempi — Reproducible ML for Credit Card Fraud Detection, Practical Handbook (2022)"
added_by: baseline
added_on: 2026-10-05
---

**Claim.** …one paragraph.
**Why.** …
**How to apply.** …concrete steps the expert should take.
**Pitfalls.** …
**Ask the user.** …questions this card raises (only for type=question or when relevant).
```

Field semantics that matter for the machinery:

| Field | Used by |
|---|---|
| `stage`, `family`, `domain` | retrieval filter; the orchestrator knows all three after stage 1 |
| `applies_when` | plain-language trigger; the orchestrator evaluates it against the brief + data card and drops cards that don't apply, so playbooks stay short |
| `enforce` | **advisory** = goes into the expert's playbook; **gap_detector** = adds a question to the requirements schema; **reviewer** = becomes a mandatory checklist item the reviewer must PASS/FAIL; **evaluator** = metric/translation rule |
| `confidence` | **baseline** = shipped with the repo; **practitioner** = added by the owner from a post/paper/experience; **validated** = confirmed on a real project (promoted via `dsx learn promote`) |
| `sources`, `added_by` | provenance; shown in reports when a card drove a decision |

---

## 2. Four channels from knowledge to behavior

```
knowledge/cards/*.md
      │
      ├─(1) compile ──► skills/<expert>-playbook/SKILL.md   preloaded into AgentDefinition.skills
      │                 (filtered by family/domain at run start; ≤ ~2k tokens per expert)
      │
      ├─(2) retrieve ─► mcp__ds__knowledge_search(stage, family, domain, query)
      │                 orchestrator injects top cards into each delegation prompt;
      │                 experts can query mid-task ("anything on intermittent demand metrics?")
      │
      ├─(3) questions ► cards with enforce=gap_detector extend schemas/<family>.yaml
      │                 → the business-analyst asks domain questions a human expert would ask
      │
      └─(4) checks ───► cards with enforce=reviewer|evaluator become checklist items
                        the reviewer must tick; a missed item is a FAIL with the card id cited
```

Why four channels and not just "put it in the prompt": playbooks give always-on priors; retrieval keeps context small when the base grows to hundreds of cards; questions make knowledge *interactive*, which is the whole point of this agent; checks make the most important knowledge *non-negotiable* instead of a suggestion the model may ignore.

### Compile step (`dsx knowledge build`)
- Maps card `stage` values onto the plugin's 4 stages (business → 1 Business; data, eda, feature_understanding → 2 Data; feature_engineering, modeling, evaluation → 3 Experiment; deployment, monitoring → 4 Ship).
- Reads all cards, validates the header against `knowledge/card.schema.yaml`, fails on duplicate ids and on contradictions that are *not* listed in `conflicts.yaml`.
- Writes one playbook per expert: cards whose `stage` maps to the plugin stages the agent works in (analyst: 1; data-expert: 2; developer/reviewer/evaluator: 3–4; writer: all), grouped by type, ordered by `enforce` severity then confidence. Cards with `applies_when` are kept with their trigger so the expert self-filters.
- Writes `schemas/<family>.generated.yaml` (gap-detector questions) and `checklists/<stage>.generated.md` (reviewer/evaluator items).
- Builds a small lexical index (BM25 over title+body+tags) for `knowledge_search`; no vector DB in v1.

### Runtime
- After Gate 1 the orchestrator knows `family` and `domain`. It calls `knowledge_search` at each call point (technical spec, build, interpretation) and pastes the top-k card bodies into the delegation prompt under a "Domain knowledge to apply" heading, with ids so the expert can cite them.
- Technical specs and build records list the cards that influenced decisions as `applied_cards: [{id, hash}]`. This is what later lets you see which knowledge actually mattered.

---

## 3. Growth loop: how the base gets enriched

| Entry point | What happens |
|---|---|
| `dsx learn add <url \| file \| ->` | A `knowledge-curator` subagent reads the LinkedIn post / paper / your notes, drafts 1–N cards (claim, conditions, applies_when, enforce suggestion, sources), shows them for review. You edit/approve; saved with `confidence: practitioner`. |
| End of a project (after Gate 4) | Orchestrator mines `decisions.log` and the reports for lessons ("structural break handling via event indicator beat dropping the window") and proposes cards. Approve → saved with the project slug as source. |
| `dsx learn promote <id>` | Marks a card `validated` after it proved useful on a real project. |
| `dsx knowledge lint` | Finds duplicates, cards that never triggered in N runs, cards whose `applies_when` is unparseable, and pairs listed in `conflicts.yaml`. |
| `dsx learn research <topic>` | Optional: a research subagent with web search drafts cards from academic/public sources for a topic you name, always human-reviewed before saving. |

Rules: human review before anything lands; provenance mandatory; LinkedIn content is pasted by you, not scraped; cards are git-versioned so the base has history and can be shared across teammates.

### Keeping the base honest
- **Knowledge regression tests.** `tests/knowledge/` holds toy projects with expected behaviors: the fraud toy must ask about analyst alert capacity; the imbalanced toy must not report accuracy as the headline metric; the intermittent-demand toy must not use MAPE. Each test cites the card ids it exercises, so deleting a card breaks a test.
- **Usage telemetry.** `applied_cards` in reports feeds a simple counter; `lint` shows dead cards.
- **Contradictions are allowed but explicit.** Two cards can disagree (e.g., SMOTE vs class weights); `conflicts.yaml` records the pair and the resolution rule ("prefer class weights for GBMs, SMOTE only when the user insists").

---

## 4. Taxonomy and baseline coverage

Cards shipped as baseline live in `knowledge/cards/<domain-or-topic>/`. Coverage at v0.1:

| Area | Cards (39) |
|---|---|
| `forecasting/` (9) | signal-vs-noise, beat-the-naive, intermittent-demand, backtesting (rolling-origin), exogenous-regressors (known-future-only), outliers-vs-events, prediction-intervals (calibrated quantiles), hierarchy-and-grain, demand-vs-sales |
| `imbalanced/` (5) | metrics, threshold-by-cost, calibration, resampling-pitfalls, stratified-cv (stratified + grouped) |
| `fraud/` (7) | atypical-behavior-features, time-of-day, label-delay, feedback-loop (selection bias), alert-capacity-metrics, hybrid-rules-model, adversarial-drift |
| `churn/` (4) | snapshot-framing, post-outcome-leakage, uplift-vs-propensity, lift-metrics |
| `credit/` (3) | metrics-and-calibration (Gini/KS), monotonic-and-explainable, reject-inference |
| `data-quality/` (5) | leakage-patterns, train-serve-skew, keys-duplicates-grain, missingness-mechanism, units-and-semantics |
| `modeling/` (4) | baseline-first, gbm-default-tabular, cv-matches-deployment, importance-is-not-causality |
| `deployment/` (2) | monitoring-plan, champion-challenger |

Each card below is deliberately short; the body is what the expert reads, the header is what the machinery uses.
