# Workflow Structure

Use this reference when the agent needs the Beacon process shape: how work
moves from project plan to one executable slice, how completed work is handed to
the next slice, and which artifact is allowed to authorize execution.

## Core Shape

```mermaid
---
config:
  theme: base
  layout: dagre
  themeVariables:
    primaryColor: "#111827"
    primaryBorderColor: "#60a5fa"
    primaryTextColor: "#f8fafc"
    secondaryColor: "#0f172a"
    tertiaryColor: "#1f2937"
    lineColor: "#94a3b8"
    edgeLabelBackground: "#0b1020"
    clusterBkg: "#050816"
    clusterBorder: "#334155"
---
flowchart LR
    Plan[PLAN.md<br/>project map] --> Design[PART DESIGN.md<br/>design authority]
    Design --> Todo[PART TODO.md<br/>slice map]
    Todo --> Current[CURRENT.md<br/>one executable slice]
    Current --> Work[Implement<br/>allowed scope only]
    Work --> Verify[UnitTestCore<br/>and manual QA]
    Verify --> Done[done/<br/>completed evidence]
    Verify -.->|failure or drift| Incident[incidents/<br/>recovery record]
    Incident -.->|fix-forward| Current
    Incident -.->|reslice| Todo
    Incident -.->|redesign| Design

    classDef action fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#f8fafc
    classDef authority fill:#0f172a,stroke:#60a5fa,stroke-width:3px,color:#f8fafc
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2
    classDef done fill:#10251a,stroke:#34d399,stroke-width:3px,color:#dcfce7

    class Plan,Design,Todo,Current authority
    class Work,Verify action
    class Incident stop
    class Done done
```

## Resume Routing

```mermaid
---
config:
  theme: base
  layout: dagre
  themeVariables:
    primaryColor: "#111827"
    primaryBorderColor: "#60a5fa"
    primaryTextColor: "#f8fafc"
    secondaryColor: "#0f172a"
    tertiaryColor: "#1f2937"
    lineColor: "#94a3b8"
    edgeLabelBackground: "#0b1020"
    clusterBkg: "#050816"
    clusterBorder: "#334155"
---
flowchart TD
    Start([Beacon work starts]) --> Rules[Read repo rules and Beacon skill]
    Rules --> Exists{Does .beacon exist?}
    Exists -->|No| Init[Initialize planning-only state]
    Exists -->|Yes| Current[Read CURRENT.md]

    Init --> Planning[Planning-only work]
    Current --> Status{CURRENT status?}
    Status -->|planning-only| Planning
    Status -->|active| Active[Read active DESIGN.md and TODO.md]
    Status -->|recovering / blocked| Recovery[Read linked incident first]

    Planning --> PlanDocs[Update PLAN, DESIGN, or TODO]
    Active --> Execute[Execute CURRENT scope only]
    Recovery --> RecoveryPath[Choose fix-forward, reslice, redesign, or blocked]

    classDef action fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#f8fafc
    classDef decision fill:#1f2937,stroke:#f59e0b,stroke-width:3px,color:#fef3c7
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2

    class Start,Rules,Init,Current,Planning,Active,PlanDocs,Execute action
    class Exists,Status decision
    class Recovery,RecoveryPath stop
```

## Single Slice Closure

```mermaid
---
config:
  theme: base
  layout: dagre
  themeVariables:
    primaryColor: "#111827"
    primaryBorderColor: "#60a5fa"
    primaryTextColor: "#f8fafc"
    secondaryColor: "#0f172a"
    tertiaryColor: "#1f2937"
    lineColor: "#94a3b8"
    edgeLabelBackground: "#0b1020"
    clusterBkg: "#050816"
    clusterBorder: "#334155"
---
flowchart TD
    Current[CURRENT.md active] --> Verify[Run verification]
    Verify --> Pass{Pass?}
    Pass -->|Yes| Archive[Archive done CURRENT and evidence]
    Archive --> TodoDone[Mark slice done in TODO.md]
    TodoDone --> More{More slices in PART?}
    More -->|Yes| Next[Promote next ready slice to CURRENT.md]
    More -->|No| PartDone[Archive PART TODO and verification report]

    Pass -->|No| Incident[Open or update incident]
    Incident --> Recovery{Recovery path?}
    Recovery -->|fix-forward| Current
    Recovery -->|reslice| Next
    Recovery -->|redesign| Design[Update PART DESIGN.md]
    Recovery -->|blocked| Blocked([CURRENT blocked])

    classDef authority fill:#0f172a,stroke:#60a5fa,stroke-width:3px,color:#f8fafc
    classDef action fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#f8fafc
    classDef decision fill:#1f2937,stroke:#f59e0b,stroke-width:3px,color:#fef3c7
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2
    classDef done fill:#10251a,stroke:#34d399,stroke-width:3px,color:#dcfce7

    class Current,TodoDone,Next,Design authority
    class Verify action
    class Pass,More,Recovery decision
    class Incident,Blocked stop
    class Archive,PartDone done
```

## Agent Rules

- `CURRENT.md` is the only executable authority.
- `PLAN.md`, `DESIGN.md`, `TODO.md`, and `BACKLOG.md` can prepare work but do
  not authorize implementation by themselves.
- Missing Beacon artifacts are not user questions by default. First repair the
  state to planning-only or to the nearest non-executable artifact state. Ask
  only when the missing content requires product or design intent that cannot be
  inferred.
- Cross-slice handoff does not require `HANDOFF.md` in V1. Use `TODO.md` status,
  archived `done/` evidence, and the next `CURRENT.md`.
- Keep active files short. Put history, command output, and completion evidence
  in `done/` or `incidents/`.
- When a PART is done, keep `parts/part-XXX/TODO.md` as a compact final index
  and archive the full completed TODO snapshot under `done/part-XXX/`.
- Multi-agent work should use separate worktrees or branches. Do not represent
  multiple active slices in one `.beacon/CURRENT.md`.
