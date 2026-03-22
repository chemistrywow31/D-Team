---
name: OpenSpec Archive Change
description: Archive a completed change and merge delta specs back to main specs directory
---

# OpenSpec Archive Change

Archive a completed change to `openspec/changes/archive/` and merge delta specs back to the main specs directory.

## Input

Optionally specify a change name. If omitted, prompt for selection from available changes.

## Steps

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use AskUserQuestion to let the user select.

2. **Pre-archive verification**

   Run `openspec status --change "<name>"` to check:
   - All required artifacts exist
   - Tasks are marked complete

   If incomplete, warn the user and ask whether to proceed anyway or fix first.

3. **Archive the change**
   ```bash
   openspec archive --change "<name>"
   ```
   This moves the change directory to `openspec/changes/archive/{date}-{name}/` and merges delta specs back to `openspec/specs/` (or equivalent main spec location).

4. **Verify archive**
   - Confirm the change no longer appears in active changes
   - Confirm archived files exist at the expected location

5. **Report completion**

## Output

```
Archived: add-user-notifications
  → openspec/changes/archive/2026-03-22-add-user-notifications/
  → Delta specs merged to openspec/specs/
  → Active changes remaining: [count]
```

## Example

Input: `/opsx:archive add-user-notifications`

Output: Change archived with date prefix, delta specs merged, confirmation of successful archive.

## Guardrails

- Warn if archiving a change with incomplete tasks
- Do NOT archive without user confirmation if verification report has CRITICAL issues
- Always verify the archive succeeded before reporting completion
