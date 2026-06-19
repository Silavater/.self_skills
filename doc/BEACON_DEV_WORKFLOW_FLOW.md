# Beacon Dev Workflow Flow

## Purpose

This document explains the Beacon development workflow as a Mermaid flowchart.
It is meant for humans reading the repository, not as executable Beacon state.

Mermaid is useful here because Beacon is a process with strict state transitions:
planning artifacts authorize design, design authorizes slices, exactly one slice
becomes executable through `CURRENT.md`, and failed work must move through an
incident before it continues.

## How To Read These Diagrams

Beacon is easier to read as a few small diagrams instead of one large workflow
map. The full process has four concerns:

- lifecycle: the happy path from plan to archive;
- routing: what the agent reads first when work starts or resumes;
- slice loop: what happens to one active slice;
- artifact authority: which file is allowed to decide what.

## Lifecycle Overview

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
    Todo --> Current[CURRENT.md<br/>one active slice]
    Current --> Work[Implement<br/>allowed scope only]
    Work --> Verify[UnitTestCore<br/>and manual QA]
    Verify --> Done[done/<br/>archive evidence]
    Verify -.->|failure| Incident[incidents/<br/>recovery record]
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
    Exists -->|Yes| ReadCurrent[Read CURRENT.md]

    Init --> Planning[Planning-only work]
    ReadCurrent --> Status{CURRENT status?}
    Status -->|planning-only| Planning
    Status -->|active| Active[Read active DESIGN.md and TODO.md]
    Status -->|recovering / blocked| Recovery[Read linked incident first]

    Planning --> PlanDocs[Update PLAN, DESIGN, or TODO]
    Active --> CurrentScope[Execute CURRENT scope only]
    Recovery --> RecoveryPath[Choose fix-forward, reslice, redesign, or blocked]

    classDef action fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#f8fafc
    classDef decision fill:#1f2937,stroke:#f59e0b,stroke-width:3px,color:#fef3c7
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2

    class Start,Rules,Init,ReadCurrent,Planning,Active,PlanDocs,CurrentScope action
    class Exists,Status decision
    class Recovery,RecoveryPath stop
```

## Single Slice Loop

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
    Ready[Ready slice in TODO.md] --> Promote[Promote exactly one slice]
    Promote --> Current[Write CURRENT.md]
    Current --> Execute[Implement allowed scope]
    Execute --> Verify[Run UnitTestCore / manual QA]
    Verify --> Pass{Pass?}

    Pass -->|Yes| Archive[Archive done CURRENT and evidence]
    Archive --> TodoDone[Mark slice done in TODO.md]
    TodoDone --> More{More slices in PART?}
    More -->|Yes| Ready
    More -->|No| PartDone[Archive PART TODO and report]

    Pass -->|No| Incident[Open or update incident]
    Incident --> Recovery{Recovery path?}
    Recovery -->|fix-forward| Current
    Recovery -->|reslice| Ready
    Recovery -->|redesign| Redesign[Update PART DESIGN.md]
    Recovery -->|blocked| Blocked([Remain blocked])
    Redesign --> Ready

    classDef action fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#f8fafc
    classDef authority fill:#0f172a,stroke:#60a5fa,stroke-width:3px,color:#f8fafc
    classDef decision fill:#1f2937,stroke:#f59e0b,stroke-width:3px,color:#fef3c7
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2
    classDef done fill:#10251a,stroke:#34d399,stroke-width:3px,color:#dcfce7

    class Ready,Promote,Current,TodoDone,Redesign authority
    class Execute,Verify action
    class Pass,More,Recovery decision
    class Incident,Blocked stop
    class Archive,PartDone done
```

## Artifact Authority

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
    Backlog[BACKLOG.md<br/>non-executable ideas] -.->|promote through process| Plan
    Plan[PLAN.md<br/>project map] --> Design[DESIGN.md<br/>part design authority]
    Design --> Todo[TODO.md<br/>all slices for one part]
    Todo --> Current[CURRENT.md<br/>only executable authority]
    Current --> Done[done/<br/>completed snapshots]
    Current --> Incidents[incidents/<br/>recovery evidence]

    classDef idea fill:#111827,stroke:#94a3b8,stroke-width:2px,color:#e5e7eb
    classDef authority fill:#0f172a,stroke:#60a5fa,stroke-width:3px,color:#f8fafc
    classDef evidence fill:#10251a,stroke:#34d399,stroke-width:3px,color:#dcfce7
    classDef stop fill:#2a1114,stroke:#f87171,stroke-width:3px,color:#fee2e2

    class Backlog idea
    class Plan,Design,Todo,Current authority
    class Done evidence
    class Incidents stop
```

## What The Diagrams Communicate

- `BACKLOG.md` is not execution permission. Work must be promoted through PLAN,
  DESIGN, TODO, and CURRENT.
- `CURRENT.md` is the only active executable authority.
- Only one slice may be active in one `.beacon/` state.
- Verification is part of the workflow, not an optional ending.
- Failed or unsafe work moves through `incidents/` before execution continues.
- Archive files keep completed evidence out of active planning files.

## Current Skill Representation

The current Beacon skill already contains a textual flow:

- `SKILL.md` has Read Order, Workflow, State Rules, Hard Stops, and Verification.
- `references/beacon-artifacts.md` describes artifact responsibilities.
- `references/slice-contract.md` describes slice sizing and promotion.
- `references/recovery-protocol.md` describes incident and recovery flow.
- `references/verification.md` describes UnitTestCore and manual QA reporting.

The skill package contains the agent-facing workflow structure reference at
`skills/dev/beacon-dev-workflow/references/workflow-structure.md`.
`SKILL.md` still keeps the compact textual workflow:
`Plan -> Design -> Slice -> Promote -> Execute -> Verify -> Recover -> Archive`.

## Expected Internal Skill Guideline

The internal development-flow structure guideline should stay in a reference
file instead of expanding `SKILL.md`:

```txt
skills/dev/beacon-dev-workflow/
  SKILL.md
  references/
    workflow-structure.md
```

`workflow-structure.md` should contain:

- the canonical flowchart;
- the rule that `CURRENT.md` is the only executable authority;
- the rule that multi-agent work should use separate worktrees or branches, not
  multiple active slices in one `.beacon/`;
- the required transition gates between PLAN, DESIGN, TODO, CURRENT,
  UnitTestCore, incidents, and done archives.

Keep `SKILL.md` as the short invocation and routing layer. Keep detailed flow
structure in the reference so the skill remains compact.
