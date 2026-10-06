# ds-expert-agent — Architecture (v0.5)

**Status:** design, v0.5 (2026-10-06). v0.4 structure (four stages) + verb-first grammar, per-stage gap detector, panel at every step (three three-way design reviews).
**Form:** a Claude Code plugin (`/ds:*` commands) backed by a deterministic `dsx` CLI; an Agent SDK app can later wrap the same engine, skills and agents.
**Architecture style:** spec-driven, stage-gated agentic workflow over a blackboard, with an orchestrator–workers panel of experts, evaluator–optimizer loops, and ports-and-adapters (short: *stage-gated blackboard orchestrator*).
**Normative companions:** `specs/SPECS.md` v2.1 (what goes in each spec, per stage) · `docs/UX-AND-EXECUTION.md` v0.3 (verb grammar, board, change handling) · `docs/EXECUTION-WALKTHROUGH.md` v1.1 (file-by-file trace of a run) · `docs/MODEL-POLICY.md` + `config/policy.yaml` (model/effort routing) · `KNOWLEDGE.md` + `knowledge/` (expert knowledge cards) · `docs/ARCHITECTURE-DIAGRAMS.md`.

---

## 1. What we are building

An interactive data-science expert that takes a diffuse request ("estimate inventory for product X for the next months, I have two years of sales") from a business user, a junior or a senior data scientist, and drives it to a deployable ML solution through the data-science lifecycle, **extracting the user's knowledge wherever the data or the request leaves a gap**.

Reference point: AutoKaggle (reader → planner → developer → reviewer, Kaggle phases, validated tool library, unit-test loop). We keep phase decomposition, validated tools and the developer/reviewer loop, and fix its weakness: a one-shot pipeline with no user in the loop and a Kaggle-shaped problem definition. Here the problem definition is produced by a conversation, and every stage exits through a human gate that certifies evidence.

### Personas (detected at intake)

| Persona | Can answer | Agent behaviour |
|---|---|---|
| Business user | the decision, costs of error, calendar/events, what "good" means | plain language, more guided questions, technical defaults chosen and explained |
| Junior DS | the above + data semantics, some modeling preferences | explains why, offers 2–3 options with a recommendation |
| Senior DS | everything, wants control | terse technical checkpoints, may edit specs directly, confirms technical specs |

Persona changes wording and question-wave limits. **It never changes gates.**

## 2. Design principles

1. **Orchestrator owns the conversation.** Only the main thread can ask the user (the Agent SDK/Claude Code constraint: no `AskUserQuestion` inside subagents). Experts return structured results; the orchestrator asks, confirms, or defaults.
2. **Specs are the contracts; reports are derived.** Every stage runs functional (WHAT) → technical (HOW) → plan (EXECUTION) → build (THE WORK). Stage 1 is the exception: a dialogue that produces the project functional spec.
3. **Gaps are first-class.** Requirements schemas per problem family plus domain questions from knowledge cards; every unfilled slot is a gap classified `must_ask`, `infer_from_data` (→ `pending_data`), or `safe_default`.
4. **Evidence, not consent.** Gates write a manifest: every definition-of-done item with its result, how it was enforced (test, reviewer, user), evidence path and hash; pinned inputs; card-set hash; approver. Reviewer items map to executable tests.
5. **Machine boundaries where leaks hide.** A feature set is point-in-time certified by a test before any model may train on it.
6. **State on disk, engine in code.** Specs, deliverables, decisions and snapshots live in the workspace; hashing, dependency edges, staleness, DoD evaluation, verdict application and cost accounting are deterministic `dsx` code, never prompt text.
7. **Expertise is data.** Domain knowledge lives in versioned cards and reaches agents through generated playbooks, retrieval, gap-detector questions and reviewer checklists.
8. **Bounded cost.** Budgets nest (plan ≤ stage ≤ project); expensive reruns need confirmation with a cost figure.

## 3. The lifecycle: four stages, three human gates

