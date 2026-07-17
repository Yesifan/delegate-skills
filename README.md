# delegate-skills

[![skills.sh](https://skills.sh/b/Yesifan/delegate-skills)](https://skills.sh/Yesifan/delegate-skills)

Skills for **delegating coding work to a separate CLI agent and landing it yourself**. Your agent (the
orchestrator) writes a self-contained brief, hands it to an implementer CLI, then reviews the diff and
commits — staying the reviewer the whole way.

One skill ships today: **`opsx-implementer`** — delegate OpenSpec task groups to OpenCode or Codex.

## OpenSpec-native pattern

This skill uses an [OpenSpec](https://github.com/fission-ai/openspec)-native approach
that follows **progressive disclosure**: the `SKILL.md` keeps a lean loop outline with context
pointers, and depth (prompt template, dispatch mechanics, review checklist, multi-task
sequencing) loads only when needed from `references/` files. No relay script, no brief.txt,
no `result.json` — the orchestrator reads specs from the change directory, constructs a
prompt inline, and pipes it directly to the chosen implementer CLI.

The implementer writes results back to the working tree and marks tasks done; the orchestrator
reviews the diff, handles design.md and spec updates after review, and commits.

## Install

Browse first:

```bash
npx skills add Yesifan/delegate-skills --list
```

Install the package, or just the skill:

```bash
npx skills add Yesifan/delegate-skills
npx skills add Yesifan/delegate-skills --skill opsx-implementer
```

Install for a specific agent, or globally:

```bash
npx skills add Yesifan/delegate-skills --skill opsx-implementer --agent claude-code
npx skills add Yesifan/delegate-skills --global
```

Works with any orchestrating agent the [Skills CLI](https://github.com/vercel-labs/skills) supports.

## What it does

### The delegation loop

The skill follows a progressive-disclosure loop — no relay, no brief.txt, no result.json:

1. **Select a task group** from an OpenSpec change's `tasks.md`.
2. **Construct a prompt** from the change's specs, design, and your supplementary context.
3. **Pipe it directly** to the implementer CLI, capturing the JSON event stream.
4. **Extract the session ID** from the output and record it in `tasks.md`.
5. **Review** the diff and re-run the project's gates yourself.
6. **Land** it — _you_ commit.

The prompt template, dispatch mechanics, review checklist, and multi-task sequencing are in
`references/` files loaded only when needed.

```text
Use $opsx-implementer to delegate task group 2 from the 'add-auth' change to Codex (or OpenCode), then review and commit.
Use $opsx-implementer to delegate task group 3 from the 'add-auth' change to OpenCode, then review and commit.
```



## The skills

### opsx-implementer

Drive either the OpenCode or Codex CLI as an OpenSpec-aware implementer, chosen at invocation.
No relay script, no brief.txt, no result.json: the orchestrator reads the spec and design from the
change directory, constructs a prompt inline, pipes it directly to the chosen CLI, and captures
the output. CLI-specific commands (dispatch, session/thread extraction, resume) live in dedicated
reference files. Prompt depth lives in `references/` files loaded on demand.

**You'll feel it when:** you have an OpenSpec change and want to delegate a task group to either
OpenCode or Codex — the skill asks once, then you're in the loop.



### gemini-delegate

_Planned._ A delegate skill for the Gemini CLI, if and when it gains a comparable non-interactive mode.
Reserved so the umbrella can grow without a rename.

## Requirements

- The [`openspec` CLI](https://github.com/fission-ai/openspec) installed and an OpenSpec change
  created (`openspec new change <name>`).
- The chosen implementer CLI (`opencode` or `codex`) installed and authenticated.
- `git` for `git status`/`git diff` during review.
- An orchestrating agent that can run shell commands and read files.
- Shell examples assume bash/zsh (macOS/Linux, or Git Bash/WSL on Windows).

## Orchestrator permissions

Some orchestrator environments restrict child-process access. When delegating,
the implementer needs:

| Resource | Why |
|---|---|
| Working tree | Read source, write code, run project gates |
| `/tmp/` or system temp dir | Dispatch artifacts (`.jsonl`, `.log`) |
| `~/.local/share/opencode/` (Linux) / `%LOCALAPPDATA%/opencode/` (Windows) | OpenCode log persistence |
| `localhost` ports | Dev servers, databases, test infrastructure |

If a dispatch is blocked by filesystem or network errors:

1. **`--pure`** — append to the dispatch command. Skips plugin loading,
   including the logger that writes the log path above.
2. **Redirect log path** — set the env var OpenCode uses (`opencode --help`
   shows it) to a path inside the workspace.
3. **Escalate permissions** — allow the blocked path or port in your
   orchestrator's permission model.

## Trust and validation

This skill is intentionally inspectable:

- All skill content is Markdown.
- `opsx-implementer` has **no scripts** — the orchestrator pipes prompts directly to the implementer
  CLI. There is nothing to inspect beyond the skill text itself.
- No skill ever commits — committing is always the orchestrator's job, after review.

**Verification status:** the prompt templates have been smoke-tested but not yet exercised in a full
delegation run. The full delegate → review → commit loop is designed for and run on Claude Code but
not yet formally verified end-to-end here. This line gets upgraded to "verified end-to-end" with
evidence, not assumption.

## Repository shape

```text
skills/
└── opsx-implementer/
    ├── SKILL.md
    └── references/
        ├── writing-the-brief.md
        ├── dispatch-and-poll.md
        ├── review-and-land.md
        ├── multi-task-queues.md
        ├── opencode-operations.md
        └── codex-operations.md
```

The skill uses **progressive disclosure**: the `SKILL.md` keeps the loop outline and context
pointers, with depth in `references/` files that load only when needed.

## License

MIT — see [LICENSE](LICENSE).
