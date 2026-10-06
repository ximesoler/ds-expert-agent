# Execution walkthrough — how the cycle runs inside a stage and how stages connect

**Status:** v1.1 (2026-10-06, verb grammar + per-stage gap detector). Companion to `specs/SPECS.md` v2.0 and `docs/UX-AND-EXECUTION.md` v0.2. Example: "estimate inventory for product X for the next months, I have two years of sales".

## 1. Where business understanding lives

Stage 1 has no technical spec, plan or build. Everything it collects is stored in **three places**, all on disk:

| What | Where | Written by |
|---|---|---|
| The understanding itself: decision, target, success, costs, scope, data declared, assumptions, gaps, consumers | `specs/project.functional.md` (sections 1–10, header with provenance and `pending_data`) | the `ds-dialogue` skill, via `dsx` |
| The conversation that produced it: every question wave, every answer, every default taken, who decided | `decisions.log` (append-only; each wave is a block `wave 1-1 … wave 1-3`) | `dsx wave` |
| The proof it was approved and under which knowledge: section hashes, requirements schema id + hash, card-set hash, persona, wave ranges, approver | `specs/01-business/manifest.yaml` | `dsx gate approve 1` |

Nothing of stage 1 lives in chat history. A restart reads the functional file and the manifest; the dialogue can resume from the open gaps. **Stage 1 reads no data**: no file paths, no profiling; §6 records in prose what the user believes exists.

Example after Gate 1 (abridged):

```
specs/project.functional.md
---
version: 1 · status: approved · persona: business · family: forecasting · domain: demand
provenance: {requirements_schema: {id: forecasting-v1, hash: 3f…}, cards_set_hash: 9a…, persona: business, waves: [L12-L48, L49-L80]}
pending_data: [g03_stockout_censoring, g07_calendar_gaps]   # asked by the Data gap detector or closed by Data plan 1
---
## 1. Decision   reorder quantity per warehouse; planner; monthly; 2 weeks before month start; today: last-year-same-month × 1.1
## 2. Target     units sold per SKU × warehouse × week; horizon 12 weeks; demand may be censored by stock-outs (pending_data g03)
## 3. Success    (a) stock-out rate < 3% at ≤ current holding cost; (b) pinball loss q50/q90 by horizon; translation: expected stock-out cost = …; (c) seasonal naive; (d) ship if (a) met on backtest; stop if no model within 5% of seasonal naive
## 4. Costs      stock-out = 4× holding cost per unit-month; batch monthly; no latency constraint; no excluded variables
…
## 8. Gaps       g01 decision cadence · must_ask · answered · "monthly" · user · wave 1-1
                 g03 stock-out censoring · infer_from_data · pending_data · — · —
                 g11 service level · safe_default · defaulted · "95%" · policy · wave 1-2
```

## 2. The cycle inside a stage (stages 2–4), step by step

The same four steps run in every stage; only the content differs. `/ds:next` executes one step per call. Each step reads and writes specific files.

```
step            who runs it                              reads                                  writes
─────────────── ──────────────────────────────────────── ────────────────────────────────────── ───────────────────────────────────────
/ds:spec        dsx spec gen (free skeleton) → gap       SPECS.md §<stage> + cited project §§,  specs/0n-stage/functional.md (v1, approved_by: user,
 --functional   detector: schema slots + panel proposals  upstream deliverables, gap cards,       ## User knowledge hashed), decisions.log waves
                (dsx gaps propose/merge) → waves to user  user text
/ds:spec        panel: contributors (parallel JSON) →     functional.md (+ User knowledge hash),  specs/0n-stage/technical.md (v1, panel_log,
 --technical    lead drafts; senior confirms; others see  upstream deliverables, cards            §Method with owner_expert)
                a card
/ds:plan        dsx plan consult → owners in §Method →    technical.md, registry pins, last       specs/0n-stage/plans/<n>.md (status: planned,
                planner merges/ranks; card                build header, budget, user hypothesis  owner per task, consulted:)
/ds:build       task owners (DAG, ≤3 parallel) →  plans/<n>.md, technical.md             code under src/ features/ experiments/ deploy/,
                → expert interpretation        → dsx expect/dispatch/fetch            specs/0n-stage/builds/<n>.md (header by dsx build finish),
                → dsx build finish                                                    registry entries, decisions.log (verdict)
/ds:gate        dsx dod → writer → user        builds/*, technical DoD, registry      specs/0n-stage/manifest.yaml, reports/0n-stage.md,
                → dsx gate approve                                                    .dsx/snapshots/<id>, project.yaml cursor++
```