```
 1 BUSINESS ──gate 1──► 2 DATA ──gate 2──► 3 EXPERIMENT ──gate 3──► 4 SHIP ──gate 4──► handoff
 dialogue only          cycle               cycle (+ evaluation exit)   cycle
 "is this the problem?" "does the data      "ship / iterate / stop"     "accept handoff"
  user                   support it?"        business owner              user + ops owner
                         user answers must_ask; DoD rows are machine-checked
```

| Stage | Covers (classic lifecycle) | Deliverables (registry nodes) | Who writes the technical | Back-edges |
|---|---|---|---|---|
| **1 Business** | business understanding, **no data** | `project.functional.md` with provenance (§6 = data expected, in prose) | — (dialogue skill; analyst proposes questions) | reopened only by a §2 change |
| **2 Data** | data understanding, EDA, feature understanding | `data_card`, `target`, `eda_report`, `feature_inventory`, `split_recommendation` | `data-expert` | `change_purpose` (§2/§3) |
| **3 Experiment** | feature engineering, modeling, evaluation | `protocol_P`, `features/v<k>` (certified), `results_table`, `champion`, `business_metric` | `developer` (drafts), `evaluator` (exit) | `back_to:2`, `change_method`, `escalate` |
| **4 Ship** | deployment, monitoring, handoff | `deploy/`, `monitoring_config`, `model_card`, README, runbook | `developer` | `back_to:3` |

Why these boundaries (from the review): the old data/EDA gate was a manifest row nobody could judge; "add lag 13" was a cross-stage round trip and is now the next experiment plan; the champion must not be approved technically and then again in money; monitoring in v1 is config plus one script. The old feature-engineering gate survives as a certification node, not a click.

The cycle inside stages 2–4 and the exact content of every artifact are in `specs/SPECS.md`; each stage's functional is completed by its own gap detector (panel proposes, user answers), the technical is built by the panel, plan and build use only the experts named by the technical.

## 4. Plugin architecture

```
┌──────────────────────────────── Claude Code (main thread = orchestrator) ───────────────────────────────┐
│  /ds:new  /ds:next  /ds:spec  /ds:plan  /ds:build  /ds:gate  /ds:change  /ds:sync     (skills = procedures)│
│  /ds:status  /ds:goto  /ds:history  /ds:impact                                         (thin over dsx)     │
│  AskUserQuestion ◄── only here                                                                               │
└───────┬───────────────────────────────┬──────────────────────────────┬──────────────────────────────────────┘
        │ Agent tool (JSON results)     │ dsx CLI (deterministic)      │ hooks
┌───────▼────────────┐        ┌─────────▼──────────────┐     ┌─────────▼─────────────────────────────┐
│ agents/ (6 + 1)    │        │ dsx                     │     │ SessionStart → dsx board --context     │
│ analyst            │        │ init · board · spec gen │     │ PreCompact   → dsx digest              │
│ data-expert        │        │ validate · graph build  │     │ PreToolUse Write|Edit → dsx guard      │
│ developer          │        │ impact · pin · dod      │     │ PreToolUse Bash → deny-list            │
│ reviewer           │        │ expect · check-report   │     │ SubagentStop → dsx check-report        │
│ evaluator          │        │ dispatch · wait · fetch │     └────────────────────────────────────────┘
│ writer             │        │ build finish · gate     │
│ curator (offline)  │        │ snapshot · cost · profile│
└────────┬───────────┘        │ knowledge build|search  │
         │ read/write          └─────────┬──────────────┘
┌────────▼────────────────────────────────▼──────────────────────────────────────────────────────────────┐
│ WORKSPACE (blackboard)  specs/ · specs/deliverables/ · src/ · features/ · experiments/ · deploy/        │
│ decisions.log · project.yaml · .dsx/snapshots/ · mlruns/ · knowledge playbooks (compiled)              │
└──────────────────────────────────────────────────────▲──────────────────────────────────────────────────┘
                                                       │ infra port: submit · wait · fetch
                                     LocalVenvAdapter (MVP) · VerdiflowAdapter (later)
```

