## Why

Current delegate skills (`codex-delegate`, `opencode-delegate`) use a brief/result forwarding pattern: the orchestrator writes a free-form brief.txt, a relay script pipes it to the implementer CLI, and the implementer returns a result.json that's consumed once and discarded. This is ad-hoc, stateless, and wasteful — the implementer (Codex/OpenCode) is itself a capable agent with access to skills and CLI tools, yet it's treated as a blind text processor.

By replacing brief/result with OpenSpec artifacts, delegation becomes **spec-driven**: the implementer reads specs/design/tasks directly from the change directory, uses OpenSpec CLI and skills to understand context, and writes back by updating tasks.md and design.md. The orchestrator's job shifts from "write a brief, parse a result" to "delegate task groups, review the diff."

## What Changes

- Create two new skills: `cx-delegate-opsx` (Codex → OpenSpec) and `oc-delegate-opsx` (OpenCode → OpenSpec)
- No relay script: the orchestrator launches the implementer CLI directly, with a prompt that references OpenSpec artifacts
- The sub-agent reads specs and design from the OpenSpec change, implements, then updates tasks.md and design.md
- The orchestrator reviews the diff and tasks status, then delegates remaining task groups or archives
- Existing `codex-delegate` and `opencode-delegate` skills are **not modified**

## Capabilities

### New Capabilities
- `cx-delegate-opsx`: OpenSpec-aware Codex delegate. Orchestrator delegates task groups from an OpenSpec change to Codex CLI. Codex reads specs/design/tasks from the change directory, implements, and updates artifacts directly.
- `oc-delegate-opsx`: OpenSpec-aware OpenCode delegate. Same pattern but for OpenCode CLI. OpenCode reads/writes OpenSpec artifacts, supports `--model` selection, and uses `build`/`plan` agents.

### Modified Capabilities
*(none — new skills only, existing skills untouched)*

## Impact

- New directories: `skills/cx-delegate-opsx/` and `skills/oc-delegate-opsx/`
- Each contains: `SKILL.md` + optional `references/`
- No changes to existing `skills/codex-delegate/` or `skills/opencode-delegate/`
- README.md should be updated to mention the new skills
- No new dependencies — relies on the implementer's existing OpenSpec CLI access
