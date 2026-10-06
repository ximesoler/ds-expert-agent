# ds-expert-agent — Architecture diagrams (v0.4, four stages)

Render: GitHub, VS Code (Markdown Preview Mermaid), Obsidian, or `npx @mermaid-js/mermaid-cli -i docs/ARCHITECTURE-DIAGRAMS.md -o docs/img/x.md -e png`.

## 0. Name of the architecture

**A spec-driven, stage-gated agentic workflow over a blackboard, with an orchestrator–workers panel of experts and evaluator–optimizer loops, built on ports-and-adapters** (short: *stage-gated blackboard orchestrator*).

| Layer | Pattern | Here |
|---|---|---|
| Control flow | agentic workflow (predefined code paths with LLM steps) + Stage-Gate | 4 stages; `dsx` decides the next step; the LLM runs at call points; every stage exits through a gate that certifies evidence |
| Collaboration | orchestrator–workers (supervisor) | Claude Code main thread owns the conversation; 6 stateless expert agents |
| Quality | evaluator–optimizer | developer → reviewer tests; independent evaluator at the Experiment exit |
| Shared state | blackboard | specs, deliverable registry, decisions.log, snapshots on disk |
| Boundaries | ports and adapters + human-in-the-loop | infra port (local venv now), UI (Claude Code now), knowledge store |

## 1. System context

```mermaid
flowchart LR
    U[("User<br/>business · junior DS · senior DS")]
    CC["Claude Code main thread<br/>/ds:* commands · skills · AskUserQuestion"]
    DSX["dsx engine<br/>board · spec gen · validate · graph · impact<br/>dod · build finish · gate · snapshot"]
    PANEL["Expert agents<br/>analyst · data-expert · developer<br/>reviewer · evaluator · writer"]
    KB[("Knowledge<br/>cards · playbooks · checklists")]
    BB[("Workspace blackboard<br/>project.functional · stage specs<br/>deliverable registry · decisions.log · snapshots")]
    INFRA["Infra port<br/>LocalVenvAdapter + MLflow (MVP)<br/>Verdiflow / Labs (later)"]
    DATA[("User data<br/>CSV / Parquet / Excel")]
    U <-->|questions · gates| CC
    CC -->|every step| DSX
    CC -->|Agent tool, JSON back| PANEL
    PANEL --> BB
    PANEL -->|knowledge_search| KB
    DSX <--> BB
    DSX -->|compile playbooks · DoD| KB
    DSX -->|submit · wait · fetch| INFRA
    INFRA --> DATA
    INFRA -->|metrics · artifacts| BB
```

## 2. Components mapped to plugin primitives

```mermaid
flowchart TB
    subgraph CMD["commands/ (thin) and skills/ (procedures)"]
        C1["/ds:status · goto · history · impact → dsx"]
        S1["ds-dialogue (stage 1)"]
        S2["ds-spec · ds-plan · ds-build · ds-gate"]
        S3["ds-change · ds-sync · ds-revalidate"]
    end
    subgraph AG["agents/"]
        A1[analyst]; A2[data-expert]; A3[developer]; A4[reviewer]; A5[evaluator]; A6[writer]; A7["curator (offline)"]
    end
    subgraph ENG["dsx/"]
        E1["board · spec gen · schema select · wave next"]
        E2["validate · graph build · impact · pin · guard"]
        E3["expect / check-report · dod · build finish · gate approve"]
        E4["dispatch / wait / fetch (infra port)"]
        E5["knowledge build | search"]
        E6["mcp__ds__ tools: profile · run_python · save_figure · mlflow_log"]
    end
    subgraph HK["hooks/"]
        H1["SessionStart → board --context"]; H2["PreCompact → digest"]; H3["PreToolUse → guard / deny-list"]; H4["SubagentStop → check-report"]
    end
    subgraph WS["workspace"]
        W1["specs/project.functional.md"]; W2["specs/0n-stage/{functional,technical,plans,builds,manifest}"]; W3["specs/deliverables/ registry"]; W4["src · features/v<k> · experiments · deploy · monitoring"]; W5["decisions.log · project.yaml · .dsx/snapshots"]
    end
    CMD --> ENG
    S1 & S2 & S3 -->|Agent tool| AG
    AG -->|JSON| E3
    AG --> E6
    ENG --> WS
    HK --> ENG
    E5 -.playbooks.-> AG
```

## 3. Lifecycle: four stages, three human gates

