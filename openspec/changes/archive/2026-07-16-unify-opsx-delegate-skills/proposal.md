## Why

`oc-delegate-opsx` (OpenCode) and `cx-delegate-opsx` (Codex) share an identical flow structure but duplicate almost every line with agent-specific wording. Every edit, fix, or enhancement must touch two skill directories. Worse, the duplication invites drift — the two skills have already diverged in review depth, brief strategy, and dispatch approach. Merging into one skill eliminates the maintenance burden and removes the cognitive cost of choosing which skill to invoke.

## What Changes

- Merge `oc-delegate-opsx` and `cx-delegate-opsx` into a single `opsx-implementer` skill
- Add a "choose implementer" step at the skill entry where the orchestrator asks the user for OpenCode or Codex
- Extract CLI-specific commands into two pure-command-reference files: `opencode-operations.md` and `codex-operations.md`
- Retain `dispatch-and-poll.md` as the shared dispatch flow, pointing to named commands in the operations files
- Retain `writing-the-brief.md`, `review-and-land.md`, `multi-task-queues.md` as shared references
- Remove the old `oc-delegate-opsx/` and `cx-delegate-opsx/` directories
- Update `skills.sh.json`, `AGENTS.md`, and `README.md` to reflect the single skill
- No cost tracking in the flow (no `extractCost`); CLI unavailability only reported, never auto-installed

## Capabilities

### New Capabilities
- `opsx-implementer`: Unified OpenSpec delegation skill supporting both OpenCode and Codex as the implementer, selected by the user at entry

### Modified Capabilities
<!-- No existing spec requirements change — this is purely a restructuring and merge. -->

## Impact

- Two skill directories removed, one created
- Four reference files restructured: dispatch-and-poll.md retained, writing-the-brief.md unified, opencode-operations.md and codex-operations.md created
- `skills.sh.json` registers one skill instead of two
- README and AGENTS.md documentation updated
- No API, dependency, or build system changes
