# Model and effort policy

**Status:** v1.0 (2026-10-06). Converged after a three-way debate (cost minimizer, quality guardian, routing engineer). Config in `config/policy.yaml`; routed at runtime by `dsx route`.

## 1. Principles

1. **Silent vs loud.** The engine makes an error loud only where a deterministic oracle exists (a test, a schema, a validator). The oracles themselves (target rule, protocol P, point-in-time rule, translation formula, the tests that encode them) are written by the model. A wrong oracle passes every test, and an escalation ladder cannot fire on it. Those **oracle call points** get a floor at Opus until mutation tests make them loud. Everything downstream of a correct oracle can be cheap.
2. **Cheapest model that is enough, with a ladder.** Loud call points start at Haiku or Sonnet and escalate on dsx-detectable triggers (schema invalid, validate fail ×2, tests fail after 2 attempts, reviewer FAIL twice on the same item, `done` with unmet DoD, user "revise").
3. **Effort ≤ high.** `xhigh` is never a default; it is reached only by escalation (validate fails twice at high, or the user revises a technical spec). Opus 5.5 defaults to `medium` and must be set explicitly.
4. **Fewer tokens beats cheaper tokens.** The engine runs every `[test]` DoD item itself (no model runs pytest); agents receive paths and pins, not bodies; results come back as JSON validated by `dsx check-report`; the main thread never loads build or report bodies.
5. **Downgrades are evidence-based, one tier at a time.** Loud points: first-try success ≥ 85% over ≥ 9 runs. Silent points: mutation catch rate 100% and zero disagreement on structured fields in shadow runs across ≥ 3 toy projects × 3 runs. `dsx route tune` proposes; a human approves.
6. **Judge cost per completed stage**, not per call. A cheap call that needs a reviewer round is not cheap.

## 2. Main thread (orchestrator session)

