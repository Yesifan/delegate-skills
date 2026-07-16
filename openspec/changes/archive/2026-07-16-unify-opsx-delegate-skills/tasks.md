## 1. Create command reference files

- [x] 1.1 Create `skills/opsx-implementer/references/opencode-operations.md` with pure command snippets for: `preflightCheck`, `dispatchBrief`, `extractSessionId`, `readReport`, `resumeSession`, `backlinkFormat`

- [x] 1.2 Create `skills/opsx-implementer/references/codex-operations.md` with pure command snippets for: `preflightCheck`, `dispatchBrief`, `extractThreadId`, `readReport`, `resumeSession`, `backlinkFormat`

## 2. Create unified shared references

- [x] 2.1 Create `skills/opsx-implementer/references/writing-the-brief.md` unified from the OC version (path-based strategy; both CLIs read spec from working tree). Remove CX-specific "embed spec content directly" instructions. Keep the shared prompt template, discovery of real gates, model selection notes.

- [x] 2.2 Create `skills/opsx-implementer/references/dispatch-and-poll.md` as shared flow logic. Retain the pre-flight, dispatch sequence, waiting, and misbehavior sections from the current OC version. Where a CLI-specific command is needed, reference it by named key (e.g., "run oc `extractSessionId` or cx `extractThreadId`").

- [x] 2.3 Copy `skills/oc-delegate-opsx/references/review-and-land.md` to `skills/opsx-implementer/references/review-and-land.md` (agent-agnostic, unchanged)

- [x] 2.4 Copy `skills/oc-delegate-opsx/references/multi-task-queues.md` to `skills/opsx-implementer/references/multi-task-queues.md` (agent-agnostic, unchanged)

## 3. Create main SKILL.md

- [x] 3.1 Create `skills/opsx-implementer/SKILL.md` with frontmatter (`name: opsx-implementer`, `disable-model-invocation: true`), adding Step 0 (ask user for implementer choice), following the OC version's flow for all other steps. Reference CLI-specific operations via named keys to the operations files. No cost tracking step. No auto-install — report errors and stop.

## 4. Update registry and docs

- [x] 4.1 Update `skills.sh.json`: replace `oc-delegate-opsx` and `cx-delegate-opsx` entries with `opsx-implementer`

- [x] 4.2 Update `AGENTS.md`: replace the two opsx entries with `opsx-implementer`

- [x] 4.3 Update `README.md`: merge the two opsx rows into one; update directory tree and any section references

## 5. Delete old skill directories

- [x] 5.1 Remove `skills/oc-delegate-opsx/` directory

- [x] 5.2 Remove `skills/cx-delegate-opsx/` directory
