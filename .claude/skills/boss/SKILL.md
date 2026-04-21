---
name: Boss
description: Entry point that spawns the Tech Lead to run the full D-Team OpenSpec+GSD workflow
disable-model-invocation: true
allowed-tools: ["Agent"]
argument-hint: "[task description or context for Tech Lead]"
---

# Boss

## Description

Launch the Tech Lead to run the complete D-Team development workflow (OpenSpec spec alignment → GSD execution → Fix-first review → dual verification → QA → release). Use this skill as the standard entry point for all D-Team requests.

## Trigger

Use when the user wants to start a task that falls under D-Team's scope (feature development, bug fix, refactor, infrastructure work). The Tech Lead orchestrates all subordinate agents (spec / core / quality / tools) and decides the scale-based strategy (micro / small / medium / large).

## Execution

When this skill is invoked, spawn the Tech Lead agent to handle the entire workflow:

1. Parse any arguments the user provided (task description, scale hint, constraints, context)
2. Spawn the `tech-lead` agent via the Agent tool with `subagent_type: "Tech Lead"`
3. Pass the user's request verbatim as the agent's prompt
4. The Tech Lead runs the full workflow defined in `.claude/agents/tech-lead.md`, including scale decision, phase dispatch, worklog management, and safety mechanisms (WTF-likelihood stop-loss, Fix-first review)

### Spawn Instructions

Use the Agent tool with these parameters:

- `subagent_type`: `"Tech Lead"`
- `description`: Short (3-5 word) summary of the task, e.g., `"Run D-Team workflow"`
- `prompt`: Include the user's original request verbatim. If no arguments were provided, instruct the Tech Lead to begin from intake — ask the user to describe the task, then select a scale-based strategy.

### With Arguments

```
/boss {task description}
```

Spawn Tech Lead with prompt:

```
Start the D-Team OpenSpec+GSD workflow.

<user_request>
{task description}
</user_request>

Decide the scale (micro/small/medium/large) per rules/scale-strategy, then dispatch through the standard phase sequence. Create the worklog at .worklog/{yyyymm}/{task-name}/ before dispatching any phase work.
```

### Without Arguments

```
/boss
```

Spawn Tech Lead with prompt:

```
The user invoked /boss with no arguments. Begin intake:
1. Ask the user to describe the task, constraints, and any existing OpenSpec change name.
2. Decide scale (micro/small/medium/large).
3. Dispatch the standard D-Team workflow.
```

## Examples

### Normal Case — medium feature

User: `/boss 加一個可以用 Google 登入的 API endpoint`

Action: Spawn Tech Lead with the user's request. Tech Lead classifies as medium, runs OpenSpec propose → GSD plan → execute → review → verify → QA → ship.

### Micro Fix Case

User: `/boss 修掉 user service 裡的 nil pointer`

Action: Spawn Tech Lead. Tech Lead classifies as micro, routes to `/gsd:quick` or direct code change with code-reviewer.

### No Arguments Case

User: `/boss`

Action: Spawn Tech Lead to begin interactive intake — clarify goal, scale, and existing OpenSpec state before dispatching.