Loop rule: after `build`, the verdict decides: `iterate` → another `plan`; `change_method` → `spec:technical` v+1; `done` + DoD met → `gate`. The stage's functional is regenerated only when a project section it cites changes.

### 2.1 Stage 2 (Data) on the inventory example

```
/ds:next  → "→ running /ds:spec --functional --data"
   skeleton already chained at gate approve 1 (purpose: prove §2 target computable at SKU×warehouse×week; deliverables: data_card … split_recommendation; acceptance from SPECS §2)
   dsx schema select forecasting demand --stage 2 → slots [units, tz, currency]
   dsx gaps propose --stage 2 → data-expert + analyst (haiku, parallel) → .dsx/gaps/2/*.json
       data-expert: g20 "which files, grain, date column?" must_ask · g21 "what does sales record on stock-out days?" must_ask(if known)/pending_data · g22 "promo calendar known ahead?" must_ask · g23 "returns negative or separate?" safe_default
   dsx gaps merge → §Gaps (12 max) ; wave 2-1 (≤4, needs_data: false): g20, units, g22, g21
   user: "data/raw/sales.csv daily per sku×warehouse; promos.xlsx · units · yes, a quarter ahead · I don't know"
   → ## User knowledge (hashed); g21 → pending_data; functional v1 approved_by: user
   Done: Data functional v1 — demand per SKU×warehouse×week from sales.csv + promos.xlsx (asked 3, deferred 1; panel: data-expert, analyst)
   Next: /ds:spec --technical --data (≈$0.40) · alt: /ds:next
/ds:next  → "→ running /ds:spec --technical --data"
   knowledge_search(stage=2, family=forecasting, domain=demand) → cards [keys-and-grain, units-and-semantics, demand-vs-sales, intermittent-demand, signal-vs-noise, outliers-vs-events]
   contributors (parallel, sonnet/low): reviewer → DoD candidates with verification; analyst → domain constraints from cards
   lead data-expert (opus/medium) → technical.md v1: tables {sales.csv grain=SKU×warehouse×day, key=(sku,wh,date)}, target rule "sum units per ISO week; mask days with on_hand=0 (pending g21)",
                        analysis list [STL, intermittency, anomalies, coverage, breaks, calendar], inventory schema, §Method {profile: data-expert, checks: developer, analyses: data-expert}, DoD with verification, panel_log
   persona=business → card shown ("Drafted by the data expert · checked by reviewer (2 checks added)"); not asked to confirm
/ds:next  → "→ running /ds:plan --data"
   dsx plan consult --stage 2 → owners in §Method: data-expert, developer (cap 2) → each ≤2 {task, est_usd}
   planner (haiku) → plans/1.md: hypothesis "sales.csv has key (sku,wh,date) unique; weekly target computable ≥ 99.9%"; tasks (owner): profile (data-expert) → checks (developer) → target (developer) → hash (dsx) → pending_data wave (main thread); limits {max_usd: 0.5}
   card: "Who acts: data expert profiles · developer writes checks · reviewer verifies · cost ≈ $0.40 · ~4 min — Run it? (yes/change/skip)"
/ds:next  → "→ running /ds:build"  (iteration 1)
   dsx expect builds/1.md src/data/*.py
   DAG by owner: data-expert → profile.json ; developer → src/data/checks.py, build_target.py  (JSON: files, how_to_run)
   reviewer  → runs tests/data/test_checks.py → JSON [{local.S2.keys-unique, PASS, checks/keys.md, sha}, …]
   dsx dispatch (LocalVenvAdapter) → run r-021 → fetch metrics {dup_rows: 0, join: n/a, target_null_rate: 0.0004, stockout_days: 212}
   batched pending_data wave (main thread, AskUserQuestion): "212 days show on_hand = 0 with sales = 0. Treat demand as censored on those days?" → user: yes
      → dsx: project.functional §7 += "demand censored on stock-out days (confirmed_by_user)", §8 g03 answered; version 2 (sections 7,8 only; Gate 1 not reopened)
   data-expert interprets → JSON {findings, verdict: iterate, new_gaps: []}
   dsx build finish → builds/1.md (header: status pass, verdict iterate, dod_progress {met: 5, unmet: 4}); registry data_card v1, target h2
/ds:next  → {next: plan}   plans/2.md: hypothesis "≥ 60% of SKU×wh series have weekly seasonality strength > 0.4; ≤ 10% intermittent"
/ds:next  → build 2 … interpretation: 38 series, 71% seasonal, 7 intermittent (ADI 1.6), 3 structural breaks (2025-03, 2025-11, 2026-01) → new_gaps [g12 break causes: must_ask]
   wave → user: "Mar-2025: promotion; Nov-2025: warehouse move; Jan-2026: unknown" → anomalies classified (event, regime, unknown→indicator)
   verdict iterate → plans/3.md (inventory + split recommendation) → build 3 → verdict done
/ds:next  → {next: gate}
   dsx dod 2 → all [test] items PASS, [user] items answered → manifest.yaml (pins data_hash, deliverable hashes, cards_set_hash)
   writer → reports/02-data.md (persona-adapted); AskUserQuestion: approve / revise WHAT / revise HOW / stop → approve
   dsx gate approve 2 → snapshot s-07, cursor → stage 3
```

