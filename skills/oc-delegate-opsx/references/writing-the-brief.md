# Writing the brief

A brief is the entire task as the OpenCode implementer will see it. OpenCode runs in a fresh session
with **no memory of your conversation, no access to your prior notes, and no shared context** — only
the text you send and whatever it can read from the working tree. If a constraint isn't in the brief
or discoverable in the repo, it doesn't exist for OpenCode. The single most common failure is a
brief that assumes context OpenCode doesn't have — and in the opsx pattern that includes the
OpenSpec spec/design/tasks you read to build it: fold them in, don't point at them.

## Source the brief from the OpenSpec change

Read the change's artifacts and embed the load-bearing parts **directly** into the brief:

1. **`tasks.md`** — pick one task group whose tasks are independent of other groups. Record the
   capability name from `openspec status` or the spec directory name; it's how you key the output
   file and the `tasks.md` back-link.
2. **The spec for the relevant capability** — embed the spec content directly into the prompt so the
   implementer doesn't need to switch context.
3. **`design.md`** — embed the relevant sections (the design holds judgements the spec can't carry:
   interface choices, sequencing, what was rejected and why).
4. **Supplementary context that is *not* in the specs** — file placement, naming, conventions the
   implementer can't infer from the spec or the repo's `AGENTS.md`.

## Match the model to the brief

OpenCode allows the model to be empty — it uses its configured default — so you only need `--model`
when you want to override the default for a specific task. When you override, it's a two-owner
decision: the **human** owns which models are allowed; **you, the orchestrator**, pick one of them
to fit the task.

- **The allowed set is the human's to state.** `opencode models` lists hundreds of entries; only the
  human knows which are their flat-rate subscriptions. Ideally they state it once in the repo's
  `AGENTS.md` or `CLAUDE.md`.
- **Read the task's difficulty off the brief you just wrote, and match within that set.** A
  mechanical, well-bounded brief (a rename sweep, a removal) is safe on a cheap, fast model; a brief
  whose risk lives in judgement (concurrency, money/auth path, an ambiguous spec) wants a strong one.
- **If no usable set is stated, ask — don't guess.** Naming the constraint to the human and letting
  them choose beats a metered bill.
- **A resumed run inherits its session's model.** Send only the delta prompt — the session retains
  its model from the original run.

## The shape that works

OpenCode responds well to compact, block-structured prompts with XML tags. Include all four blocks.

```xml
<role>
You are implementing a task group from an OpenSpec change. Work through each task one at a
time. After completing each task, edit `openspec/changes/{name}/tasks.md` to mark it done:
change `- [ ] {task-id}` to `- [x] {task-id}`.
</role>

<task>
Task {id} from OpenSpec change '{name}'. Capability: {capability}.
Change root: openspec/changes/{name}/

Spec requirements:
{spec content — embedded directly}

Design context:
{relevant design sections}

Supplementary instructions (not in specs):
{your supplementary context — file placement, naming, conventions}

What to leave untouched:
{files or areas the implementer must not modify}
</task>

<verification_loop>
Run these before finishing and fix anything they surface, don't just report it:
  {project test command}
  {project lint command}
  {project build/typecheck command}
Confirm the working tree shows only the intended changes afterward.
</verification_loop>

<action_safety>
Keep changes scoped to the task. No unrelated refactors, renames, or cleanup unless
required for correctness. Do NOT run git add or git commit — the orchestrator commits
after reviewing. Leave the work uncommitted in the working tree.
If requirements are ambiguous or you hit a blocker, stop and report it in your
structured output — do NOT guess, work around silently, or change scope without
explicit justification.
</action_safety>

<headless_environment>
You run headless — no one can answer interactive prompts. For every CLI command:
  - Use non-interactive flags: --yes, -y, --force, --no-interactive
  - If a tool lacks one, prefix: `yes | <command>`
  - Package managers: CI=true pnpm install (or equivalent)
  - DB tools: drizzle-kit push --force, prisma migrate dev --yes
Network-bound operations (DB push, API calls) default to localhost. If they
fail with EPERM/EACCES, a sandbox network boundary is blocking them — report
it in structured output, do NOT attempt workarounds.
</headless_environment>

<structured_output_contract>
End with a report in this exact shape:
  1. What changed and why
  2. Files touched
  3. Gate outcomes (paste the test/lint counts)
  4. Anything you deviated on, left open, or want a decision on
     (including implementation decisions, spec adjustments, or design issues.
     Do NOT edit design.md or spec files — report here instead)
</structured_output_contract>
```

## Discover the real gates — don't hardcode

`<verification_loop>` is only useful if it names the project's *actual* commands. Discover them from
the repo's `CLAUDE.md`, `AGENTS.md`, `Makefile`, or `package.json` — do not assume or hardcode — and
embed them in the block. A brief that says "run the tests" without naming them gets you an implementer
that guesses, or skips.

## One task group per brief

Keep each brief to a single, bounded task group. One brief → one OpenCode run → one commit keeps
review and rollback clean and lets a later group assume the earlier one landed. If multiple groups
have no dependency, you can run them in parallel — see [multi-task-queues.md](multi-task-queues.md).

Send this brief (see [dispatch-and-poll.md](dispatch-and-poll.md)); review the result and land it
yourself (see [review-and-land.md](review-and-land.md)).