# Exploring the codebase

Delegate codebase exploration to the implementer when the orchestrator has a question about the
local repository — architecture understanding, root cause analysis, impact assessment, or any
investigation whose intermediate process is disposable but whose findings matter.

## When to delegate exploration

| Condition | Tendency |
|-----------|----------|
| Simple grep / single-file read | **Do it yourself** — overhead not worth it |
| Multi-file investigation across modules | **Delegate** — protect orchestrator context |
| Need evidence (file paths, line numbers, snippets) | **Delegate** — implementer collects systematically |
| Subjective judgement call | **Do it yourself** — orchestrator owns the call |

## The explore brief

An explore brief is simpler than an implementation brief. Omit `<verification_loop>` and
`<action_safety>` — no gates, no edits. Dispatch with `--agent plan` and no `--auto`
(see [dispatch-and-poll.md](dispatch-and-poll.md)).

```xml
<role>
You are investigating a local codebase. Read files, search for patterns, and answer the
question below with evidence from the repository. Do NOT edit any files.
</role>

<task>
{the question — what to investigate, files/directories to examine, what "done" looks like}

What to leave untouched:
{explicitly — all files, since this is read-only}
</task>

<structured_output_contract>
End with a report in this exact shape:
  1. Findings — direct answer to the question
  2. Evidence — file paths, line numbers, key snippets for each finding
  3. Open questions or follow-ups if any
</structured_output_contract>
```

## Processing the results

The orchestrator reads the report from the captured output and uses the findings — answer the
user, update design docs, or decide next steps. No commit step; the implementer produces a report,
not a diff.

## See also

- [dispatch-and-poll.md](dispatch-and-poll.md) — the dispatch command (use `--agent plan`, no
  `--auto`).
- [multi-task-queues.md](multi-task-queues.md) — explore tasks can interleave with
  implementation tasks.