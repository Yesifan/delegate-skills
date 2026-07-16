# Codex operations

```
## preflightCheck

codex --version
codex login status

## dispatchBrief

codex exec -s workspace-write --json -o /tmp/delegate_final.txt -C /path/to/repo -- <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[prompt content]
PROMPT

Output files:
- `/tmp/delegate_{capability}_{task-id}.jsonl` — event stream (carries `thread_id`)
- `/tmp/delegate_final.txt` — final structured report (-o target)

## extractThreadId

THREAD_ID=$(grep -o '"thread_id":"[^"]*"' /tmp/delegate_{capability}_{task-id}.jsonl \
  | head -1 | cut -d'"' -f4)



## readReport

cat /tmp/delegate_final.txt

## resumeSession

codex exec resume {thread_id} --json -o /tmp/delegate_final.txt -- <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[delta prompt]
PROMPT