```mermaid
stateDiagram-v2
    [*] --> S1
    state "1 Business (dialogue)" as S1 {
        [*] --> Intake
        Intake --> SlotList : persona · family · domain · schema
        SlotList --> Waves : classify gaps
        Waves --> Waves : must_ask wave (≤4)
        Waves --> Gate1 : no must_ask open
    }
    Gate1 --> S2 : user approves (manifest)
    state "2 Data (cycle)" as S2 {
        [*] --> F2 : functional (generated)
        F2 --> T2 : technical
        T2 --> P2 : plan 1 profile + target + pending_data wave
        P2 --> B2 : build (reviewer tests)
        B2 --> P2 : iterate (EDA analyses)
        B2 --> T2 : change_method
        B2 --> Gate2 : DoD met
    }
    Gate2 --> S3 : "data supports it"
    state "3 Experiment (cycle)" as S3 {
        [*] --> F3
        F3 --> T3 : technical defines protocol P
        T3 --> P3 : plan n = (features vk, method, P)
        P3 --> Cert : point-in-time test
        Cert --> B3 : certified
        Cert --> P3 : fails (fix features)
        B3 --> P3 : iterate / next rung
        B3 --> Eval : threshold or budget
        Eval --> Gate3 : evaluator business metric
        Eval --> P3 : conditional
    }
    Gate3 --> S4 : business owner: ship
    state "4 Ship (cycle)" as S4 {
        [*] --> F4
        F4 --> T4
        T4 --> P4 : plan 1 packaging · plan 2 monitoring + handoff
        P4 --> B4 : tests
        B4 --> P4 : iterate
        B4 --> Gate4 : DoD met (model card before prod)
    }
    Gate4 --> [*] : handoff accepted
    S2 --> S1 : change_purpose (§2)
    S3 --> S1 : change_purpose
    S3 --> S2 : back_to 2 (inventory doubt)
    S4 --> S3 : back_to 3 (not servable)
```

## 4. One Experiment iteration (sequence)

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant CC as Claude Code (skill ds-build)
    participant X as dsx
    participant D as developer
    participant R as reviewer
    participant I as infra
    participant E as evaluator
    CC->>X: board --json (next = build 3-3)
    CC->>X: expect builds/3-3 · features/v3
    CC->>D: plan 3-3 path + technical + cards
    D-->>CC: JSON {files, how_to_run}
    CC->>X: dod --pre (runs tests/test_point_in_time.py)
    X-->>CC: features/v3 pit_certified = true
    CC->>R: checklist for stage 3
    R->>I: smoke run on sample
    R-->>CC: JSON [{dod_id, result, evidence_path}]
    CC->>X: dispatch (refuses if uncertified) → wait → fetch
    I-->>X: MLflow run, metrics
    CC->>D: interpret (metrics path)
    D-->>CC: JSON {findings, verdict: iterate|done}
    CC->>X: build finish (hashes, DoD, render builds/3-3.md, apply verdict, board)
    alt verdict done and threshold met
        CC->>E: translate from logged predictions
        E-->>CC: JSON {business_metric, go/no-go}
        CC->>X: dod → manifest
        CC->>U: Gate 3: ship / iterate / stop
        U-->>CC: decision
        CC->>X: gate approve 3 → snapshot → cursor 4
    end
```

## 5. Call points: code vs LLM vs human

```mermaid
flowchart LR
    classDef code fill:#eef,stroke:#88a
    classDef llm fill:#fee,stroke:#a88
    classDef human fill:#efe,stroke:#8a8
    N["dsx board → next"]:::code --> FG["functional (spec gen)"]:::code --> T["technical (expert)"]:::llm --> Q{"must_ask?"}:::code
    Q -->|yes| W["question wave"]:::human --> P
    Q -->|no| P["plan (planner)"]:::llm --> B["build code (developer)"]:::llm --> R["reviewer tests + JSON"]:::llm --> X["dispatch · wait · fetch"]:::code --> INT["interpret (expert)"]:::llm --> F["build finish · DoD"]:::code
    F -->|unmet| P
    F -->|met| G["gate: manifest + decision"]:::human --> N
    L1["blue = deterministic dsx"]:::code
    L2["red = LLM call point"]:::llm
    L3["green = human"]:::human
```

## 6. MVP scope

| Component | MVP | Later |
|---|---|---|
| Surface | Claude Code plugin | Agent SDK app, web |
| Infra port | LocalVenvAdapter + MLflow file store | Verdiflow / Labs |
| Staleness | stage-level with cost; section-level for the project functional | section-level everywhere, LLM revalidation, branches |
| Agents | 6 + curator | domain packs add cards, not agents |
| Deployment | FastAPI + Docker + MLflow registry, tests locally | registry push, cloud run |
