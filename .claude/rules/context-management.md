---
name: Context Management
description: Control context size and information flow using GSD context engineering and summary-based reporting
---

# Context Management

## Applicability

- Applies to: All agents (coordinator and all worker agents)

## Rule Content

### Atomic Subtask Decomposition

The Tech Lead (coordinator) must break every task into focused, atomic subtasks. Each subtask must target a single concern — one file group, one feature slice, or one logical unit. You must not assign a subtask that spans multiple unrelated modules or features.

For medium+ features using GSD: The GSD planner handles task decomposition into atomic units with explicit `<files>`, `<verify>`, and `<done>` criteria. The Tech Lead orchestrates GSD commands rather than manually decomposing.

### Independent Context Windows (GSD Execution)

When using `/gsd:execute-phase`, each task runs in an independent 200K context window. This prevents quality degradation from context accumulation. The main session stays at low utilization, maintaining coordination quality.

Benefits:
- Each implementation task starts with clean context
- No cross-contamination between unrelated tasks
- Parallel tasks cannot pollute each other's context
- Main orchestration session remains lightweight

### Wave-Based Parallel Execution

GSD executor groups tasks by dependency into waves:
- Tasks within the same wave execute in parallel (independent context windows)
- Tasks across waves execute sequentially (respecting dependencies)
- Each completed task produces an atomic git commit

### Coordinator Must Include Context Paths in Task Dispatch

When the coordinator dispatches a task to any agent via the Task tool, the dispatch must include:

1. **Current worklog path**: The directory where this agent must write its outputs
2. **Upstream reference paths**: Paths to relevant upstream phase worklogs or OpenSpec artifacts that this agent must read for context
3. **Task scope summary**: A concise description of what this specific task must accomplish

The coordinator must not pass full upstream content inline. Pass paths; let the agent read what it needs.

### Summary-Based Reporting

Agents must report results as concise summaries containing:

- What was done (one to three sentences)
- Files created or modified (list of paths)
- Decisions made and their rationale (bullet points)
- Open issues or blockers (if any)

You must not dump full file contents, complete logs, or raw command output into reports. Summarize before passing information to the next agent.

### Context Injection Scoping

You must include only the files directly relevant to the current subtask when injecting context. You must not inject entire directories. Specify exact file paths or use targeted glob patterns scoped to the current concern.

### Context Limit Handling

When the context limit approaches 70% utilization, you must:

1. Checkpoint current progress by writing a summary to the task output
2. Complete the current atomic unit of work
3. Continue remaining work in a new step with a fresh context window

You must not attempt to continue complex work when context is near capacity.

### Session Persistence for Large Features

For features spanning multiple sessions:

1. Run `/gsd:pause-work` before ending a session — produces a handoff document with full state
2. Run `/gsd:resume-work` at the start of the next session — restores context from handoff
3. OpenSpec specs serve as cross-session requirement anchors — always re-read specs when resuming

### Large Code Review Splitting

You must split large code reviews by module or functional area. Each review pass must focus on one module. You must not review an entire codebase or multiple unrelated modules in a single context window.

## Violation Determination

- Coordinator assigns a subtask spanning multiple unrelated modules without decomposition → Violation
- Coordinator dispatches a Task without including worklog path and context paths → Violation
- Agent reports contain full file dumps or raw command output exceeding 50 lines instead of summaries → Violation
- Context injection includes an entire directory instead of specific files → Violation
- Agent continues working past 70% context utilization without checkpointing → Violation
- Code review covers multiple unrelated modules in a single pass → Violation
- Large feature session ends without `/gsd:pause-work` when work is incomplete → Violation

## Exceptions

- When debugging a cross-module issue that requires simultaneous visibility of multiple files, the coordinator may authorize a broader context injection, documented in the subtask description with a rationale
- Micro/small features (< 2 hr) do not require GSD context windows or session persistence — standard context management suffices
