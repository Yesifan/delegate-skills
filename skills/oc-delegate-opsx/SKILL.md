---
name: oc-delegate-opsx
disable-model-invocation: true
description: >-
  Delegate OpenSpec task groups to the OpenCode CLI for implementation, then review
  and land it yourself.
license: MIT
compatibility: Requires `opencode` CLI installed and authenticated, git, and an OpenSpec change directory.
metadata:
  version: 0.1.0
---

# OpenCode OpenSpec Delegate

You are the **orchestrator**. This skill delegates a bounded task group from an
[OpenSpec](https://github.com/fission-ai/openspec) change to the OpenCode CLI for
implementation, then you verify the result and land it yourself. You construct a prompt from
the change's spec, design, and tasks, and pipe it directly to `opencode run`.

An implementer is a tool, not the default choice. Use it only when both of the following conditions are met:

1. Delegation can save context or tokens.
2. The task’s intermediate execution steps have no lasting value; only the final result matters.

## When to delegate vs. do it yourself

Evaluate each task group before delegating:

| Condition                                                         | Tendency                                  |
| ----------------------------------------------------------------- | ----------------------------------------- |
| Simple, judgmental, or few-line change                            | **Do it yourself** — avoid overhead       |
| Complex implementation, multi-file, needs independent reasoning   | **Delegate** — protect your context       |
| Task where intermediate process has no value, only result matters | **Delegate** — perfect fit                |
| Code review of implementer's work                                 | **Do it yourself** — you are the reviewer |

## Prerequisites

`opencode` CLI is available and authenticated (`opencode --version`, `opencode auth list` shows at
least one credential), an OpenSpec change exists with specs, design, and tasks artifacts
(`openspec list --json`), and you are in (or will point `--dir` at) the target git repository. If
`opencode` is not on PATH, report the error and tell the user to install (`npm i -g opencode-ai`) — do not attempt to install it yourself. Full pre-flight and recovery:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

## The delegation loop

### 1. Select a task group

First verify the change is implementable:
`openspec instructions apply --change "{name}" --json`
Stop if `state` is `blocked` (missing artifacts) or `all_done` (no work remains).

Then read the change's `tasks.md` and pick one task group.
Record the capability name from `openspec status` or the spec directory name. If multiple groups have
no dependency, you can run them in parallel (see [references/multi-task-queues.md](references/multi-task-queues.md)).

### 2. Construct the prompt

Read the delta spec, design.md, and main spec. Discover the project's **real
gate commands** from the repo's `CLAUDE.md`, `AGENTS.md`, `Makefile`, or `package.json` — do not
assume or hardcode — and embed them in the `<verification_loop>` block. Full template:
[references/writing-the-brief.md](references/writing-the-brief.md).

### 3. Dispatch

Pipe the prompt to `opencode run`, redirecting stdout to a fixed-path output file keyed by the
capability and task id.
Use `--dir <repo>` to set the working root. The full command:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

### 4. Capture the session ID

Extract the session ID from the captured `.jsonl` and record it in `tasks.md` (git-tracked) for
future resume. The extraction command and the `delegate: opencode {session_id}` back-link:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

### 5. Review — do not trust the self-report

Three layers, in order: check test integrity (before the gates mean anything),
re-run the gates yourself, then walk the implementer sweep against every changed file. Surface design
decisions; stop if correct completion requires going beyond the spec — scope changes are the human's
call. Full checklist: [references/review-and-land.md](references/review-and-land.md).

### 6. Land or rework

If the work is correct and gates pass, commit it. If it needs changes, resume the session with a
delta prompt — don't restate the whole task. The resume command and the rework cycle:
[references/review-and-land.md](references/review-and-land.md). If more task groups remain, return
to step 1.

## References

- [references/writing-the-brief.md](references/writing-the-brief.md) — sourcing the brief from the
  OpenSpec change, the four-block prompt template, the report contract, and picking the model per task.
- [references/dispatch-and-poll.md](references/dispatch-and-poll.md) — the `opencode run` command
  shape, the `--auto` rule, session-id capture, and recovery when a run misbehaves.
- [references/review-and-land.md](references/review-and-land.md) — the three review layers, the
  implementer sweep, the commit boundary, and the resume-based rework cycle.
- [references/multi-task-queues.md](references/multi-task-queues.md) — sequential vs parallel
  groups, carrying constraints forward, the end-of-run coherence check, and interleaving with the
  OpenSpec lifecycle.