### 4.1 Agents (`agents/*.md`, fresh context per call, JSON results)

Model and effort are **routed per call point** by `dsx route` from `config/policy.yaml` (see `docs/MODEL-POLICY.md`), not fixed per agent: one source file per role, variants generated per rung. Principle: Haiku for parametrizable work, Sonnet for most building, Opus only at the *oracle* call points (target rule, protocol P, point-in-time/backtest/translation code, Experiment interpretation, evaluator) until mutation tests make them loud. Effort never exceeds `high` by default.

| Agent | Tools | Role | v1 rung (default → escalation) |
|---|---|---|---|
| `analyst` | Read, Glob, Grep, mcp__ds__* | persona/family/domain detection; slot pre-fill; `must_ask` drafting | haiku 2k → sonnet/low–medium |
| `data-expert` | Read, Bash(venv), run_python, save_figure | Data technical spec (oracle: target rule, PIT rule); analyses; inventory; interpretation | technical opus/medium → opus/high; interpretation sonnet/medium |
| `developer` | Read, Write, Edit, Bash, run_python, mlflow_log | Experiment technical (oracle: protocol P) and code; Ship code | technical S3 opus/high (senior: medium); oracle code opus/medium; other code sonnet/medium–high → opus; parametrized builds haiku 4k |
| `reviewer` | Read, Grep, Bash(run only) | advisory checklist items only (`[test]` items are run by `dsx dod --run`) | haiku 1k → sonnet/medium on card flag |
| `evaluator` | Read, run_python | business translation from logged predictions; error analysis; go/no-go (never the developer) | opus/medium → opus/high on user revise |
| `writer` | Read, Write | gate summaries, reports, model card, README | haiku 1k (sonnet/low for business persona at gates) |
| `curator` | Read, Write, WebFetch | drafts knowledge cards (`/ds:learn` only) | sonnet/medium |

Main thread (orchestrator session): Sonnet / medium. Per-stage expertise arrives through compiled playbooks and retrieved cards, not through more agent files. Experts never ask the user. **The panel acts at every step:** at `/ds:spec --functional` the stage's experts propose the questions the gap detector asks (`dsx gaps propose`); at `/ds:spec --technical` contributors return section contributions and a lead drafts; at `/ds:plan` only the owners named in the technical's `§Method` are consulted (cap 2); at `/ds:build` each task runs by its `owner`. Rosters: `specs/SPECS.md` §5.

### 4.2 Skills (procedures)

`ds-dialogue` (stage 1, no data), `ds-spec` (functional branch = skeleton + gap detector with panel proposals + waves; technical branch = contributors + lead), `ds-plan` (consult owners + planner + card), `ds-build` (task DAG by owner, reviewer, `dod --run`, finish), `ds-gate`, `ds-revalidate`, `ds-change`, `ds-sync`. Every SKILL.md starts with `dsx board --context` and reads only the files it lists. All commands go through one parser (`dsx parse`); `/ds:next` emits the same parse JSON from the board, prints the verb it runs, chains free steps and stops at paid or asking steps. Grammar: `docs/UX-AND-EXECUTION.md` §2.

### 4.3 `dsx` (deterministic engine)

`init` · `parse` · `board [--json|--context]` · `spec gen` · `schema select [--stage]` · `gaps propose | merge` · `wave next [--stage]` · `plan consult` · `spec commit` · `build start | abandon` · `validate` · `graph build` · `impact` · `pin` · `expect` / `check-report` · `guard` · `dispatch` / `wait` / `fetch` · `dod [--run]` (executes `[test]` items itself) · `route` · `mutate` · `shadow` · `build finish` (one transaction: reviewer JSON → evidence hashes → verdict → board) · `gate approve` (manifest + snapshot + cursor) · `cost add` · `snapshot` · `profile [--glance]` · `knowledge build | search` · `digest`.

