# Context Management

Applies to all agents.

## Atomic Subtask Decomposition

Tech Lead must break every task into focused, atomic subtasks targeting a single concern.

## Summary-Based Reporting

Agents must report results as concise summaries:
- What was done (1-3 sentences)
- Files created or modified (list of paths)
- Decisions and rationale (bullet points)
- Open issues or blockers

Do not dump full file contents or raw command output.

## Context Injection Scoping

Include only files directly relevant to the current subtask. Do not inject entire directories.

## Context Limit Handling

When approaching context limits:
1. Checkpoint progress by writing summary
2. Complete current atomic unit
3. Continue in a new context window

## Handoff Documentation

For multi-session work, write a handoff note before ending:
- Current progress
- Open items
- Next steps
- File paths for context restoration

## Worklog

All work documented in `.worklog/yyyymm/task-name/phase-n-label/`:
- `references.md` — Sources consulted
- `findings.md` — Key discoveries
- `decisions.md` — Decisions with rationale and evidence chain
