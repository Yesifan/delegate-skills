# Multi-task queues

Default to **parallel**; serialize only where updates overlap.

## Dependency-first, then parallel

Implement shared dependencies first (common interface, shared utility), then fan out independent
groups in parallel. Parallel groups must touch **non-overlapping files** — if two groups both modify
`src/auth/middleware.ts`, run them serially.

## Per-group: quick check

After each group: any blockers? gates pass? surface-level scan ok? Catch obvious failures early.

## Close: full review

After all groups complete: run full test/lint/build suite, do a repo-wide sweep, verify
cross-group integrity, and walk every changed file. Land only after this passes.

## Carry constraints forward

When a later group depends on a name or decision that emerged during implementation, **restate it
explicitly** in that group's prompt — the implementer has no memory of prior runs.