### 4.4 Custom tools (`mcp__ds__`, in-process MCP server)

`profile_dataset`, `run_python` (project venv, timeout, cwd pinned), `save_figure` / `list_artifacts`, `read_state` / `log_decision`, `mlflow_log`, `knowledge_search(stage, family, domain, query)`.

### 4.5 Hooks and permissions

`SessionStart` prints the board context (≤ ~10k tokens: board, project functional, cursor stage functional + technical, current plan, last build header). `PreCompact` writes a digest. `PreToolUse Write|Edit` → `dsx guard`: `builds/` and finished `experiments/<n>/` append-only, `data/raw/` read-only, `project.yaml` only via dsx. `PreToolUse Bash` deny-list (destructive commands, installs outside the venv, network before stage 4). `SubagentStop` → `dsx check-report` validates the file the skill registered with `dsx expect`. Permission mode `acceptEdits` inside the workspace; deployment actions always ask. Caps: `max_budget_usd`, concurrency 3, spawn depth 1.

### 4.6 State and change handling (summary; full in `docs/UX-AND-EXECUTION.md`)

- Deliverable registry (`specs/deliverables/`) with version + content hash; dependency edges owned by dsx (static table + registry); headers carry pins only.
- Section-level hashing for `project.functional.md` (static section → stage table including §7 assumptions and §8 gaps); stage-level staleness elsewhere.
- A change marks dependents stale, never deletes; the board shows what is stale, why, and the cost to rerun from past builds; expensive reruns need confirmation; users may pin `accepted_stale` with a note.
- Snapshots in `.dsx/snapshots/` at every gate and verdict; git commit when a repo exists. Branches and LLM triage are v2.

## 5. Knowledge layer

Cards in `knowledge/cards/` (one claim per file, YAML header: stage, family, domain, type, applies_when, enforce, confidence, sources). `dsx knowledge build` compiles playbooks per agent and stage, gap-detector question schemas, and DoD checklists; `knowledge_search` retrieves at call points. Cards mapped to stages: business → 1; data, eda, feature_understanding → 2; feature_engineering, modeling, evaluation → 3; deployment, monitoring → 4. `applied_cards` are recorded as `{id, hash}`. Growth loop and regression tests: `KNOWLEDGE.md`.

## 6. Workspace layout (one directory per project)

```
projects/<slug>/
├── project.yaml                  # cursor, per-stage status, budget (written only by dsx)
├── decisions.log                 # append-only: waves, answers, verdicts, gate approvals, pins
├── specs/
│   ├── project.functional.md     # stage 1 artifact, section-hashed, provenance block
│   ├── deliverables/             # registry: data_card.md, feature_inventory.yaml, protocol.md, champion.yaml, …
│   ├── 02-data/      {functional.md, technical.md, plans/, builds/, manifest.yaml}
│   ├── 03-experiment/{…}
│   └── 04-ship/      {…}
├── data/raw/ (read-only) · data/processed/
├── src/{data,experiment}/ · features/v<k>/ · experiments/<n>/ · deploy/ · monitoring/
├── reports/                      # derived at gates
├── mlruns/ · .venv/ · .dsx/{snapshots,gaps,locks,telemetry.jsonl}
└── README.md · MODEL_CARD.md · RUNBOOK.md (stage 4)
```

## 7. Repository layout (the plugin)

