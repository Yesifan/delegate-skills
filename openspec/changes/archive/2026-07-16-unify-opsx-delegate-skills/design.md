## Context

Two opsx delegate skills (`oc-delegate-opsx`, `cx-delegate-opsx`) share the same six-step delegation loop — select task group, write brief, dispatch, capture session/thread ID, review, land — but maintain separate SKILL.md and reference files for each CLI. The duplication is near-complete: the flow logic, review checklist, and multi-task guidance are identical; only the dispatch commands, session-capture patterns, and resume commands differ. Every bugfix or enhancement must be applied twice, and the two have already accumulated minor divergences (brief strategy, review depth).

## Goals / Non-Goals

**Goals:**
- Single `opsx-implementer` skill that handles both OpenCode and Codex implementers
- User chooses the implementer at skill entry; the choice persists for the entire delegation loop
- CLI-specific commands extracted into isolated, lookup-only reference files (`opencode-operations.md`, `codex-operations.md`)
- Dispatch flow (`dispatch-and-poll.md`) retained as shared flow logic, referencing named commands in the operations files
- Other shared references (`writing-the-brief.md`, `review-and-land.md`, `multi-task-queues.md`) remain agent-agnostic

**Non-Goals:**
- No new capabilities or flow changes beyond the merge
- No spec-level requirement changes (existing specs are archived unchanged)
- No cost tracking — `extractCost` is removed from the flow
- No auto-install of missing CLIs — errors are reported to the user

## Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Merge approach | Single skill, user-chooses at entry | Zero cognitive load on invocation — one skill to remember; the branching happens inside, after a single question |
| Command reference format | Named command snippets only, no flow context | Pure-command files are minimal and stable; flow logic lives in dispatch-and-poll.md where branching decisions are documented |
| Brief strategy | Path-based (implementer reads from working tree) for both CLIs | CX's `-C` flag and workspace-write sandbox provide working-tree access — no need to embed spec content. Eliminates a source of drift |
| Entry naming | Step 0: ask user, record in orchestrator context | No persistence mechanism needed; the orchestrator holds the choice for the loop's duration. `tasks.md` back-link records the CLI choice for resume persistence (e.g., `delegate: opencode {id}`) |
| Parallel dispatch | Follow existing multi-task-queues.md; same CLI throughout | User selects once; parallel groups use the same implementer. Mixed-CLI queues add complexity with no clear benefit |
| Cost tracking | Removed from flow | Neither OC nor CX provide reliable cost data in the same format; removing avoids inconsistent UX |
| CLI not found | Report and stop | The orchestrator cannot fix the user's environment. Auto-install is fragile and out of scope |

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Command reference files drift from actual CLI flags | Operation files are minimal and auditable; CLI changes are visible in one place per CLI |
| User answers "OpenCode" but meant "Codex" | Back-link in `tasks.md` records the actual CLI used; mismatches surface at resume |
| CX workspace-write sandbox may not provide full working-tree access in all versions | If it fails, the report-back mechanism in review step catches the gap; brief can fall back to embedding spec content as a documented alternative |