### 2.2 Stage 3 (Experiment), abridged

```
spec gen 3        → functional.md: inputs pinned {feature_inventory h4, split_recommendation h5, target h2, project §1 §3 §4 v2}
spec technical 3  → developer drafts: protocol P (rolling-origin, 8 origins, h=12 weeks, gap 2 weeks, metric pinball q50/q90, benchmarks naive/snaive/AutoETS, noise 2%),
                    allowed universe from inventory, ladder [snaive → AutoETS → LightGBM global], budget {max_experiments 6, max_usd 12}
                    → dsx registers protocol_P v1 (frozen at iteration 1)
plan 1            → (features v1: calendar + lags 1–8, snaive & AutoETS, P)
build 1           → developer writes features/v1/build_features.py + tests/test_point_in_time.py → dsx dod --pre → pit_certified ✔
                    → reviewer JSON → dsx dispatch (refuses if uncertified) → MLflow run → interpretation: AutoETS beats snaive by 9% → iterate
plan 2 / build 2  → (features v2: + rolling means + promo/move indicators, LightGBM global, P) → beats AutoETS by 6% (> noise) → champion
plan 3 / build 3  → exit iteration: evaluator (not the developer) translates from logged predictions: expected stock-out rate 2.4% at q90, holding cost −3% → go
gate 3            → manifest pins P hash, features/v2 hash, run id, env lock, cost inputs; business owner: ship
```

Changing the horizon later (`/ds:change` → §2) marks Data (coverage) and Experiment (P, backtests) stale with costs; Ship untouched; only the affected plans rerun.

## 3. How stages connect

Stages are connected by **deliverables and pins**, not by chat context:

```
project.functional.md ──§1 §2 §3 §5 §6──► 02 Data ──data_card · target · feature_inventory · split_recommendation──► 03 Experiment
         │                                                                                                   │
         ├──§1 §3 §4────────────────────────────────────────────────────────────────────────────────────────►│ (pins)
         │                                                                                                   ▼
         └──§1 §4 §9──────────────────────────────────────────────────────────────► 04 Ship ◄── protocol_P · features/v<k> · champion · business_metric
```

- A stage's generated functional lists exactly which project sections and which upstream deliverables it depends on (pins written by `dsx pin`).
- A stage's technical reads those deliverables by path and pins their hashes.
- A gate manifest freezes all pins. Downstream stages start from the manifest, so "connected" means "pinned to a hash that `dsx impact` can compare".
- Back-edges are verdicts: `change_purpose` edits a project section (new version, staleness by the section table); `back_to:2` reopens Data with a new plan; nothing is deleted.

## 4. What the user sees per stage (business persona)

| Stage | Questions | Approvals |
|---|---|---|
| 1 Business | 2–4 waves (≤ 4 questions each), no data | 1 (problema vigente) |
| 2 Data | gap-detector waves (files, units, stock-out meaning, drivers known ahead; 2–3 waves) + 1 batched pending-data wave after plan 1 + anomaly waves as EDA raises them | 1 (does the data support it) |
| 3 Modeling | gap-detector wave (budget, excluded families, segments that matter, cost inputs) | 1 (ship / iterate / stop, in business units) |
| 4 Ship | gap-detector wave (batch vs online, cadence, owner, alert channel) | 1 (accept handoff) |

A senior persona additionally confirms each technical spec and may edit any spec file; `/ds:sync` folds the edits back into the graph.
