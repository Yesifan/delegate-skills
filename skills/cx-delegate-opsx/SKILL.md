---
name: cx-delegate-opsx
description: >-
  Delegate OpenSpec task groups or codebase exploration to the Codex CLI, then review
  and land it yourself. Use for implementation when a change is ready — phrasings like
  "delegate to Codex", "have Codex implement this". Use for local codebase exploration —
  phrasings like "have Codex look at", "ask Codex what X does". Not for tasks small
  enough to do inline.
license: MIT
compatibility: Requires `codex` CLI installed and authenticated, Node 18+, git, and an OpenSpec change directory.
metadata:
  version: 0.1.0
---

# Codex OpenSpec Delegate

You are the **orchestrator**. This skill delegates a bounded task group from an
[OpenSpec](https://github.com/fission-ai/openspec) change to the OpenAI Codex CLI for
implementation, then you verify the result and land it yourself. You construct a prompt from
the change's spec, design, and tasks, and pipe it directly to `codex exec`.

An implementer is a tool, not the default choice. Use it only when both of the following conditions are met:

1. Delegation can save context or tokens.
2. The task’s intermediate execution steps have no lasting value; only the final result matters.

## When to delegate vs. do it yourself

Evaluate each task group before delegating:

| Condition                                                         | Tendency                                               |
| ----------------------------------------------------------------- | ------------------------------------------------------ |
| Simple, judgmental, or few-line change                            | **Do it yourself** — avoid overhead                    |
| Complex implementation, multi-file, needs independent reasoning   | **Delegate** — protect your context                    |
| Task where intermediate process has no value, only result matters | **Delegate** — perfect fit                             |
| Exploratory / research question                                   | **Delegate explore mode** — don't pollute your context |
| Code review of implementer's work                                   | **Do it yourself** — you are the reviewer              |

## Prerequisites

`codex` CLI is available and authenticated (`codex --version`, `codex login status`), an OpenSpec
change exists with specs, design, and tasks artifacts (`openspec list --json`), and you are in (or
will point `--cd` at) the target git repository. If `codex` is not on PATH, report the error and tell
the user to install (`npm i -g @openai/codex`) and run `codex login` — do not attempt to install it
yourself. Full pre-flight and recovery:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

## Implementation delegation vs. exploration

This skill covers two modes:

- **Implementation**: delegate a task group from an OpenSpec change. Follow the loop below
  (steps 1-6).
- **Exploration**: delegate a codebase question — no OpenSpec needed, no edits, no commit.
  Use the template in [references/exploring-the-codebase.md](references/exploring-the-codebase.md),
  dispatch with `-s read-only`. When the run finishes, read the report and use the findings.
  No gate re-run or implementer sweep — evidence quality is verified by reading the findings
  against the question.

## The delegation loop

### 1. Select a task group

First verify the change is implementable:
`openspec instructions apply --change "{name}" --json`
Stop if `state` is `blocked` (missing artifacts) or `all_done` (no work remains).

Then read the change's `tasks.md` and pick one task group (e.g. "2. Model layer") whose tasks are
independent of other groups. Record the capability name from `openspec status` or the spec directory
name. If multiple groups have no dependency, you can run them in parallel (see
[references/multi-task-queues.md](references/multi-task-queues.md)).

### 2. Construct the prompt

**Implementation**: read the delta spec, design.md, and main spec. Discover the project's **real
gate commands** from the repo's `CLAUDE.md`, `AGENTS.md`, `Makefile`, or `package.json` — do not
assume or hardcode — and embed them in the `<verification_loop>` block. Full template:
[references/writing-the-brief.md](references/writing-the-brief.md).

**Exploration**: write the question and scope directly using the template in
[references/exploring-the-codebase.md](references/exploring-the-codebase.md).

### 3. Dispatch

Pipe the prompt to `codex exec`, redirecting stderr to a fixed-path output file keyed by the
capability and task id. Use `-C / --cd <repo>` to set the working root. The full command shape
(default write sandbox vs. read-only explore mode) and what the captured `.jsonl` carries:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

### 4. Capture the thread ID

Extract the thread ID from the captured `.jsonl` and record it in `tasks.md` (git-tracked) for
future resume. The extraction command and the `delegate: codex {thread_id}` back-link:
[references/dispatch-and-poll.md](references/dispatch-and-poll.md).

### 5. Review — do not trust the self-report

**Implementation**: Three layers, in order: check test integrity (before the gates mean anything),
re-run the gates yourself, then walk the implementer sweep against every changed file. Surface design
decisions; stop if correct completion requires going beyond the spec — scope changes are the human's
call. Full checklist: [references/review-and-land.md](references/review-and-land.md).

**Exploration**: verify evidence quality. The implementer's report should cite file paths and line
numbers — spot-check key citations against the repo. Surface any findings that need the human's
judgement.

### 6. Land or rework

If the work is correct and gates pass, commit it. If it needs changes, resume the session with a
delta prompt — don't restate the whole task. The `codex exec resume` command and the rework cycle:
[references/review-and-land.md](references/review-and-land.md). If more task groups remain, return
to step 1.

## OpenSpec lifecycle

This skill slots into OpenSpec's propose → apply → archive flow: after _propose_ the artifacts
exist but no code, so you delegate task groups; after _partial implementation_ you delegate the rest
and resume sessions for rework; after _all tasks done_ you sync delta specs to main specs
(`openspec-sync-specs`) and archive. Each delegation step is an _apply_ step inside that flow; you
don't run the lifecycle here, you interleave with it. Depth:
[references/multi-task-queues.md](references/multi-task-queues.md).

## References

- [references/writing-the-brief.md](references/writing-the-brief.md) — sourcing the brief from the
  OpenSpec change, the four-block prompt template, the report contract, and picking the model per task.
- [references/dispatch-and-poll.md](references/dispatch-and-poll.md) — the `codex exec` command
  shape, explore mode, thread-id capture, and recovery when a run misbehaves.
- [references/review-and-land.md](references/review-and-land.md) — the three review layers, the
  implementer sweep, the commit boundary, and the resume-based rework cycle.
- [references/multi-task-queues.md](references/multi-task-queues.md) — sequential vs parallel
  groups, carrying constraints forward, the end-of-run coherence check, and interleaving with the
  OpenSpec lifecycle.
- [references/exploring-the-codebase.md](references/exploring-the-codebase.md) — when to delegate
  exploration, the explore brief template, and how to process findings.
