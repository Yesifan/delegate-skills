## 1. Create cx-delegate-opsx skill

- [x] 1.1 Create skill directory `skills/cx-delegate-opsx/` with `SKILL.md` including frontmatter (name, description, compatibility, metadata)
- [x] 1.2 Write the main skill body: delegation loop — orchestrator constructs prompt from OpenSpec artifacts, pipes to `codex exec`, captures stdout, reviews diff + artifacts
- [x] 1.3 Document the prompt template: embed spec content directly, pass supplementary context inline, instruct sub-agent to update tasks.md and design.md
- [x] 1.4 Document session capture: orchestrator parses threadId from stderr JSON stream, records as `delegate: codex <id>` in tasks.md
- [x] 1.5 Cover resume flow: orchestrator reads session ID from tasks.md, passes to `codex exec resume <id>` for delta rework
- [x] 1.6 Document explore mode: `codex exec -s read-only` for delegating research tasks without edit capability
- [x] 1.7 Document orchestrator decision logic: delegate vs. do-it-yourself based on task complexity and context preservation

## 2. Create oc-delegate-opsx skill

- [x] 2.1 Create skill directory `skills/oc-delegate-opsx/` with `SKILL.md` including frontmatter
- [x] 2.2 Write the main skill body: delegation loop — orchestrator constructs prompt from OpenSpec artifacts, pipes to `opencode run --format json`, captures stdout to `/tmp/delegate_{cap}_{task}.jsonl`, reviews diff + artifacts
- [x] 2.3 Cover model selection: orchestrator must pass `--model` on fresh runs, model ownership by human
- [x] 2.4 Document agent selection: `build` (write-capable, passes `--auto`) vs `plan` (read-only, no `--auto`)
- [x] 2.5 Document session capture: orchestrator parses `sessionID` from JSON event stream, records as `delegate: opencode <id>` in tasks.md
- [x] 2.6 Cover resume flow: orchestrator reads session ID from tasks.md, passes `--session <id>` for delta rework
- [x] 2.7 Document explore mode: `--agent plan` for delegating research tasks without edit capability
- [x] 2.8 Document orchestrator decision logic: delegate vs. do-it-yourself based on task complexity and context preservation

## 3. Update repo-level documentation

- [x] 3.1 Update `README.md` to list the two new skills with brief descriptions
- [x] 3.2 Validate locally: `npx skills add . --list` shows both new skills
- [x] 3.3 Smoke-test both skills' prompt templates with a `--read-only` run on a throwaway repo
