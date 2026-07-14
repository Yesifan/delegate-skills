## 1. Enrich prompt template

- [x] 1.1 Add `verification_loop` block to prompt template: instruct sub-agent to run the project's actual gate commands, discoverable from repo config files, and fix failures before reporting done
- [x] 1.2 Add `action_safety` block to prompt template: no git add/commit, no scope creep, leave untouched what's specified
- [x] 1.3 Add `structured_output_contract` block to prompt template: specify report format (what changed, files touched, gate outcomes, deviations)

## 2. Expand review checklist

- [x] 2.1 Add test integrity check: review test edits BEFORE re-running gates — flag weakened assertions, added skips, deleted tests
- [x] 2.2 Add implementer sweep with 11 failure patterns: hardcoded success, catch-all errors that swallow failures, unverified imports, dead weight, duplicate patterns, internals-testing, near-duplicate tests, speculative surface, impossible guards, loosened test boundaries, skipped/disabled tests

## 3. Add troubleshooting section

- [x] 3.1 Add troubleshooting guidance: CLI binary check (`command -v`, `--version`), output file diagnosis (reading stderr tail), common failure causes and resolutions
- [x] 3.2 Add completion verification note: process exit + output file existence before reading results

## 4. Add multi-task sequencing guidance

- [x] 4.1 Add constraint forwarding: restate decisions from earlier task groups in later prompts — sub-agent has no memory of prior runs
- [x] 4.2 Add final coherence check: run full test suite after all groups done, repo-wide sweep for the change's concern

## 5. Apply both skills

- [x] 5.1 Apply all changes to `skills/cx-delegate-opsx/SKILL.md`
- [x] 5.2 Apply all changes to `skills/oc-delegate-opsx/SKILL.md`
- [x] 5.3 Validate: `npx skills add . --list` shows both skills
