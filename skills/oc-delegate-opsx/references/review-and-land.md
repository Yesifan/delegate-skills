# Review and land

The implementer did the typing; you own the judgement. The discipline is simple to state and easy to
skip under time pressure: **verify against reality, never against the self-report — and read the diff
as generated code, which fails in ways a green gate can't see.**

## Read the report first

Before the three layers below, read the implementer's final report from the captured output. The
`<structured_output_contract>` in the prompt asked for a summary in a fixed shape (what changed,
files touched, gate outcomes, deviations). Extract the last `type: "text"` event from the `.jsonl`
file (the `python3` command in [dispatch-and-poll.md](dispatch-and-poll.md) does this). Read it —
it tells you what the implementer claims. Then verify every claim against reality in the layers
that follow.

## Layer 1 — Check test integrity

If the diff touches existing tests, review those edits *first*, before re-running gates means
anything. A weakened assertion, an added skip, or a deleted test makes the gate measure less than it
did before the run; green is only meaningful if the yardstick wasn't shortened.

- **Unbriefed test edits are a contract change**, not part of the fix. Flag them.
- **Skipped, disabled, or commented-out tests** in the diff: treat the underlying test as failing
  until proven otherwise.
- **Loosened assertions** (exact → contains/truthy, error checks broadened, tolerance widened): same
  treatment.

## Layer 2 — Re-run the gates yourself

The implementer's claim that gates passed is a claim, not evidence. Re-run the project's actual
test/lint/build commands in the working tree and read the output. **Passing is necessary, not
sufficient** — an implementer can *game* a gate, not just misreport it. That's what layer 1 and
layer 3 exist to catch.

For changes with their own verification shape:

- **Migrations / schema:** round-trip them (apply, reverse, re-apply on a scratch target) and check
  for drift, rather than trusting that "the migration is reversible."
- **Removals / renames:** grep the codebase for dangling references to whatever was removed.
- **Anything stateful:** exercise the actual behavior, don't just confirm it compiles.

## Layer 3 — Implementer sweep

Generated code fails in systematic ways that gates are structurally blind to. Walk these patterns
against every changed file before you commit:

- **Hardcoded success** — a canned `{status: "ok"}` or default return on a path that should do
  real work.
- **Catch-all error handling** that swallows failures instead of propagating.
- **Unverified imports / API calls** — confirm every new dependency and method exists in the
  *installed* version (read the lockfile, don't trust plausibility).
- **Dead weight** — unused imports, unreachable branches, comment scaffolding.
- **Duplicate pattern** — a new HTTP client, error idiom, or logging style beside the existing one
  instead of reusing it.
- **Internals-testing** — tests that assert internal helpers were called, or mock the project's own
  functions.
- **Near-duplicate tests** — test bodies differing by one value; fold to one data-driven test or
  drop the copies.
- **Speculative surface** — optional params, config flags, or abstractions with no caller in the
  diff or the repo.
- **Impossible guards** — null/type checks for values the code's own contract already excludes.
- **Read the diff against the spec** — scope creep, scope shortfall, quiet judgement calls.

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