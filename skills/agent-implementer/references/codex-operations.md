# Codex operations

## preflightCheck

```bash
codex --version
codex login status
```

## dispatchBrief

```bash
codex exec -s workspace-write --json -o /tmp/delegate_final.txt -C /path/to/repo -- <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[prompt content]
PROMPT
```

Output files:

- `/tmp/delegate_{capability}_{task-id}.jsonl` — event stream (carries `thread_id`)
- `/tmp/delegate_final.txt` — final structured report (-o target)

## extractThreadId

```bash
THREAD_ID=$(grep -o '"thread_id":"[^"]*"' /tmp/delegate_{capability}_{task-id}.jsonl \
  | head -1 | cut -d'"' -f4)
```

## readReport

```bash
cat /tmp/delegate_final.txt
```

## resumeSession

```bash
codex exec resume {thread_id} --json -o /tmp/delegate_final.txt -- <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[delta prompt]
PROMPT
```
