## Context

Current delegate skills (`codex-delegate`, `opencode-delegate`) use a relay script pattern: the orchestrator writes a brief.txt, the relay pipes it to the implementer CLI, and the implementer returns result.json. This treats the implementer (Codex/OpenCode) as a blind text processor despite it being a capable agent with access to skills and CLI tools.

The new `cx-delegate-opsx` and `oc-delegate-opsx` skills replace this with an **OpenSpec-native** delegation model: the implementer reads specs/design/tasks directly from the OpenSpec change directory, uses OpenSpec CLI and skills to understand context, and writes results back by updating artifacts. The orchestrator's job becomes: delegate task groups → review diff + artifacts.

## Goals / Non-Goals

**Goals:**
- Complete tasks efficiently while conserving orchestrator context and tokens
- Eliminate the relay script and brief/result forwarding layer
- Sub-agent reads specs from prompt (embedded from OpenSpec change) and writes back by updating artifacts
- Orchestrator delegates at task-group granularity, can parallelize independent groups
- Orchestrator can also do tasks directly when that's more efficient
- Orchestrator session is captured from CLI output, recorded in tasks.md for resume
- Sub-agent supports both build and explore modes
- Existing `codex-delegate` and `opencode-delegate` are untouched

**Non-Goals:**
- Not modifying the existing delegate skills or relay.mjs
- Not building a new CLI tool or wrapper
- Not changing how Codex/OpenCode CLI works internally
- Cost tracking is not a required feature

## Decisions

### Decision 1: No relay script — orchestrator launches implementer directly

The orchestrator constructs the full prompt inline and pipes it to the implementer CLI, redirecting output to a fixed-path file:

```bash
# Codex
codex exec -s workspace-write --json -o <final.txt> -- - <<'PROMPT' \
  2> /tmp/delegate_{capability}_{task-id}.jsonl
[prompt content]
PROMPT

# OpenCode
opencode run --agent build --model provider/model --format json --auto -- <<'PROMPT' \
  > /tmp/delegate_{capability}_{task-id}.jsonl 2>&1
[prompt content]
PROMPT
```

**Why**: The implementer (Codex/OpenCode) is itself a full agent. Adding a relay script between them adds complexity with no benefit — the orchestrator can construct the prompt and capture stdout itself.

**Session capture**: The orchestrator extracts the session ID from the captured output file after the run completes, not from sub-agent self-report. This is more reliable and removes the need for the sub-agent to cooperate on reporting.

Session ID extraction:
```bash
# OpenCode — sessionID field in JSON event stream
SESSION_ID=$(grep -o '"sessionID":"[^"]*"' /tmp/delegate_{cap}_{task}.jsonl | head -1 | cut -d'"' -f4)

# Codex — threadId in JSON events
THREAD_ID=$(grep -o '"thread_id":"[^"]*"' /tmp/delegate_{cap}_{task}.jsonl | head -1 | cut -d'"' -f4)
```

### Decision 2: Prompt contains OpenSpec context, not a free-form brief

