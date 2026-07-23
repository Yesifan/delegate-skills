# delegate-skills

[![skills.sh](https://skills.sh/b/Yesifan/delegate-skills)](https://skills.sh/Yesifan/delegate-skills)

Skills for **delegating coding work to a separate CLI agent and landing it yourself**. Your agent (the
orchestrator) writes a self-contained brief, hands it to an implementer CLI, then reviews the diff and
commits — staying the reviewer the whole way.

One skill ships today: **`external-implementer`** — delegate a bounded coding task to OpenCode CLI, OpenCode Server, or Codex CLI, then review and land it yourself.

## Progressive-disclosure pattern

This skill follows **progressive disclosure**: the `SKILL.md` keeps a lean loop outline with
context pointers, and depth (prompt template, dispatch mechanics, review checklist, multi-task
sequencing) loads only when needed from `references/` files. No relay script, no brief.txt,
no `result.json` — the orchestrator constructs a brief inline and pipes it directly to the
chosen implementer CLI.

## Install

Browse first:

```bash
npx skills add Yesifan/delegate-skills --list
```

Install the package, or just the skill:

```bash
npx skills add Yesifan/delegate-skills
npx skills add Yesifan/delegate-skills --skill external-implementer
```

Install for a specific agent, or globally:

```bash
npx skills add Yesifan/delegate-skills --skill external-implementer --agent claude-code
npx skills add Yesifan/delegate-skills --global
```

Works with any orchestrating agent the [Skills CLI](https://github.com/vercel-labs/skills) supports.

## What it does

### The delegation loop

The skill follows a progressive-disclosure loop — no relay, no brief.txt, no result.json:

1. **Select a task group** from what needs implementing.
2. **Construct a brief** from project facts and your supplementary context.
3. **Pipe it directly** to the implementer (CLI heredoc or Server API), capturing session info.
4. **Extract the session ID** from the dispatch output.
5. **Review** the diff and re-run the project's gates yourself.
6. **Land** it — _you_ commit.

The brief template, dispatch mechanics, and per-CLI operations are in
`references/` files loaded only when needed.

```text
Use $external-implementer to delegate task group 2 to Codex (or OpenCode), then review and commit.
Use $external-implementer to delegate task group 3 to OpenCode, then review and commit.
```



## The skills

### external-implementer

Drive OpenCode CLI, OpenCode Server, or Codex CLI as the implementer, chosen at invocation. No
relay script, no brief.txt, no result.json: the orchestrator constructs a brief inline, pipes it
directly to the chosen CLI or Server API, and captures the output. Per-implementer commands
(dispatch, session/thread extraction, resume) live in dedicated reference files. Prompt depth
lives in `references/` files loaded on demand.

**You'll feel it when:** you have a bounded task to delegate to OpenCode CLI, OpenCode Server, or
Codex — the skill asks once, then you're in the loop.



## Requirements

- The chosen implementer:
  - **OpenCode CLI** or **Codex CLI**: CLI installed and authenticated.
  - **OpenCode Server**: running `opencode serve` reachable at the configured host:port. Auth via `OPENCODE_SERVER_PASSWORD` (HTTP Basic Auth).
- `git` for `git status`/`git diff` during review.
- An orchestrating agent that can run shell commands and read files.
- Shell examples assume bash/zsh (macOS/Linux, or Git Bash/WSL on Windows).

## Orchestrator permissions

Some orchestrator environments restrict child-process access. When delegating,
the implementer needs:

| Resource | CLI mode | Server mode |
|---|---|---|
| Working tree | Read source, write code, run project gates | Same |
| `/tmp/` or system temp dir | Dispatch artifacts (`.jsonl`, `.log`) | Not needed |
| `~/.local/share/opencode/` (Linux) / `%LOCALAPPDATA%/opencode/` (Windows) | OpenCode log persistence | Same |
| Server host:port | — | HTTP access to `opencode serve` |
| `localhost` ports | Dev servers, databases, test infrastructure | Same |

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
- `external-implementer` has **no scripts** — the orchestrator pipes prompts directly to the implementer
  CLI. There is nothing to inspect beyond the skill text itself.
- No skill ever commits — committing is always the orchestrator's job, after review.

## Repository shape

```text
skills/
└── external-implementer/
    ├── SKILL.md
    └── references/
        ├── writing-the-brief.md
        ├── opencode-operations.md
        ├── opencode-server-operations.md
        └── codex-operations.md
```

The skill uses **progressive disclosure**: the `SKILL.md` keeps the loop outline and context
pointers, with depth in `references/` files that load only when needed.

## License

MIT — see [LICENSE](LICENSE).
