---
name: opsx-implementer
disable-model-invocation: true
description: >-
  Delegate OpenSpec task groups to a CLI coding agent (OpenCode or Codex) for
  implementation, then review and land it yourself.
license: MIT
compatibility: >-
  Requires `opencode` or `codex` CLI installed and authenticated, or a running
  OpenCode server (`opencode serve`), git, and an OpenSpec change directory.
metadata:
  version: 0.1.0
---

# OpenSpec Implementer

You are the **orchestrator**. This skill delegates a bounded task group from an [OpenSpec](https://github.com/fission-ai/openspec) change to a CLI coding agent for implementation, then you verify and land it yourself.

You construct a brief from the change's spec, and pipe it directly to the implementer.

## When to delegate vs. do it yourself

An implementer is a tool, not the default choice. Use it only when both conditions are met:

1. Delegation can save context or tokens.
2. Only the final result matters — intermediate steps have no lasting value.

## 0. Choose the implementer

Ask the user: "Which implementer? (OpenCode CLI, OpenCode Server, or Codex CLI)"
Record the choice. It stays fixed for the delegation loop.

If **OpenCode Server** is selected, ask for host and port (default: `127.0.0.1:4096`).

## Prerequisites

Run the chosen CLI's `preflightCheck`. If the CLI is unavailable or unauthenticated, report the error to the user and stop — do not attempt to install it.

An OpenSpec change must exist with specs, design, and tasks

## The delegation loop

For simple tasks, delegate and execute them all at once.
For complex tasks, plan the execution upfront, then delegate in multiple rounds following the plan.

### 1. Select a task group

First verify the change is implementable:
`openspec instructions apply --change "{name}" --json`
Stop if `state` is `blocked` or `all_done`.

Then read `tasks.md` and pick one task group (or subset, at your discretion).

For independent groups, default to **parallel**; use serial only where updates overlap.
Parallel groups must be **non-overlapping** everywhere (files modified, commands run) — if two tasks both depend on `Task A`, implement `Task A` first, then the rest serially.

### 2. Construct the brief

Source the brief from project facts and the template at [references/writing-the-brief.md](references/writing-the-brief.md).

### 3. Dispatch and capture session

Run the chosen implementer's `dispatchBrief` or `resumeSession` from the
operations reference.

Session ID comes from the implementer's own mechanism — see the operations
reference for how to extract it.

#### Record

Annotate `tasks.md` after the corresponding task.
The example below means task 1 was mainly done by session1, but sub-task 1.1 was done by session2:

```md
## 1. task 1 [opsx-implementer opencode/codex {session1}]

- [x] 1.1 xxx [opsx-implementer opencode/codex {session2}]
- [x] 1.2 xxx
```

### 4. Wait the task end

### 5. Review

If there are parallel tasks, you should wait for all parallel tasks to complete and then conduct an overall review based on the reports and git diff.

### 6. Land or rework

If the work is correct, mark it done in OpenSpec.
If rework is needed, use `resumeSession` to hand it back to the original CLI agent, or do it yourself.

### 7. continue to the next round or finish

If have next round,
Stage the previous round's changes with `git add` so the next round can `git diff` cleanly.
Update the spec status.
Go back to step 1 for the next round.

If it's finished,
report the complete task execution status.

If there are problems,
end the task as well and report the current execution status and the details of the problems.

## References

- [references/writing-the-brief.md](references/writing-the-brief.md) — sourcing the brief from the
  OpenSpec change, the report contract.

- Query the methods of [`preflightCheck`, `dispatchBrief`, `extractSessionId`, `readReport`, `resumeSession`] in the documents based on the currently used implementer driver.
  - [references/opencode-operations.md](references/opencode-operations.md) — OpenCode CLI
  - [references/opencode-server-operations.md](references/opencode-server-operations.md) — OpenCode Server API
  - [references/codex-operations.md](references/codex-operations.md) — Codex CLI