The orchestrator constructs the prompt by reading artifacts from the change directory. The prompt contains the spec context directly (so the sub-agent doesn't need to switch context to read files), plus supplementary details not in the spec:

```
You are delegated to implement task {id} from OpenSpec change '{name}'.
Capability: {capability}
Change root: openspec/changes/{name}/

Spec requirements:
{spec content — embedded directly}

Design context:
{relevant design sections}

Supplementary instructions (not in specs):
{supplementary context from orchestrator, passed directly in CLI}

After implementation:
1. Mark tasks done: edit openspec/changes/{name}/tasks.md,
   change `- [ ] {task-id}` to `- [x] {task-id}`
2. Add implementation decisions to openspec/changes/{name}/design.md
3. If the spec needs adjustment, update openspec/changes/{name}/specs/{capability}/spec.md
```

**Why**: The specs are the source of truth for requirements. The orchestrator adds context that's too fine-grained for the spec (file placement, naming conventions, local patterns) directly in the CLI invocation, not in an intermediate file.

**File naming convention**: Every delegation run writes its output to `/tmp/delegate_{capability}_{task-id}.jsonl`. This fixed path lets the orchestrator know where to find session ID, error output, and completion status without any sub-agent cooperation.

### Decision 3: Sub-agent uses OpenSpec CLI/skills directly

The sub-agent has access to the OpenSpec CLI and the `openspec-apply` (or equivalent) skill. The prompt directs it to load and use those tools for reading task context.

**Why**: Leverages the sub-agent's full capability instead of treating it as a blind text processor. The sub-agent can run `openspec status`, `openspec instructions`, etc.

### Decision 4: Orchestrator delegates by task group, not entire change

The orchestrator reads `tasks.md`, picks one task group, and delegates that group only. After review, it delegates the next group. Multiple task groups can run in parallel if they have no dependencies.

**Why**: Follows the bounded-task principle from the existing delegate skills. Keeps each delegation run focused and reviewable.

### Decision 5: Orchestrator decides delegate vs do-it-yourself

The orchestrator evaluates each task group and decides whether to delegate or handle it directly:

| Condition | Tendency |
|-----------|----------|
| Simple, judgmental, or few-line change | **Do it yourself** — avoid delegation overhead |
| Complex implementation, multi-file, needs independent reasoning | **Delegate** — protect orchestrator context |
| Exploratory question | **Delegate to plan agent** — don't pollute orchestrator context |
| Code review of sub-agent's work | **Do it yourself** — orchestrator is the reviewer |

**Why**: The primary goal is completing tasks efficiently while conserving the orchestrator's context window and token budget. Sub-agents are a tool, not a requirement — the orchestrator should use them when they save resources, not as a default.

### Decision 6: tasks.md as coordination artifact

`tasks.md` serves double duty:
1. **Implementation checklist** — what needs to be done (standard OpenSpec)
2. **Delegation coordination record** — which sub-agent (and session) handled each task

When the orchestrator delegates a task, it appends `delegate: opencode <session-id>` (or `delegate: codex <thread-id>`) to the task line in `tasks.md`. This makes the session ID discoverable for resume without any additional file management.

```
Before:  - [ ] 2.1 Implement JWT middleware
After:   - [ ] 2.1 Implement JWT middleware  `delegate: codex ses_abc`
```

**Resume flow**: The orchestrator reads `tasks.md`, finds the session ID from a previous partial run, and passes it to the CLI:
```bash
# OpenCode resume
opencode run --session ses_abc -- <<'PROMPT' > /tmp/delegate_auth_2-1.jsonl 2>&1

# Codex resume
codex exec resume ses_abc --json -o <final.txt> -- - <<'PROMPT' \
  2> /tmp/delegate_auth_2-1.jsonl
```

### Decision 7: Sub-agent also runs explore tasks

The sub-agent supports two modes:
- **Build mode**: `codex exec -s workspace-write` / `opencode run --agent build` — write-capable implementation
- **Explore mode**: `codex exec -s read-only` / `opencode run --agent plan` — read-only exploration, no `--auto`

The orchestrator can delegate exploration tasks (e.g. "investigate whether library X supports feature Y") to a plan/read-only sub-agent during the OpenSpec explore phase, before any change is created.

**Why**: Keeps exploratory work out of the orchestrator's context while still leveraging a sub-agent's research capability.

## Risks / Trade-offs

- **[Risk] Orchestrator prompt construction is manual** — the orchestrator must read artifacts and build the prompt each time. **Mitigation**: The SKILL.md provides a prompt template the orchestrator fills in.
- **[Risk] No cost tracking** — without a relay script, cost capture is optional. **Mitigation**: Cost is not a required feature; if needed, the orchestrator can grep cost from the captured output file.
- **[Risk] Resume flow requires session file to exist** — if the output file is lost, resume is impossible. **Mitigation**: The session ID is also recorded in tasks.md (git-tracked), so it survives output file cleanup.
- **[Risk] Sub-agent must know OpenSpec file format** — it needs to update tasks.md checkboxes and design.md sections correctly. **Mitigation**: The prompt includes format examples and the sub-agent can read existing artifact files as reference.
