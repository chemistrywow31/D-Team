---
name: Browser Verify
description: Browser-based spec verification using Playwright MCP to test real user flows against OpenSpec requirements
---

# Browser Verify

Verify that the running application matches OpenSpec specs by operating a real browser via Playwright MCP. Read the spec, open the browser, execute every scenario, screenshot every result.

## Step 0: Environment Readiness Check

Before any browser operation, verify Playwright MCP is available. Run these checks in order:

### 0a. Check MCP tools are loaded

Attempt to call `mcp__playwright__browser_navigate`. If the tool is not recognized (tool not found error), Playwright MCP is not active.

### 0b. If MCP tools are NOT available — auto-setup

1. **Check `.mcp.json`** for playwright config:
   ```bash
   cat .mcp.json 2>/dev/null | grep -q "playwright"
   ```

2. **If missing, create or merge** the playwright MCP entry:
   ```bash
   # If .mcp.json does not exist
   cat > .mcp.json << 'EOF'
   {
     "mcpServers": {
       "playwright": {
         "command": "npx",
         "args": ["@playwright/mcp@latest", "--user-data-dir", "/Users/wow/.playwright-mcp-profile"]
       }
     }
   }
   EOF
   ```
   If `.mcp.json` already exists with other servers, read it first and add the `playwright` key without overwriting existing entries.

3. **Install Playwright and Chromium**:
   ```bash
   npm install -D @playwright/mcp@latest
   npx playwright install chromium
   mkdir -p /Users/wow/.playwright-mcp-profile
   ```

4. **Inform user and stop**:
   ```
   Playwright MCP has been configured and Chromium installed.
   Restart the Claude Code session to activate browser tools.
   Browser verification will resume automatically on next run.
   ```
   Do NOT attempt browser operations in this session — MCP config loads at session start only.

### 0c. If MCP tools ARE available — proceed to Step 1

No setup needed. Continue with the verification workflow.

## Playwright MCP Tools

| Tool | Purpose |
|------|---------|
| `mcp__playwright__browser_navigate` | Open a URL |
| `mcp__playwright__browser_snapshot` | Get accessibility tree (element discovery) |
| `mcp__playwright__browser_click` | Click an element |
| `mcp__playwright__browser_type` | Type into an input field |
| `mcp__playwright__browser_fill_form` | Fill multiple form fields |
| `mcp__playwright__browser_take_screenshot` | Capture page state as evidence |
| `mcp__playwright__browser_press_key` | Press keyboard keys (Enter, Tab, Escape) |

## Workflow

### Step 1: Load Spec Requirements

Read the OpenSpec specs for the change under test:

```
openspec/changes/<name>/specs/       → MUST/SHOULD/COULD requirements
openspec/changes/<name>/design.md    → Expected UI behavior, routes, interactions
```

Build a verification checklist:

```markdown
## Verification Checklist: [change name]

| # | Requirement | Priority | Route | Action | Expected | Status |
|---|------------|----------|-------|--------|----------|--------|
| 1 | [MUST] User can submit form | CRITICAL | /form | Fill fields, click submit | Success message | PENDING |
| 2 | [MUST] Validation rejects empty email | CRITICAL | /form | Submit with empty email | Error shown | PENDING |
| 3 | [SHOULD] Loading spinner shown | HIGH | /form | Click submit | Spinner visible | PENDING |
```

### Step 2: Confirm Application is Running

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
```

If not running, start the application using the project's dev command (check `package.json` scripts or `Makefile`). Wait until the health check returns 200.

### Step 3: Diff-Aware Scope Detection

```bash
git diff origin/main --name-only
```

Map changed files to affected routes. Focus testing on these routes only.

### Step 4: Execute Verification Checklist

For each checklist item, execute this sequence:

#### 4a. Navigate to the route

```
→ mcp__playwright__browser_navigate({ url: "http://localhost:3000/route" })
```

#### 4b. Snapshot for element discovery

```
→ mcp__playwright__browser_snapshot()
```

Read the accessibility tree to find the correct element references (buttons, inputs, links). Cache the snapshot — do not re-snapshot on the same page unless a selector fails.

#### 4c. Execute the spec scenario

For each Given/When/Then step:

**Given** (setup state):
```
→ mcp__playwright__browser_navigate({ url: "..." })
→ mcp__playwright__browser_fill_form({ ... })   // if pre-fill needed
```

**When** (perform action):
```
→ mcp__playwright__browser_click({ element: "Submit button" })
   or
→ mcp__playwright__browser_type({ element: "Email input", text: "test@example.com" })
   or
→ mcp__playwright__browser_press_key({ key: "Enter" })
```

**Then** (verify outcome):
```
→ mcp__playwright__browser_snapshot()    // check element state
→ mcp__playwright__browser_take_screenshot()  // visual evidence
```

Verify the snapshot contains the expected element state:
- Success message visible in accessibility tree
- Error text present with correct message
- Element enabled/disabled state matches expectation
- Navigation occurred (URL changed)
- New content appeared in the DOM

#### 4d. Screenshot as evidence

```
→ mcp__playwright__browser_take_screenshot()
```

Save screenshots to `.worklog/` under the current task's phase directory:
```
.worklog/{yyyymm}/{task}/phase-{n}-qa/screenshots/
  ├── req-001-form-submit-success.png
  ├── req-002-validation-empty-email.png
  └── req-003-loading-spinner.png
