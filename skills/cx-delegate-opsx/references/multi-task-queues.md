# Multi-task queues

A change usually has several task groups. The single-group loop scales to the queue, and the
discipline that makes it trustworthy is sequencing and bookkeeping — not parallelism.

## Run sequentially, one commit per group

Resist fanning out the whole queue at once. Run groups **one at a time, in dependency order**,
landing each (review + gates + commit) before dispatching the next. Three reasons:

- **Later groups assume earlier ones landed.** Group 3's brief can say "the X added in the previous
  step exists" only if the previous step actually committed.
- **One commit per group** keeps history reviewable and any single group revertible.
- **Each review is honest.** A clean working tree before each dispatch means the next group's diff
  shows only *its* changes, not a pile-up from earlier groups.

## Carry constraints forward

Implementation surfaces facts the original plan didn't have — a helper name, a fixture location, an
interface choice. When a later group depends on one of those, **restate it explicitly in that
group's prompt**. The sub-agent has no memory of prior runs, so a constraint that emerged in group 2
must be restated in group 5's prompt or it won't hold. This is the queue equivalent of keeping briefs
self-contained.

## Parallel when independent

Groups on separate files with no dependency can run in parallel, but you sacrifice a clean working
tree per task — review gets harder because each group's diff is entangled with the others. Default to
sequential; reach for parallel only when the independence is real and the groups are large enough
that the wall-clock saving is worth the review cost.

## Close with a coherence check

Per-group review proves each step in isolation; it doesn't prove the steps cohere. After the last
task group, do not assume each passing slice adds up to a passing whole:

- Run the full test/build suite on the final tree — not just each task's slice.
- Do a repo-wide sweep for the change's concern (after a removal, grep for surviving references;
  after a rename, confirm no stragglers).
- For schema work, replay all migrations from a clean state and check for drift.

## Interleaving with OpenSpec's lifecycle

This skill interleaves with OpenSpec's propose → apply → archive flow:

- **After propose**: artifacts exist but no code. Delegate task groups one by one.
- **After partial implementation**: delegate remaining task groups; resume sessions for rework.
- **After all tasks done**: sync delta specs to main specs (`openspec-sync-specs`), then archive.

Each delegation step is itself an *apply* step inside that flow; the landed commits are what you
hand back to OpenSpec's archiving. You don't run the OpenSpec lifecycle here — you slot into it.