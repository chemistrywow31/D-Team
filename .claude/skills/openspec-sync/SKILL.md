---
name: OpenSpec Sync Specs
description: Manually sync delta specs from an active change back to the main specs directory mid-development
---

# OpenSpec Sync Specs

Manually sync delta specs from an active change back to the main specs directory without archiving the change. Use this mid-development when you want the main specs to reflect work-in-progress changes.

## Input

Optionally specify a change name. If omitted, prompt for selection from available changes.

## Steps

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use AskUserQuestion to let the user select.

2. **Sync delta specs**
   ```bash
   openspec sync --change "<name>"
   ```
   This copies delta specs from `openspec/changes/<name>/specs/` to the main `openspec/specs/` directory, merging with existing specs.

3. **Report what was synced**
   - List files that were updated or created in main specs
   - Note any conflicts that were resolved

## Output

```
Synced specs from: add-user-notifications
  → Updated: openspec/specs/notifications/spec.md
  → Created: openspec/specs/notification-preferences/spec.md
  → Change remains active (not archived)
```

## Example

Input: `/opsx:sync add-user-notifications`

Output: Delta specs merged to main specs directory, change remains active for continued work.

## Guardrails

- Do NOT archive the change — sync only copies specs
- Warn if delta specs conflict with existing main specs
- Always report which files were affected
