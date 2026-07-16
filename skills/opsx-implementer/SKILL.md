---
name: opsx-implementer
disable-model-invocation: true
description: >-
  Delegate OpenSpec task groups to a CLI coding agent (OpenCode or Codex) for
  implementation, then review and land it yourself.
license: MIT
compatibility: Requires `opencode` or `codex` CLI installed and authenticated, git, and an OpenSpec change directory.
metadata:
  version: 0.1.0
---

# OpenSpec Implementer Delegate

You are the **orchestrator**. This skill delegates a bounded task group from an
[OpenSpec](https://github.com/fission-ai/openspec) change to a CLI coding agent for
implementation, then you verify and land it yourself. You construct a prompt from
the change's spec, design, and tasks, and pipe it directly to the implementer CLI.

An implementer is a tool, not the default choice. Use it only when both conditions are met:

1. Delegation can save context or tokens.
2. Only the final result matters — intermediate steps have no lasting value.

## When to delegate vs. do it yourself

Evaluate each task group before delegating:

| Condition                                                         | Tendency                                  |
| ----------------------------------------------------------------- | ----------------------------------------- |
| Simple, judgmental, or few-line change                            | **Do it yourself** — avoid overhead       |
| Complex implementation, multi-file, needs independent reasoning   | **Delegate** — protect your context       |
| Only result matters, intermediate process has no value            | **Delegate** — perfect fit                |
| Code review of implementer's work                                 | **Do it yourself** — you are the reviewer |

## 0. Choose the implementer

Ask the user: "Which implementer? (OpenCode or Codex)" Record the choice. It stays fixed for the delegation loop — all task groups in this session use the same CLI.

## Prerequisites

Run the chosen CLI's `preflightCheck`. If the CLI is unavailable or unauthenticated, report the error to the user and stop — do not attempt to install it. An OpenSpec change must exist with specs, design, and tasks (`openspec list --json`), and you must be in (or point the dispatch command at) the target git repository.

## The delegation loop

### 1. Select a task group

First verify the change is implementable:
`openspec instructions apply --change "{name}" --json`
Stop if `state` is `blocked` or `all_done`.

Then read `tasks.md` and pick one task group (or subset, at your discretion).
Record the capability name from `openspec status` or the spec directory name.

For independent groups, default to **parallel**; use serial only where updates overlap.
Parallel groups must be **non-overlapping** everywhere (files modified, commands run) — if two tasks both depend on `Task A`, implement `Task A` first, then the rest serially.

### 2. Construct the prompt

Source the brief from project facts and the template at [references/writing-the-brief.md](references/writing-the-brief.md).

### 3. Dispatch

Run the chosen CLI's `dispatchBrief`.

### 4. Capture the session/thread ID

Use the CLI's `extractSessionId`/`extractThreadId` to record and resume the session.

#### Record

Annotate `tasks.md` after the corresponding task.
The example below means task 1 was mainly done by session1, but sub-task 1.1 was done by session2:

```
## 1. task 1 [opsx-implementer opencode/codex {session1}]

- [x] 1.1 xxx [opsx-implementer opencode/codex {session2}]
- [x] 1.2 xxx
- [x] 1.3 xxx
```

### 5. Wait

The implementer CLI is blocking — it returns when the implementer finishes (or fails). The dispatch command **is** the wait. After it exits:

- exit code 0 → implementer completed
- exit code non-zero → read stderr for the cause
- **Hung** (did not exit after expected time). Read the output and analyze.

Do not poll for progress during the wait — it wastes tokens and context.
Setting a longer timeout or using a notification mechanism is optional.

### 6. Review — do not trust the self-report

- Read the report via the chosen CLI's `readReport`.
- Check the task summary.
  Verify nothing was missed or misaligned against the original brief.
- Review the implementer's changes against the project's code conventions.
- Mark genuinely completed tasks in `tasks.md`.

### 7. Land or rework

If the work is correct, mark it done in OpenSpec.
If rework is needed, use `resumeSession` to hand it back to the original CLI agent, or do it yourself.

## References

- [references/writing-the-brief.md](references/writing-the-brief.md) — sourcing the brief from the
  OpenSpec change, the report contract.
- [references/opencode-operations.md](references/opencode-operations.md) — OpenCode CLI commands:
  `preflightCheck`, `dispatchBrief`, `extractSessionId`, `readReport`, `resumeSession`.
- [references/codex-operations.md](references/codex-operations.md) — Codex CLI commands:
  `preflightCheck`, `dispatchBrief`, `extractThreadId`, `readReport`, `resumeSession`.
