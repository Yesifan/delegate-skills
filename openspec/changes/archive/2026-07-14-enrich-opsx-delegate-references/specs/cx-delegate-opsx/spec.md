## ADDED Requirements

### Requirement: Prompt includes verification_loop block

The prompt the orchestrator constructs SHALL include a `<verification_loop>` block that embeds the project's actual gate commands (test, lint, build). The orchestrator SHALL discover these from the repo's CLAUDE.md, AGENTS.md, Makefile, or package.json.

#### Scenario: Verification loop in prompt
- **WHEN** the orchestrator constructs the prompt
- **THEN** the orchestrator reads the real gate commands from the repo
- **THEN** the prompt includes a verification block telling the sub-agent to run those commands and fix any failures before reporting done

### Requirement: Prompt includes action_safety block

The prompt SHALL include an `<action_safety>` block that instructs the sub-agent: no git add/commit, no scope creep (changes beyond what the spec/task asks for), and leave untouched what the brief says to leave untouched.

#### Scenario: Action safety in prompt
- **WHEN** the prompt includes action_safety instructions
- **THEN** the sub-agent restricts edits to the task's scope
- **THEN** the sub-agent does NOT run git add or git commit

### Requirement: Prompt includes structured_output_contract

The prompt SHALL include a `<structured_output_contract>` block specifying the report format: what changed and why, files touched, gate outcomes with test/lint counts, and any deviations from the spec.

#### Scenario: Structured output in prompt
- **WHEN** the sub-agent finishes implementation
- **THEN** it reports in the requested format
- **THEN** the orchestrator can parse the report from the captured output

### Requirement: Review checks test integrity before gates

The orchestrator SHALL review the sub-agent's test edits BEFORE re-running the gates. If existing tests were weakened (assertions loosened, skips added, tests deleted), the orchestrator SHALL flag these as contract changes, not accept them as part of the fix.

#### Scenario: Test integrity check
- **WHEN** the sub-agent's diff touches existing tests
- **THEN** the orchestrator reviews those edits first
- **THEN** the orchestrator flags weakened assertions, added skips, or deleted tests

### Requirement: Review includes implementer sweep for gate-blind failures

The orchestrator SHALL check the diff against 11 failure patterns that gates are structurally blind to:
1. Hardcoded success/fixture data
2. Catch-all error handling that returns defaults instead of propagating
3. Unverified imports or API calls (not confirmed against installed version)
4. Dead weight: unused imports, unreachable branches, scaffolding comments
5. Second way to do what the file already does (new HTTP client, error idiom, logging style)
6. Tests that assert internals (mocking own functions, asserting internal calls)
7. Near-duplicate test bodies differing by one value
8. Speculative surface: optional params, config flags, abstractions with no caller
9. Guards for impossible cases (null checks on values the contract already excludes)
10. Loosened test boundaries (exact match → contains/truthy)
11. Skipped/disabled/commented-out tests added in the diff

#### Scenario: Implementer sweep
- **WHEN** the orchestrator reviews the sub-agent's diff
- **THEN** the orchestrator walks all 11 patterns against every changed file

### Requirement: Skill includes troubleshooting guidance

The SKILL.md SHALL include a troubleshooting section covering: how to verify the CLI is available and what version, how to read the output file for diagnosis, and common failure causes with their resolutions.

#### Scenario: CLI unavailable
- **WHEN** codex CLI is not found
- **THEN** the orchestrator reports the error and tells the user to install

#### Scenario: Failed run diagnosis
- **WHEN** a delegation run exits with non-zero code
- **THEN** the orchestrator reads the output file's stderr tail to identify the cause

### Requirement: Skill includes constraint forwarding guidance

When delegating multiple task groups sequentially, the orchestrator SHALL restate constraints discovered in earlier groups in later groups' prompts. The sub-agent has no memory of prior runs.

#### Scenario: Constraint forwarding
- **WHEN** the orchestrator delegates task group N+1
- **THEN** the orchestrator checks what decisions/constraints emerged from group N
- **THEN** the orchestrator includes those constraints explicitly in group N+1's prompt

### Requirement: Skill includes final coherence check

After all task groups for a change are complete, the orchestrator SHALL run the full test/build suite on the final tree and do a repo-wide sweep for the change's concern (e.g. after a removal, grep for surviving references).

#### Scenario: Coherence check
- **WHEN** all delegated task groups are done and reviewed
- **THEN** the orchestrator runs the complete test suite, not just each task's slice
- **THEN** the orchestrator does a repo-wide check for the change's subject