**Sonnet 5.5 / effort medium** throughout (keep medium, not low, during the stage-1 dialogue: it is the only judgment reading the user's answers when the analyst is Haiku). No reasoning-heavy work happens here: `dsx board --json` picks the step, `dsx validate` / `dod` / `build finish` enforce correctness, oracle work is delegated.

Inline (no delegation) only when output < 500 tokens and no data access: phrasing a question wave from `dsx wave next`, the gate menu, confirming a one-line verdict, confirming a plan hypothesis.

Never inline: draft a technical spec or protocol P; interpret results; write oracle tests; classify a gap `safe_default` outside the deterministic policy; load build or report bodies; decide staleness or "still valid"; resolve a reviewer FAIL by editing the DoD.

Per-step budget: ≤ 15k input (mostly cached session prefix), ≤ 1k output. `SessionStart` loads `dsx board --context` once (≤ 10k); `PreCompact → dsx digest` keeps it there.

## 3. Sign-off table (v1 defaults)

Haiku 4.5 has no `effort`; it uses `thinking.budget_tokens` (shown as `Nk`). "Oracle" rows are the silent points.

| Call point | Stage | Model / effort | Escalation trigger → target (max attempts) | Downgrade evidence |
|---|---|---|---|---|
| analyst.detect (persona, family, domain) | 1 | haiku / 2k | confidence < 0.7 or schema invalid → sonnet/low (2) | floor |
| analyst.prefill (slots) | 1 | haiku / 2k | validate consistency fail ×2 → sonnet/medium (2); non-verbatim values never close a `must_ask` slot | slot precision/recall ≥ opus − 2 pp |
| wave phrasing + answer consistency | 1 | main thread (sonnet/medium) | card flag → sonnet/medium subagent | question count per persona in range; gap-card catch 100% |
| gaps.propose (stage gap detector: each consulted expert proposes ≤ 8 candidate questions) | 2–4 | haiku / 2k | schema invalid → sonnet/low (2) | `must_ask` set identical vs opus on fixtures |
| technical.contribute (contributors return section contributions + DoD candidates, parallel, ≤ 3, cap ≈ $0.10) | 2–4 | sonnet / low | schema invalid → sonnet/medium | shadow diff on DoD lists |
| plan.consult (owners named in `§Method`, cap 2, each ≤ 2 hypotheses/tasks with cost) | 2–4 | haiku / 2k (Modeling: sonnet / low) | schema invalid → sonnet/low | rung/task choice matches opus |
| **technical S2 (target rule, PIT rule, analysis list)** — oracle | 2 | **opus / medium** | validate ×2 or senior "revise HOW" → opus/high (3) | mutation tests green + shadow diff on target rule / maturity gap / PIT rule = 0 → sonnet/high (forecasting first) |
| **technical S3 (protocol P, feature universe, ladder)** — oracle | 3 | **opus / high** (opus/medium when a senior confirms the HOW) | validate ×2 or user revise → opus/xhigh | shadow diff on scheme / gap / origins / min lag / noise threshold = 0 over 9 runs; infeasible-P fixture → opus/medium |
| technical S4 (serving contract, tests, monitoring) | 4 | sonnet / medium | validate ×2 → opus/medium (3) | replay / latency / dry-run pass rate |
| plan: Data, Ship | 2, 4 | haiku / 2k | non-numeric hypothesis ×2 → sonnet/low (3) | task list identical |
| plan: Experiment | 3 | sonnet / low | validate ×2 → sonnet/medium (3) | rung choice matches |
| build: Data plan 1, Ship plan 2 (parametrized library calls) | 2, 4 | haiku / 4k | tests fail ×2 → sonnet/medium (4) | floor |
| build: Data plan 2+, Ship plan 1 | 2, 4 | sonnet / medium | tests fail ×2 → opus/medium (4) | pass@1 ≥ opus − 5 pp |
| **build: oracle code** (`build_features` v1, `test_point_in_time`, `backtest`, `translate`) — oracle | 3 | **opus / medium** | mutation test fails or tests fail ×2 → opus/high (4) | `dsx mutate` kill rate 100% over 9 runs → sonnet/high |
| build: non-oracle Experiment code (features v2+, runs, configs, benchmarks) | 3 | sonnet / high (forecasting) · sonnet / medium (tabular) | tests fail ×2 → opus/medium → opus/high (4) | escalation rate < 10% |
| reviewer: `[test]` items | 2–4 | **dsx, no model** | — | — |
| reviewer: `[reviewer]` advisory items | 2–4 | haiku / 1k checklist JSON; sonnet/medium when a card flag fires ("suspiciously high performance", `questions-answered`, `sensitivity`) | contradicts a test outcome → sonnet/high (3) | kappa ≥ 0.9 vs opus; leaky-champion fixture flagged |
| interpretation S2, S4 | 2, 4 | sonnet / medium | required fields missing or `done` with unmet DoD → opus/medium (3) | verdict identical over 9 runs |
| **interpretation S3** — oracle-adjacent | 3 | **opus / medium**; dsx computes `beats_by > noise` and writes accepted/rejected deterministically; the model writes the recommendation and proposes the verdict | same → opus/high | shadow verdict agreement ≥ 95% over 45 runs → sonnet/medium |
| **evaluator exit** — oracle | 3 | **opus / medium** (one call per project; no direction-based escalation: the silent failure is a wrong *go*) | user "revise" at Gate 3 → opus/high (2) | shadow: business metric ± tolerance and worst-segment agreement → sonnet/high |
| writer (gate summaries, reports, model card) | 1–4 | haiku / 1k (sonnet/low for business persona at gates) | user "revise" ×2 → sonnet/low (3) | manifest-number inclusion check; judge rubric ≥ opus − 2 pp |
| curator (`/ds:learn`) | offline | sonnet / medium | conflict detected → opus/medium | `tests/knowledge/`, human review |
| revalidate (v2) | — | haiku / 1k may only propose *which nodes to rerun*; "still valid" is always a user pin | — | never auto |

Persona overrides: senior → technical drafts one notch lower (they edit and confirm); business → writer sonnet/low at gates. Family override: forecasting Experiment non-oracle builds at sonnet/high; tabular/churn at sonnet/medium.

## 4. Cost and latency (inventory example, one full run)

Assumptions: Data 2 plans, Experiment 3 rungs + exit, Ship 2 plans; ~44 subagent calls, ~40 main-thread steps; ~1.9M input tokens (mostly cache reads), ~240k output.

| | All Opus / high everywhere | This policy (v1) | After mutation tests (oracle rows → sonnet/high) |
|---|---|---|---|
| Main thread | ≈ $2.9 | ≈ $0.8 | ≈ $0.8 |
| Builds | ≈ $4.5–8.0 | ≈ $3.0–3.9 | ≈ $2.5 |
| Reviewer | ≈ $1.6–2.5 | ≈ $0.2–0.3 | ≈ $0.2 |
| Technical specs, interpretations, evaluator, small calls | ≈ $2.8–3.9 | ≈ $1.4–1.8 | ≈ $1.3 |
| **Total** | **≈ $12–14** | **≈ $5.4–5.8 (−55 to −60%)** | **≈ $4.8** |
| LLM wall-clock | ≈ 2.5 h | ≈ 1 h | ≈ 1 h |

The biggest lever is output tokens, not price per token: the engine executing tests, JSON results, and paths-not-bodies prompts. The panel steps added in v0.5 (gap proposals, technical contributors, plan consults) add ≈ $0.4–0.6 per run; waves, cards and footers are main-thread or dsx-rendered.

## 5. Caching

Each agent's stable prefix = frontmatter system prompt (~1.5k) + compiled stage × family playbook (2–4k) + report schema/template (~1k) ≈ 6–8k, with the variable part (paths, pins, hypothesis) last. Hit rates ≈ 0% on the first call of a step (5-minute TTL), 60–75% on retries and multi-turn tool loops. Caches are model-scoped, so escalation costs one cold write (~$0.03 on Opus); mixing models per agent is free as long as prefix order never varies.

## 6. Machinery the engine needs (build order)

| Milestone | Addition |
|---|---|
| **M0** | `dsx route <call_point> --family --persona --attempt` reading `config/policy.yaml` → `{agent variant, model, effort/budget, max_attempts}`; agent variants generated from one source per role (`developer.md`, `developer-opus.md`, …). `dsx cost add --call` appends to `.dsx/telemetry.jsonl`: `ts, project, stage, iteration, call_point, attempt, agent, model, effort|budget_tokens, input_tokens, cache_read, cache_write, output_tokens, cost_usd, latency_s, outcome {schema_ok, tests_passed/total, dod_delta, verdict}, escalated_from, policy_version`. |
| **M2** | `dsx dod --run` executes `[test]` items and produces `{dod_id, result, evidence_path, sha256}`; `tests/e2e/fixtures/<toy>/` (inventory, churn, fraud) with scripted answers per persona. |
| **M3** | `dsx mutate <build>` applies `tests/mutations/*.yaml` (target shifted −1, future rolling mean, entity baseline including the current row, random-split protocol, wrong translation sign) to a copy under `.dsx/mutations/<build>/`; the generated oracle tests must FAIL on every mutation; catch rate < 100% forces `iterate`. `dsx shadow <call_point> --model X` runs a call point twice and diffs structured fields into `.dsx/shadow/<cp>.jsonl`. `dsx` computes `beats_by > noise` for Experiment interpretations. |
| **M5** | `dsx route tune` proposes one-tier downgrades only when shadow disagreement = 0 across ≥ 9 runs and mutation catch rate = 100% (silent points), or first-try success ≥ 85% (loud points); writes a `policy.yaml` diff for human approval. Promotes a rung when escalations exceed 30%. |

## 7. Evaluation protocol for the policy itself

Arms on the e2e fixtures (3 toys × 3 personas × 5 seeds): A = all-Opus/high reference, B = this policy, C = B with effort one notch up. Per call point, non-inferiority with a 2 pp margin on the structured metric named in the table (slot precision/recall, `must_ask` set equality, task-list equality, protocol P fields, verdict agreement, business metric within tolerance, kappa on checklist items, pass@1 for builds, judge rubric for prose). Keep the cheapest rung that passes.