```

#### 4e. Record result

Update the checklist item status: `PASS`, `FAIL`, or `BLOCKED`.

For failures, document immediately:
- What the spec says must happen
- What actually happened (from snapshot + screenshot)
- Console errors (check snapshot for error elements)

### Step 5: State Coverage Testing

For each affected route, verify these states exist and render correctly:

| State | How to Trigger | What to Verify |
|-------|---------------|----------------|
| Empty | Navigate with no data / clear data | Empty state message or illustration shown |
| Loading | Trigger async action, observe | Spinner or skeleton visible |
| Error | Submit invalid data / disconnect API | Error message with recovery path |
| Success | Complete happy path flow | Success feedback shown |
| Edge | Long text, special chars, boundary values | No overflow, no crash, graceful handling |

For each state:
```
→ mcp__playwright__browser_navigate({ url: "..." })
→ [trigger the state]
→ mcp__playwright__browser_snapshot()
→ mcp__playwright__browser_take_screenshot()
```

### Step 6: Health Scoring

After all tests, compute the health score:

| Dimension | Weight | Scoring |
|-----------|--------|---------|
| Console errors | 20 | -5 per JS error found in snapshots, -2 per warning |
| Spec compliance | 30 | (passed MUST + SHOULD) / total requirements |
| Broken interactions | 20 | -10 per broken flow (click does nothing, nav fails) |
| Visual correctness | 15 | Layout intact, no overflow, responsive |
| Accessibility | 15 | ARIA labels present, keyboard navigable, semantic HTML |

### Step 7: Generate Regression Tests

For each verified flow, generate a Playwright test file:

```typescript
// tests/e2e/browser-verify-{change-name}.spec.ts
// Generated from browser-verify — review before committing

import { test, expect } from '@playwright/test';

test('[REQ-001] User can submit form', async ({ page }) => {
  await page.goto('/form');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="name"]', 'Test User');
  await page.click('button[type="submit"]');
  await expect(page.locator('.success-message')).toBeVisible();
});

test('[REQ-002] Validation rejects empty email', async ({ page }) => {
  await page.goto('/form');
  await page.click('button[type="submit"]');
  await expect(page.locator('.error-message')).toContainText('Email is required');
});
```

Use selectors discovered from `browser_snapshot` accessibility tree. Save to the project's test directory.

## Output Format

```markdown
## Browser Verification Report

**Change**: [OpenSpec change name]
**Date**: [YYYY-MM-DD]
**Base URL**: http://localhost:3000

### Health Score: [N/100]

### Verification Checklist

| # | Requirement | Priority | Status | Evidence |
|---|------------|----------|--------|----------|
| 1 | [MUST] Form submission works | CRITICAL | PASS | screenshots/req-001.png |
| 2 | [MUST] Validation rejects empty | CRITICAL | PASS | screenshots/req-002.png |
| 3 | [SHOULD] Loading spinner | HIGH | FAIL | screenshots/req-003.png |

### Spec Compliance
- MUST: N/N passed (N%)
- SHOULD: N/N passed (N%)
- Total scenarios verified: N

### Issues Found

#### Issue 1: Loading spinner missing
- **Severity**: HIGH
- **Requirement**: [SHOULD] Display loading indicator during form submission
- **Route**: /form
- **Steps**: Fill form → click Submit → observe
- **Expected**: Spinner visible between click and response
- **Actual**: Button text unchanged, no visual feedback
- **Screenshot**: screenshots/req-003.png

### State Coverage

| Route | Empty | Loading | Error | Success | Edge |
|-------|-------|---------|-------|---------|------|
| /form | N/A | FAIL | PASS | PASS | PASS |

### Generated Tests
- `tests/e2e/browser-verify-{change}.spec.ts`: N tests

### Verdict: [PASS | CONDITIONAL PASS | FAIL]
```

Verdict criteria:
- **PASS**: All MUST pass, no CRITICAL issues
- **CONDITIONAL PASS**: All MUST pass, SHOULD or state coverage issues exist
- **FAIL**: Any MUST fails

## Example

Input: `/browser-verify checkout-redesign`

Execution:
1. Read `openspec/changes/checkout-redesign/specs/` → 5 MUST, 3 SHOULD requirements
2. `curl localhost:3000` → 200 OK
3. `git diff origin/main --name-only` → `src/pages/checkout.tsx`, `src/api/cart.ts`
4. Navigate to `/checkout`:
   ```
   → browser_navigate({ url: "http://localhost:3000/checkout" })
   → browser_snapshot() → find "Place Order" button, "Cart Items" list
   → browser_fill_form({ fields: { quantity: "2" } })
   → browser_click({ element: "Place Order" })
   → browser_snapshot() → confirm "Order Confirmed" message
   → browser_take_screenshot() → req-001-place-order.png
   ```
5. Verify empty cart redirect:
   ```
   → browser_navigate({ url: "http://localhost:3000/checkout" })
   → browser_snapshot() → FAIL: blank page, no redirect to /cart
   → browser_take_screenshot() → req-005-empty-cart.png
   ```
6. Health: 82/100 (1 SHOULD failing)
7. Generate 8 Playwright tests
8. Verdict: CONDITIONAL PASS

## Guardrails

- Do not modify application code — verify only
- Every MUST requirement must have a screenshot, whether it passes or fails
- Always run `browser_snapshot` before interacting — never guess at selectors
- Do not re-snapshot the same page unless a selector fails
- Report honestly — a blank page is a FAIL, not "needs investigation"
- Generated tests are drafts — mark for human review before committing
- One browser interaction at a time — do not attempt parallel browser sessions
