# Dispatch and poll

The opsx pattern dispenses with a relay script: you pipe the brief straight to `codex exec` and
redirect the JSON event stream to a fixed-path file. Your job is "run a command, redirect output,
read the file." That is deliberately less machinery than the standalone `codex-delegate` skill —
OpenSpec already supplies the structure the brief needs, so the helper layer is dead weight.

## Before the first run: check the binary

```bash
command -v codex      # the binary that will answer
codex --version       # must support exec --json, -o, and exec resume
```

`codex` CLI must be available and authenticated. If not, report the error and tell the user to
install (`npm i -g @openai/codex`) and run `codex login` — do not attempt to install it yourself.

## Dispatching

Pipe the prompt to Codex, redirecting stderr to a fixed-path output file keyed by the capability
and task id. Use `--cd <repo>` (or `-C <repo>`) to set the working root (default: current
directory):

```bash
codex exec -s workspace-write --json -o /tmp/delegate_final.txt -C /path/to/repo -- - <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[prompt content]
PROMPT
```

`-s workspace-write` is the write-capable sandbox. The output file at
`/tmp/delegate_{capability}_{task-id}.jsonl` captures the JSON event stream. The orchestrator does not
need a relay script.

## Capture the thread ID

After the run completes, extract the thread ID from the captured output:

```bash
THREAD_ID=$(grep -o '"thread_id":"[^"]*"' /tmp/delegate_{capability}_{task-id}.jsonl \
  | head -1 | cut -d'"' -f4)
```

Record it in `tasks.md` for future resume:

```
- [ ] 2.1 Implement JWT middleware  `delegate: codex {thread_id}`
```

If the output file is lost, the thread ID survives in `tasks.md` (git-tracked). You'll use it to
resume a session for rework (see [review-and-land.md](review-and-land.md)).

### Read the report

The implementer's structured output report is at `/tmp/delegate_final.txt` (the `-o` target of
`codex exec`). This file contains the final summary the implementer wrote in response to the
`<structured_output_contract>` block in your prompt. Read it before the review — it tells you
what the implementer claims to have done and why. The review checklist at
[review-and-land.md](review-and-land.md) tells you how to verify each claim against reality.

## When a run misbehaves

- **CLI unavailable.** `codex` is not on PATH — `codex --version` fails. Report the error and tell
  the user to install (`npm i -g @openai/codex`) and run `codex login`. Do not attempt to install it
  yourself.
- **Run failed (non-zero exit).** Read the stderr captured in the output file:
  ```bash
  tail -20 /tmp/delegate_{capability}_{task-id}.jsonl
  ```
  Common causes: auth lapse (`codex login status`), invalid `--model`, or the sandbox blocked a
  needed operation. Fix the cause and re-dispatch with a delta prompt; do not paper over it by
  doing the work yourself.
- **Empty final message.** The implementer exited before producing a report. Treat as a failed run;
  the events log usually shows where it stopped.

## What you'd give up by re-introducing a helper

A `relay.mjs` here buys you a captured `result.json` and a touched-files summary — but the
fixed-path `.jsonl` already carries the thread ID and the event stream, and `git status` in the
working root already shows what changed. The opsx workflow's structure comes from OpenSpec, not from
the dispatch layer; adding a helper would be ceremony the brief doesn't need.