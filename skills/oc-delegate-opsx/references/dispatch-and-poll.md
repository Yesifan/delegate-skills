# Dispatch and poll

## Before the first run: check the binary

```bash
opencode --version       # session support varies by version
```

During the execution of `opencode`, report any problems to the human and let the human solve them. Do not attempt to install or repair by yourself.

## Dispatching

Pipe the prompt to OpenCode, redirecting stdout to a fixed-path output file keyed by the capability
and task id. Use `--dir <repo>` to set the working root (default: current directory):

```bash
opencode run --agent build [--model {provider/model}] --dir /path/to/repo \
  --format json --auto -- <<'PROMPT' \
  > /tmp/delegate_{capability}_{task-id}.jsonl \
  2>/tmp/delegate_{capability}_{task-id}.log
[prompt content]
PROMPT
```

The `--format json` stream is newline-delimited JSON events; `/tmp/delegate_{capability}_{task-id}.jsonl`
captures them; stderr goes to a sibling `.log`. No relay script, no temp artifacts directory — the orchestrator reads these two files.

## JSONL event format

Each line is one JSON event with a `type` field. The format is a sequential event stream
recording the run:

| Event type    | When                                            | Useful fields                                                                               |
| ------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `step_start`  | A new step begins                               | `sessionID` — the run's session identifier                                                  |
| `tool_use`    | The agent calls a tool (bash, read/write, etc.) | `part.tool`, `part.state.input`, `part.state.output`                                        |
| `step_finish` | A step ends                                     | `part.reason` (`tool-calls` / `stop`), `part.tokens`, **`part.cost`** — step-level USD cost |
| `text`        | The agent emits text output                     | **`part.text`** — the implementer's response content                                        |

Every event carries these common fields: `type`, `timestamp` (unix ms), `sessionID`.

Typical event sequence:

1. `step_start` — run confirmed
2. `tool_use` / `tool_use` / ... — the agent works (reads files, runs bash, writes code)
3. `step_finish` — work phase done, with cost and token count
4. `step_start` — response generation begins
5. `text` — the agent's final message (the `<structured_output_contract>` report)
6. `step_finish` — run ends, final cost and token count

The last `type: "text"` event is the implementer's report. Total cost is the sum of all
`step_finish` events' `part.cost`.

## Capture the session ID

After the run completes, extract the session ID from the captured output:

```bash
SESSION_ID=$(grep -o '"sessionID":"[^"]*"' /tmp/delegate_{capability}_{task-id}.jsonl \
  | head -1 | cut -d'"' -f4)
```

Record it in `tasks.md` for future resume:

```
- [ ] 2.1 Implement JWT middleware  `delegate: opencode {session_id}`
```

If the output file is lost, the session ID survives in `tasks.md` (git-tracked). You'll use it to
resume the session for rework (see [review-and-land.md](review-and-land.md)).

### Read the report

The implementer's structured-output report is in the captured `.jsonl` file — the last
`type: "text"` event is the implementer's response to the `<structured_output_contract>`
block. Extract it:

```bash
jq -r 'select(.type == "text") | .part.text' /tmp/delegate_{capability}_{task-id}.jsonl | tail -1
```

The report tells you what the implementer claims to have done and why. Total run cost can
also be extracted from the event stream:

```bash
jq -s '[.[] | select(.type == "step_finish") | .part?.cost // empty] | add | "Cost: \(.)"' /tmp/delegate_{capability}_{task-id}.jsonl
```

## Waiting for completion

`opencode run` is blocking — it returns when the implementer finishes (or fails).
The dispatch command above **is** the wait. After it exits:

- exit code 0 → implementer completed
- exit code non-zero → read the stderr `.log` file for the cause

Do NOT poll the `.jsonl` file mid-run with `wc -l` or `tail` — the event stream
may be buffered, so line count is misleading, and repeated checking burns your
token budget without gaining useful signal. Trust the exit code.

If you need mid-run visibility (e.g. to detect interactive prompt hangs), set a
reasonable timeout on the shell command itself — an idle timeout catches hangs
without per-round polling.

## When a run misbehaves

- **Run failed (non-zero exit).** Read the stderr captured in the `.log` file:
  ```bash
  tail -20 /tmp/delegate_{capability}_{task-id}.log
  ```
  Common causes: auth lapse, invalid `--model`, or a permission the run couldn't auto-approve. Fix
  the cause and re-dispatch with a delta prompt; do not paper over it by doing the work yourself.
- **Run hangs (no exit after expected duration).** The implementer likely hit an
  interactive prompt. Cancel the run, add the `<headless_environment>` block to
  the brief (see [writing-the-brief.md](writing-the-brief.md)), and re-dispatch.
- **Empty final message.** The implementer exited before producing a report. Treat as a failed run;
  the .jsonl event stream usually shows where it stopped.
