# delegate-skills

[![skills.sh](https://skills.sh/b/Yesifan/delegate-skills)](https://skills.sh/Yesifan/delegate-skills)

Skills for **delegating coding work to a separate CLI agent and landing it yourself**. Your agent (the
orchestrator) writes a self-contained brief, hands it to an implementer CLI, then reviews the diff and
commits — staying the reviewer the whole way.

Four skills ship today:

| Skill                   | Implementer      | Pattern                                        |
| ----------------------- | ---------------- | ---------------------------------------------- |
| **`codex-delegate`**    | OpenAI Codex CLI | Brief → relay.mjs → result.json                |
| **`opencode-delegate`** | OpenCode CLI     | Brief → relay.mjs → result.json                |
| **`cx-delegate-opsx`**  | OpenAI Codex CLI | OpenSpec spec → direct exec → update artifacts |
| **`oc-delegate-opsx`**  | OpenCode CLI     | OpenSpec spec → direct exec → update artifacts |

## OpenSpec-native pattern

The **-opsx** skills use an [OpenSpec](https://github.com/fission-ai/openspec)-native approach
that follows **progressive disclosure**: the `SKILL.md` keeps a lean loop outline with context
pointers, and depth (prompt template, dispatch mechanics, review checklist, multi-task
sequencing) loads only when needed from `references/` files. No relay script, no brief.txt,
no `result.json` — the orchestrator reads specs from the change directory, constructs a
prompt inline, and pipes it directly to the implementer CLI.

The implementer writes results back to the working tree and marks tasks done; the orchestrator
reviews the diff, handles design.md and spec updates after review, and commits.

## Install

Browse first:

```bash
npx skills add Yesifan/delegate-skills --list
```

Install the package, or just one skill:

```bash
npx skills add Yesifan/delegate-skills
npx skills add Yesifan/delegate-skills --skill codex-delegate
npx skills add Yesifan/delegate-skills --skill opencode-delegate
```

Install for a specific agent, or globally:

```bash
npx skills add Yesifan/delegate-skills --skill codex-delegate --agent claude-code
npx skills add Yesifan/delegate-skills --global
```

Works with any orchestrating agent the [Skills CLI](https://github.com/vercel-labs/skills) supports.

## What it does

### OpenSpec-native loop (opsx skills)

The opsx skills follow a progressive-disclosure loop — no relay, no brief.txt, no result.json:

1. **Select a task group** from an OpenSpec change's `tasks.md`.
2. **Construct a prompt** from the change's specs, design, and your supplementary context.
3. **Pipe it directly** to the implementer CLI, capturing the JSON event stream.
4. **Extract the session ID** from the output and record it in `tasks.md`.
5. **Review** the diff and re-run the project's gates yourself.
6. **Land** it — _you_ commit.

The prompt template, dispatch mechanics, review checklist, and multi-task sequencing are in
`references/` files loaded only when needed.

### Brief-based loop (relay skills)

The original `codex-delegate` and `opencode-delegate` use the relay-based loop:

1. **Write a brief** — a self-contained task spec; the implementer sees only what you send.
2. **Dispatch** it with the bundled `relay.mjs`.
3. **Wait** for completion — the helper writes a structured `result.json`.
4. **Review** the diff — re-run the project's gates yourself; pair with [guard skills](https://github.com/amElnagdy/guard-skills).
5. **Land** it — _you_ commit, because committing belongs to the reviewer.

```text
Use $cx-delegate-opsx to delegate task group 2 from the 'add-auth' change to Codex, then review and commit.
Use $oc-delegate-opsx to delegate task group 3 from the 'add-auth' change to OpenCode, then review and commit.
Use $codex-delegate to have Codex implement the refactor in services/billing/, then review and commit it.
Use $opencode-delegate to run this queue of migration tasks through OpenCode while I review each one.
```

## How this differs from the OpenAI Codex plugin

The official openai-codex Claude Code plugin is excellent and
**complementary** — this skill builds on the same `codex` CLI, it doesn't replace the plugin. They
point in different directions:

- The plugin's `codex:codex-rescue` agent is a **forwarder**: it hands one task to Codex and returns
  the output. It deliberately does not poll, review, or commit.
- The plugin's review command and stop-review gate run the **inverse** direction: **Codex reviews your work**.
- `codex-delegate` is the **orchestration loop in the other direction**: _you_ drive Codex to
  implement across one task or a queue, and _you_ review and land each result. That loop — brief →
  dispatch → poll → review → commit, with the orchestrator owning the commit — is what the plugin
  leaves to you, and what this skill encodes.

If you have the plugin installed, its companion CLI is an optional alternative dispatch backend; the
bundled `relay.mjs` is the default because it needs nothing but the `codex` binary.

## The skills

### cx-delegate-opsx

Drive the OpenAI Codex CLI as an OpenSpec-aware implementer. No relay script, no brief.txt, no
result.json: the orchestrator reads the spec and design from the change directory, constructs a
prompt inline, and pipes it directly to `codex exec`. The sub-agent reads the spec from the prompt,
implements, and marks tasks done. The orchestrator captures the thread ID from stderr and records it
in tasks.md for resume. Prompt depth lives in `references/` files loaded on demand.

**You'll feel it when:** you have an OpenSpec change with specs and tasks, and you delegate a task
group to Codex by writing a prompt — no boilerplate, no intermediate files.

### oc-delegate-opsx

Drive the OpenCode CLI as an OpenSpec-aware implementer. Same pattern as cx-delegate-opsx but for
`opencode run`: the orchestrator constructs a prompt from the change artifacts, pipes it with
`--format json`, and captures the JSON event stream to `/tmp/delegate_{cap}_{task}.jsonl`. Supports
build and plan agents for write and explore modes. Prompt depth lives in `references/` files loaded
on demand.

**You'll feel it when:** you have an OpenSpec change and delegate a task group to OpenCode —
just a prompt, a pipe, and a review.

### codex-delegate

Drive the OpenAI Codex CLI as a background implementer: write the brief, dispatch via `relay.mjs`,
review the diff, commit it yourself. Ships four references (writing the brief, dispatch/poll, review/
land, multi-task queues) loaded only when needed, and one small helper script.

**You'll feel it when:** a bounded task — a migration, a mechanical refactor, a removal sweep — gets
handed to Codex, comes back as a clean diff with a structured report, and you commit it after re-running
the gates yourself instead of typing it all by hand.

### opencode-delegate

Drive the OpenCode CLI as a background implementer: write the brief, dispatch via `relay.mjs`, review
the diff, commit it yourself. Same four references and loop as `codex-delegate`. Autonomy is set by the
**agent** rather than a sandbox enum — `build` (write-capable) by default, `plan` (read-only) for
review/diagnosis — and the brief is piped to `opencode run` on stdin so multi-line XML briefs need no
quoting.

**You'll feel it when:** a bounded task gets handed to OpenCode, comes back as a clean diff with a
structured report and the run's cost, and you commit it after re-running the gates yourself.

### gemini-delegate

_Planned._ A delegate skill for the Gemini CLI, if and when it gains a comparable non-interactive mode.
Reserved so the umbrella can grow without a rename.

## Requirements

- For `codex-delegate` and `cx-delegate-opsx`: the [`codex` CLI](https://github.com/openai/codex)
  installed and authenticated (`codex login`).
- For `opencode-delegate` and `oc-delegate-opsx`: the [`opencode` CLI](https://opencode.ai)
  installed and authenticated (`opencode auth login`).
- For `cx-delegate-opsx` and `oc-delegate-opsx`: the [`openspec` CLI](https://github.com/fission-ai/openspec)
  installed and an OpenSpec change created (`openspec new change <name>`).
- Node 18+ and `git`.
- An orchestrating agent that can run shell commands and read files.
- Shell examples assume bash/zsh (macOS/Linux, or Git Bash/WSL on Windows).

## Trust and validation

This package is intentionally inspectable:

- All skill content is Markdown.
- `codex-delegate` and `opencode-delegate` each have a `scripts/relay.mjs` (Node built-ins only,
  no network calls, no credentials, no telemetry). Read it before you run it.
- `cx-delegate-opsx` and `oc-delegate-opsx` have **no scripts** — the orchestrator pipes prompts
  directly to the implementer CLI. There is nothing to inspect beyond the skill text itself.
- No skill ever commits — committing is always the orchestrator's job, after review.

**Verification status:** the original delegate skills' relay mechanics are verified — argument handling,
exit codes, `result.json`, resume, and (for `opencode-delegate`) the required-model guard. The -opsx
skills' prompt templates have been smoke-tested but not yet exercised in a full delegation run. The full
delegate → review → commit loop is designed for and run on Claude Code but not yet formally verified
end-to-end here. This line gets upgraded to "verified end-to-end" with evidence, not assumption.

## Repository shape

```text
skills/
├── codex-delegate/
│   ├── SKILL.md
│   ├── scripts/relay.mjs
│   └── references/
│       ├── writing-the-brief.md
│       ├── dispatch-and-poll.md
│       ├── review-and-land.md
│       └── multi-task-queues.md
├── cx-delegate-opsx/
│   ├── SKILL.md
│   └── references/
│       ├── writing-the-brief.md
│       ├── dispatch-and-poll.md
│       ├── review-and-land.md
│       ├── multi-task-queues.md
│       └── exploring-the-codebase.md
├── opencode-delegate/
│   ├── SKILL.md
│   ├── scripts/relay.mjs
│   └── references/
│       ├── writing-the-brief.md
│       ├── dispatch-and-poll.md
│       ├── review-and-land.md
│       └── multi-task-queues.md
└── oc-delegate-opsx/
    ├── SKILL.md
    └── references/
        ├── writing-the-brief.md
        ├── dispatch-and-poll.md
        ├── review-and-land.md
        ├── multi-task-queues.md
        └── exploring-the-codebase.md
```

The opsx skills use **progressive disclosure**: the `SKILL.md` keeps the loop outline and context
pointers, with depth in `references/` files that load only when needed. The original relay skills
keep the relay + references structure for backward compatibility.

## License

MIT — see [LICENSE](LICENSE).
