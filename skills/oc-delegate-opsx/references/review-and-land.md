# Review and land

## Read the report first

Read the implementer's task summary. Extract it via the `jq` command in [dispatch-and-poll.md](dispatch-and-poll.md).

## 1 — Check the task summary

Verify nothing was missed or misaligned against the original brief.

## 3 — Implementer sweep

Review the implementer's changes against the project's code conventions.

## Surface, don't absorb

Report design decisions the implementer made, note non-blocking nitpicks, and stop if correct
completion requires going beyond the spec — scope changes are the human's call. The OpenSpec brief
defines a boundary; whatever happens between start and result is disposable, but the result still has
to land inside that boundary.

## Land or rework

- If the work is correct and gates pass, commit it.
- If it needs changes, resume the session with a delta prompt — don't restate the whole task:

  ```bash
  opencode run --session {session_id} [--model {provider/model}] -- <<'PROMPT' \
    > /tmp/delegate_{capability}_{task-id}.jsonl \
    2>/tmp/delegate_{capability}_{task-id}.log
  [delta prompt — only the changes needed]
  PROMPT
  ```

  `--session` keeps the implementer's context from the first run (and its model), so a short delta
  is enough. Then review again — rework gets the same test check, gate re-run, diff-read, and sweep
  as the original, no shortcuts. Repeat until it's right, then commit.

- If more task groups remain, return to step 1 (see [multi-task-queues.md](multi-task-queues.md)).
