# Writing the brief

A brief is the task as the implementer sees it. The implementer runs in a fresh session with
**no memory of your conversation and no shared context** — only the text you send and whatever
it can read from the working tree. If a constraint isn't in the brief or discoverable in the
repo, it doesn't exist for the implementer.

## Source the brief from the OpenSpec change

Reference spec files by path (the implementer reads them from the working tree); embed curated
content (design sections, supplementary instructions) inline:

1. **`tasks.md`** — pick one task group (or subset, at your discretion). Record the capability
   name from `openspec status` or the spec directory name; it's how you key the output file and
   the `tasks.md` back-link.
2. **`design.md`** — embed the relevant sections (the design holds judgements the spec can't
   carry: interface choices, sequencing, what was rejected and why).
3. **Supplementary context not in the specs** — file placement, naming, conventions the
   implementer can't infer from the spec or the repo's `AGENTS.md`.

## The shape that works

Both OpenCode and Codex respond well to compact, block-structured prompts with XML tags.

```xml
<role>
You are implementing a task group from an OpenSpec change. Work through each task one at a
time.
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

<references>
Files in the working tree that are relevant to this task:
- Spec: openspec/changes/{name}/specs/{capability}/spec.md
- {other reference files}
</references>

<action_safety>
Never modify spec, design, or tasks files under the openspec change directory.
Do not run `git add` or `git commit`.
Scope all commands (lint, etc.) to the task — never touch files outside it.
If an out-of-scope module is missing, stop and report it in structured output.
If unclear or blocked, stop and report — do not guess, silently work around, or expand scope
without cause.
{other safety constraints}
</action_safety>

<headless_environment>
You run headless — no one can answer interactive prompts.
Use non-interactive flags for every CLI command: --yes, -y, --force, --no-interactive
Don't try to solve permission issues on your own. Report them immediately.
</headless_environment>

<structured_output_contract>
End with a report in this exact shape:
  - Task completion status and list of files changed per task (with line numbers if possible)
  - {any other info you need}
  - Anything you deviated on, left open, or want a decision on
    (including implementation decisions, spec adjustments, or design issues.
    Do NOT edit design.md or spec files — report here instead)

</structured_output_contract>
```