```
ds-expert-agent/
├── .claude-plugin/plugin.json
├── commands/        ds-spec.md ds-plan.md ds-build.md ds-gate.md ds-next.md ds-goto.md ds-status.md ds-history.md ds-impact.md ds-change.md ds-sync.md ds-learn.md (ds-new.md alias)
│                    # argument-hint: spec [--functional|--technical] [--business|--data|--modeling|--ship] ["text"] · plan [--stage] ["hypothesis"] · build [--stage] [--task n] [--resume]
├── skills/          ds-dialogue/ ds-spec/ ds-plan/ ds-build/ ds-gate/ ds-revalidate/ ds-change/ ds-sync/   (+ generated playbooks)
├── agents/          analyst.md data-expert.md developer.md reviewer.md evaluator.md writer.md curator.md
├── hooks/           hooks.json + scripts calling dsx
├── dsx/             Python package + CLI (engine, infra adapters, mcp__ds__ server, knowledge compiler)
├── schemas/         forecasting.yaml tabular.yaml (requirements slots) · card.schema.yaml
├── knowledge/       cards/ conflicts.yaml
├── templates/       specs/ deliverables/ deploy/
├── specs/SPECS.md · docs/ · KNOWLEDGE.md · ARCHITECTURE.md
└── tests/           unit/ (dsx) · e2e/ (toy projects with scripted answers) · knowledge/ (card-driven behaviour)
```

## 8. Tech stack

Python 3.12, `uv`; Claude Code plugin APIs (commands, skills, agents, hooks); `claude-agent-sdk` later for the second surface; pandas/pyarrow (polars optional); scikit-learn, lightgbm, statsforecast (optional prophet); mlflow, fastapi, uvicorn, pydantic, docker; typer + rich for `dsx`; pytest. Models and effort routed per call point (`docs/MODEL-POLICY.md`, `config/policy.yaml`).

## 9. Roadmap (plugin on local infra)

| Milestone | Deliverable | Proves |
|---|---|---|
| **M0 Engine** | `dsx` core: init, board (--json/--context), spec gen, validate, graph build, pin, guard, expect/check-report, dod, build finish, gate approve, snapshot, cost (+ per-call telemetry), `route` from `config/policy.yaml`, LocalVenvAdapter (submit/wait/fetch); hooks; plugin skeleton | the board and the transactions work with no ML |
| **M1 Dialogue** | `ds-dialogue` skill, requirements schemas (forecasting, tabular), gap classifier, wave loop, Gate 1 manifest | the inventory and fraud examples yield a complete project functional with `pending_data` gaps |
| **M2 Data** | `data-expert`, `reviewer`, `dsx dod --run`, e2e fixtures with scripted answers, profiling/target/EDA tools, inventory, split recommendation, Gate 2 | "does the data support it?" is answered with evidence |
| **M3 Experiment** | `developer`, `evaluator`, protocol P, certification test, backtest engine, ladder, MLflow, business translation, `dsx mutate` + `dsx shadow`, engine-computed noise rule, Gate 3 | a champion with a trustworthy validation and a business number |
| **M4 Ship** | packaging templates, tests, monitoring job, handoff docs, Gate 4 | a runnable artifact and an operable handoff |
| **M5 Hardening** | e2e toy projects (inventory forecasting, churn, fraud) × personas; knowledge regression tests; policy evaluation arms and `dsx route tune`; cost caps; `/ds:change` + `/ds:sync` flows | repeatable, bounded-cost runs and safe changes |

v2: section-level impact beyond the project functional, LLM revalidation triage, `/ds:branch` / `compare`, Verdiflow adapter, Agent SDK surface, web UI.

## 10. Risks

| Risk | Mitigation |
|---|---|
| Too many or too few questions | slot list with `why_it_matters`, persona wave limits, `pending_data` deferral, e2e fixtures counting questions per persona |
| Leaky validation | certification test before training; reviewer items are executable tests; benchmarks on identical folds |
| Board and files drift | only dsx writes headers, verdicts, manifests; hooks block manual edits of builds and state |
| Context loss on restart/compaction | SessionStart context, PreCompact digest, everything on disk |
| Runaway cost | nested budgets, reruns need confirmation with cost figures, caps on subagents |
| Unsafe actions | guard and deny-list hooks; deployment actions always ask |

## 11. Open questions

1. Inventory cost model and generic cost matrix are the v1 business translations; which others (churn revenue-at-risk, fraud value caught) ship in M3?
2. MLflow local file store in v1; shared server later.
3. BigQuery/SQL data sources (gcloud present on this machine) are a v2 Data adapter.
