---
name: OpenSpec Fast-Forward
description: Fast-forward a change by producing all planning artifacts at once without interactive confirmation
---

# OpenSpec Fast-Forward

Fast-forward: produce all planning artifacts (proposal, specs, design, tasks) at once. Use when you have a clear picture of what to build and want to skip the iterative artifact-by-artifact flow.

## Input

A change name (kebab-case) AND a description of what to build. Both are required for fast-forward mode.

## Steps

1. **Create the change**
   ```bash
   openspec new change "<name>"
   ```

2. **Get artifact instructions for all artifacts**
   ```bash
   openspec status --change "<name>" --json
   ```
   For each artifact in dependency order:
   ```bash
   openspec instructions <artifact-id> --change "<name>" --json
   ```

3. **Generate all artifacts in one pass**

   Create all artifacts without pausing for user confirmation between each:
   - `proposal.md` — Business justification based on user's description
   - `specs/` — Detailed requirements extracted from the description
   - `design.md` — Technical design based on specs and codebase analysis
   - `tasks.md` — Implementation steps based on design

4. **Show final status**
   ```bash
   openspec status --change "<name>"
   ```

## Output

```
Fast-forwarded: add-user-notifications
  All artifacts created in one pass:
  - proposal.md: [one-line summary]
  - specs/: [N capability specs]
  - design.md: [one-line summary of approach]
  - tasks.md: [N tasks across M phases]

Ready for implementation.
Review artifacts before proceeding: openspec/changes/add-user-notifications/
```

## Example

Input: `/opsx:ff add-real-time-sync "Add real-time data sync between mobile and web clients using WebSocket with offline queue"`

Output: All 4+ artifacts created immediately, ready for user review before GSD planning.

## Guardrails

- Always produce ALL artifacts — do not stop partway
- Make reasonable decisions to maintain momentum — do not ask questions in fast-forward mode
- Clearly indicate that artifacts should be reviewed before implementation begins
- If the description is too vague to produce meaningful specs, fall back to `/opsx:propose` flow instead
