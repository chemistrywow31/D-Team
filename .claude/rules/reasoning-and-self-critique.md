---
name: Reasoning and Self-Critique
description: Every agent must include structural Reasoning and Self-Critique gates around its workflow
---

# Reasoning and Self-Critique

## Applicability

- Applies to: All D-Team agents

## Rule Content

### Two Structural Gates Around Every Workflow

Every agent must enforce two structural gates around its workflow:

1. **`## Reasoning` gate** — runs before the workflow. Forces the agent to think before acting.
2. **`## Self-Critique` gate** — runs after the workflow produces a draft, before submission. Forces the agent to challenge its own output.

Both gates are structural sections in the agent .md file. Claude reliably follows structural boundaries; the gates work because the template forces the agent to fill them in.

### Section Ordering in Agent Template

Every agent .md file must place these two sections relative to `## Workflow` as follows:

```
## Role
## Responsibilities
## Reasoning            ← Gate 1: think before acting
## Workflow             ← Execute (or "Implementation Workflow")
## Self-Critique        ← Gate 2: challenge before submitting
## Available Skills
## Examples
... rest
```

### Canonical `## Reasoning` Block

Every agent's `## Reasoning` section must contain four labeled subsections.

```markdown
## Reasoning

Before executing the workflow, complete this reasoning gate. Do not start the workflow until all four slots are filled. Write the reasoning to the worklog or to a structured note in your task return.

### Knowns
- {What information is confirmed? What inputs are available? Which OpenSpec/GSD artifacts apply?}

### Unknowns
- {What is missing? What assumptions are being made? What would need to be verified?}

### Plan
- {What approach will be taken? Why this approach over alternatives?}

### Risks
- {What could go wrong? Which assumptions, if false, would invalidate the plan? What is the falsification condition?}
```

### Canonical `## Self-Critique` Block

Every agent's `## Self-Critique` section must contain five labeled checks.

```markdown
## Self-Critique

After producing draft output, run this critique pass before submission. If any check exposes a gap, revise the draft and re-run all five checks. Submit only when every check passes, or escalate per the Uncertainty Protocol when revision cannot close the gap.

### Evidence Check
- Does every claim trace back to a source, finding, OpenSpec scenario, or upstream worklog entry?

### Position Check
- Did I take a clear position with stated reasoning, or did I hedge with vague agreement?

### Counterexample Check
- What is the strongest argument against this output? Did I address it?

### Completeness Check
- Does the output answer the actual task scope, or only the easy parts?

### Failure Mode Check
- Where would this output break first under realistic downstream use?
```

### Coordinators Run a Pre-Dispatch Variant

Tech Lead must additionally run a **Pre-Dispatch Reasoning** before each agent dispatch:

```markdown
## Pre-Dispatch Reasoning (Coordinator only)

Before dispatching any agent, fill this gate:

### What This Dispatch Must Achieve
- {Single concrete outcome}

### Why This Agent
- {Why this agent over alternatives}

### Inputs the Agent Needs
- {Worklog paths, OpenSpec change name, design.md path, GSD plan path}

### Predicted Failure Modes
- {What the agent might get wrong; what to check on return}
```

### When the Gates Apply

Both gates apply to every output that crosses an agent boundary:
- Files generated (specs, design.md, code commits, review reports)
- Decisions written to `decisions.md`
- Reports returned to the Tech Lead
- Recommendations delivered to the user

### Self-Critique Cannot Be Outsourced

The agent that produces the output must run its own Self-Critique. Code-reviewer, security-reviewer, process-reviewer, spec-test-auditor are additional layers — not replacements.

### Tier 1 Agents

Build-resolver, refactor-cleaner, and doc-updater are Tier 1 utilities. They may use a reduced 2-slot Self-Critique:

```markdown
## Self-Critique

### Format Check
- Does the output match the required format exactly?

### Input Coverage Check
- Was every required input field consumed?
```

Tier 1 agents may omit `## Reasoning` only when the agent .md states explicit Tier 1 justification (zero judgment calls).

### Failure Recovery

If Self-Critique exposes a gap that revision cannot close after 3 attempts, the agent must escalate via the Uncertainty Protocol with `INSUFFICIENT_DATA` or `BLOCKED`.

## Violation Determination

- Agent .md missing `## Reasoning` section (and not Tier 1) → Violation
- Agent .md missing `## Self-Critique` section → Violation
- `## Reasoning` placed after `## Workflow` → Violation
- `## Self-Critique` placed before `## Workflow` → Violation
- `## Reasoning` block missing any of the four canonical slots → Violation
- `## Self-Critique` block missing any of the five canonical checks (Tier 2+) → Violation
- Tech Lead missing `## Pre-Dispatch Reasoning` section → Violation
- Tier 1 reduction declared without justification → Violation

## Exceptions

- Tier 1 agents may use the reduced Self-Critique format and may omit `## Reasoning` per the Tier 1 carve-out.

## Tradeoff

Tradeoff: Both gates add structured reasoning steps before and after every workflow. For trivial tasks the cost is real — typically 30-60 extra seconds of reasoning per dispatch. The payoff is reliability — agents catch their own evidence gaps and unaddressed counterexamples before downstream agents have to.
