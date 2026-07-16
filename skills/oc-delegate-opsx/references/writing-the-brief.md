# Writing the brief

A brief is the entire task as the OpenCode implementer will see it. OpenCode runs in a fresh session
with **no memory of your conversation, and no shared context** — only
the text you send and whatever it can read from the working tree. If a constraint isn't in the brief
or discoverable in the repo, it doesn't exist for OpenCode.

## Source the brief from the OpenSpec change

Reference spec files by path (the implementer reads them from the working tree); embed curated content (design sections, supplementary instructions) directly:

1. **`tasks.md`** — pick one task group (or subset, at your discretion). Record the
   capability name from `openspec status` or the spec directory name; it's how you key the output
   file and the `tasks.md` back-link.
2. **`design.md`** — embed the relevant sections (the design holds judgements the spec can't carry:
   interface choices, sequencing, what was rejected and why).
3. **Supplementary context that is _not_ in the specs** — file placement, naming, conventions the
   implementer can't infer from the spec or the repo's `AGENTS.md`.

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

<references>
Files in the working tree that are relevant to this task:
- Spec: openspec/changes/{name}/specs/{capability}/spec.md
- {other reference files}
</references>

<headless_environment>
You run headless — no one can answer interactive prompts.
Use non-interactive flags for every CLI command: --yes, -y, --force, --no-interactive
Don't try to solve any permission issues on your own. Report them immediately.
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
