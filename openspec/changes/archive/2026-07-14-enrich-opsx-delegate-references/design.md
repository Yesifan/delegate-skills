## Context

Both opsx delegate skills (`cx-delegate-opsx`, `oc-delegate-opsx`) are self-contained SKILL.md files with no separate references. They currently have 5 review checklist items and a minimal prompt template. The original delegate skills (`codex-delegate`, `opencode-delegate`) ship 4 reference files with deep content — notably review-and-land.md's 11 failure pattern sweep and writing-the-brief.md's structured prompt blocks — that the opsx skills lack. This change ports the valuable patterns inline without creating a references/ directory.

## Goals / Non-Goals

**Goals:**
- Enrich prompt template with verification_loop, action_safety, structured_output_contract blocks
- Expand review checklist with the implementer sweep (11 failure patterns gates miss)
- Add troubleshooting guidance for failed runs
- Add constraint forwarding and final coherence check for multi-task sequencing
- All content inline in SKILL.md

**Non-Goals:**
- No new files or directories (no references/)
- No changes to original codex-delegate or opencode-delegate skills
- Not restructuring the delegation loop itself

## Decisions

### Decision 1: All new content inline, no separate references

Content is added directly to SKILL.md sections, not extracted to a references/ directory. The review section expands from 5 lines to ~25 lines; the prompt template adds 3 optional blocks.

### Decision 2: Prompt template adds three XML blocks

The prompt template (Step 2) gains optional blocks the orchestrator fills in:

| Block | When to include | Content |
|-------|----------------|---------|
| `<verification_loop>` | Every implementation task | Actual gate commands discovered from repo |
| `<action_safety>` | Every task | Don't commit, don't scope creep |
| `<structured_output_contract>` | Every task | Report format: what changed, files, gates, deviations |

### Decision 3: Review step restructured into three layers

Layer 1: Test integrity check — did sub-agent weaken test assertions before making them pass?
Layer 2: Gate re-run — orchestrator runs the real commands, doesn't trust self-report
Layer 3: Implementer sweep — 11 patterns gates are blind to

### Decision 4: Troubleshooting and multi-task guidance as reference sections

"Troubleshooting" and "Running multiple task groups" sit as in-skill reference sections after the delegation loop, retaining the self-contained property while pushing depth below the primary steps.

## Risks / Trade-offs

- **[Risk] SKILL.md length increases** — each skill grows ~100-150 lines. **Mitigation**: No sprawl — the added lines are genuine step detail, not sedimen
- **[Risk] Prompt complexity** — orchestrator must now craft a richer prompt. **Mitigation**: The template shows the blocks; orchestrator can omit blocks for simple tasks
- **[Risk] Sweep detail easy to skim** — 11 patterns is a lot to check. **Mitigation**: Grouped by category (data integrity, code quality, test quality)
