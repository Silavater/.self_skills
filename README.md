# Self Skills

作者為了方便開發，以及為了避免被 AI 搞到中風寫的各種 SKILL，建議拿需要的去用就好。

This repository is a workspace for personal Codex skills. The canonical source
for authored skills lives under `skills/`; `.agents/skills/` is the installed or
synchronized copy used by local agents.

## How To Use This Repo

- Browse `skills/` to see the maintained source skills.
- Copy or install only the skills that match your workflow.
- Treat `.agents/skills/` as runtime/vendor material unless you are explicitly
  syncing or testing an installed skill.
- Keep small tasks lightweight. Not every edit needs a workflow skill.

## Current Skills

### `skills/dev/beacon-dev-workflow`

`beacon-dev-workflow` is a repository-local planning workflow skill for work
that needs durable state, slice discipline, verification, recovery, and
archive evidence.

It helps an agent:

- plan long-running work through `.beacon/PLAN.md`;
- design one project part before slicing implementation;
- split work into verifiable slices;
- promote exactly one slice into `.beacon/CURRENT.md`;
- run or report verification through UnitTestCore and manual QA notes;
- open recovery incidents when work becomes unsafe, invalid, blocked, or drifts
  outside scope;
- archive completed current work and part-level evidence.

Use it for:

- Beacon-related work;
- larger projects that span multiple sessions;
- work that needs handoff, recovery, or auditability;
- risky implementation where scope drift would be expensive.

Do not use it for:

- typo fixes;
- tiny documentation edits;
- obvious single-file changes;
- tasks where a short plan plus normal verification is enough.

In short: Beacon is useful when the work has state debt. If the task is small,
skip the ceremony.

### `skills/quality-of-life/pyut-zh-why`

`pyut-zh-why` is a tiny quality-of-life skill for Windows/PowerShell sessions
where files may contain Chinese or other CJK text.

It tells the agent to read file contents through Python instead of PowerShell
`Get-Content`, `cat`, or `type`, because those commands often produce mojibake
when the encoding path is wrong.

Use it when an agent is about to read, quote, summarize, or inspect text files
that may contain Chinese. It is intentionally small: no scripts, no framework,
just the default rule.

## Supporting Files

- `AgentRule.md`: general agent behavior rules for this workspace.
- `AGENTS.md`: workspace layout and editing rules for agents.
- `doc/`: durable human-facing docs, including `doc/SKILL_INDEX.md`.
- `.doc/`: local planning/design notes, ignored by git.
- `.beacon/`: Beacon runtime state for developing Beacon itself, ignored by git.

## Author Rant Board

Agents must not edit the content between these markers.

<!-- AGENT-DO-NOT-EDIT-START -->
感覺有機會未來這裡只有那個skill🐑
<!-- AGENT-DO-NOT-EDIT-END -->
