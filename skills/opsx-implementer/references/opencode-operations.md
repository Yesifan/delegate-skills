# OpenCode operations

```
## preflightCheck

opencode --version
opencode auth list

## dispatchBrief

opencode run --agent build [--model {provider/model}] --dir /path/to/repo \
  --format json --auto -- <<'PROMPT' \
  > /tmp/delegate_{capability}_{task-id}.jsonl \
  2>/tmp/delegate_{capability}_{task-id}.log
[prompt content]
PROMPT

Output files:
- `/tmp/delegate_{capability}_{task-id}.jsonl` — event stream
- `/tmp/delegate_{capability}_{task-id}.log` — stderr

Event stream:
  - `step_start` — `sessionID`, `timestamp`
  - `tool_use` — `part.tool`, `part.state.input`, `part.state.output`
  - `step_finish` — `part.reason`, `part.tokens`, `part.cost`
  - `text` — `part.text`

Common fields on every event: `type`, `timestamp` (unix ms), `sessionID`.

## extractSessionId

SESSION_ID=$(grep -o '"sessionID":"[^"]*"' /tmp/delegate_{capability}_{task-id}.jsonl \
  | head -1 | cut -d'"' -f4)



## readReport

jq -r 'select(.type == "text") | .part.text' /tmp/delegate_{capability}_{task-id}.jsonl | tail -1

The last `type: "text"` event is the implementer's response to the
<structured_output_contract> block.

## resumeSession

opencode run --session {session_id} [--model {provider/model}] -- <<'PROMPT' \
  > /tmp/delegate_{capability}_{task-id}.jsonl \
  2>/tmp/delegate_{capability}_{task-id}.log
[delta prompt]
PROMPT

```
