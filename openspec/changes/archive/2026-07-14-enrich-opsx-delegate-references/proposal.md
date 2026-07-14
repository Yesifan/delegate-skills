## Why

The opsx delegate skills (`cx-delegate-opsx`, `oc-delegate-opsx`) are lean but missing critical content that exists in the original delegate skills' reference files: a thorough review sweep (11 failure patterns gates are blind to), a richer prompt template (verification loop, action safety, structured output contract), troubleshooting guidance, and multi-task sequencing discipline. Without these, sub-agent output quality depends on the orchestrator rediscovering the same insights each run.

## What Changes

- Enrich prompt template in both skills: add `verification_loop`, `action_safety`, `structured_output_contract` blocks
- Expand review checklist: add implementer sweep (11 failure patterns) + test integrity check before gate re-run
- Add troubleshooting section: CLI binary check, output file diagnosis, common failure causes
- Add multi-task sequencing guidance: constraint forwarding, final coherence check
- All content inline in SKILL.md, no separate references/ directory

## Capabilities

### New Capabilities
*(none)*

### Modified Capabilities
- `cx-delegate-opsx`: Requirements enriched — prompt template expanded, review checklist deepened, troubleshooting and multi-task guidance added
- `oc-delegate-opsx`: Same requirements enrichment as cx-delegate-opsx

## Impact

- Two files modified: `skills/cx-delegate-opsx/SKILL.md`, `skills/oc-delegate-opsx/SKILL.md`
- No new files or directories created
- No changes to original `codex-delegate` or `opencode-delegate` skills
