---
name: OpenSpec Propose
description: Propose a new change with all artifacts (proposal, specs, design, tasks) generated in sequence
---

# OpenSpec Propose

Propose a new change — create the change scaffold and generate all artifacts in one step.

Artifacts produced:
- `proposal.md` (what & why)
- `specs/` (detailed requirements per capability)
- `design.md` (how — technical design)
- `tasks.md` (implementation steps)

When ready to implement: medium+ features → `/gsd:plan-phase N`; small features → implement directly.

## Input

The user's request must include a change name (kebab-case) OR a description of what they want to build.

## Steps

1. **If no clear input provided, ask what they want to build**

   Use the AskUserQuestion tool to ask:
   > "What change do you want to work on? Describe what you want to build or fix."

   From their description, derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).

2. **Create the change directory**
   ```bash
   openspec new change "<name>"
   ```
   This creates a scaffolded change at `openspec/changes/<name>/`.

3. **Get the artifact build order**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to get artifact list and dependency order.

4. **Create artifacts in sequence until ready for implementation**

   Loop through artifacts in dependency order:

   a. For each artifact that is ready (dependencies satisfied):
      - Get instructions: `openspec instructions <artifact-id> --change "<name>" --json`
      - Read any completed dependency files for context
      - Create the artifact file using the template as structure
      - Apply context and rules as constraints — do NOT copy them into the file

   b. Continue until all required artifacts are complete

   c. If an artifact requires user input (unclear context):
      - Use AskUserQuestion to clarify, then continue

5. **Show final status**
   ```bash
   openspec status --change "<name>"
   ```

## Output

After completing all artifacts, summarize:
- Change name and location
- List of artifacts created with brief descriptions
- Readiness: "All artifacts created. Ready for implementation."
- Next step: "For medium+ features, run `/gsd:plan-phase N`. For small features, implement directly."

## Example

Input: `/opsx:propose "add-user-notifications"`

Output:
```
Change created: openspec/changes/add-user-notifications/
  - proposal.md: Push notification system for user engagement
  - specs/notification-delivery/spec.md: Delivery channels and routing rules
  - specs/notification-preferences/spec.md: User preference management
  - design.md: WebSocket + queue architecture with fallback to email
  - tasks.md: 8 tasks across 3 phases

All artifacts created. Ready for implementation.
For medium+ features, run /gsd:plan-phase 1.
```

## Guardrails

- Create ALL artifacts needed for implementation
- Always read dependency artifacts before creating a new one
- If context is critically unclear, ask the user — but prefer making reasonable decisions to keep momentum
- If a change with that name already exists, ask if the user wants to continue it or create a new one
- Verify each artifact file exists after writing before proceeding to next
