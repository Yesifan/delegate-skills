## ADDED Requirements

### Requirement: Orchestrator delegates task groups via direct prompt

The orchestrator SHALL delegate task groups by constructing a prompt inline and piping it to `codex exec`, without using a relay script or brief.txt file.

#### Scenario: Delegate a task group
- **WHEN** the orchestrator wants to delegate task group N from an OpenSpec change
- **THEN** the orchestrator reads the spec, design, and task descriptions from the change directory
- **THEN** the orchestrator constructs a prompt containing the spec content and supplementary instructions
- **THEN** the orchestrator pipes the prompt to `codex exec -- -`

### Requirement: Sub-agent reads OpenSpec artifacts for context

The Codex sub-agent SHALL receive the specs, design, and tasks context in its prompt, not as opaque brief text. It SHALL use `openspec status` and `openspec instructions` to read additional context when instructed.

#### Scenario: Read spec from prompt
- **WHEN** the prompt includes the spec content from `openspec/changes/<name>/specs/<capability>/spec.md`
- **THEN** the sub-agent uses that content to understand what to implement

### Requirement: Sub-agent updates OpenSpec artifacts after implementation

After implementing, the Codex sub-agent SHALL update the change artifacts in the working tree:
- Mark completed tasks in `tasks.md` by changing `- [ ]` to `- [x]`
- Add implementation decisions to `design.md` if not already documented
- Update delta specs if implementation reveals spec inaccuracies

#### Scenario: Mark task done
- **WHEN** the sub-agent completes a task
- **THEN** it edits `tasks.md` to mark that task as done

### Requirement: Orchestrator captures session ID from CLI output

The orchestrator SHALL capture the Codex thread ID from the CLI's JSON output stream, not from sub-agent self-report. The orchestrator SHALL redirect stdout to a file with a fixed naming convention and parse the session ID from it after the run completes.

#### Scenario: Capture session from output
- **WHEN** the orchestrator delegates to `codex exec --json -o <final.txt>`
- **THEN** the orchestrator redirects stderr to `/tmp/delegate_<capability>_<task-id>.jsonl`
- **THEN** the orchestrator extracts `threadId` from the JSON stream after completion

### Requirement: Orchestrator records session in tasks.md for resume

The orchestrator SHALL write the captured session ID into `tasks.md` next to the delegated task, so the task artifact serves as a coordination record. The format SHALL be `delegate: codex <session-id>` appended to the task line.

#### Scenario: Record session in tasks
- **WHEN** the orchestrator captures a session ID from a delegation run
- **THEN** the orchestrator edits `tasks.md` to append `delegate: codex ses_xxx` to the task line
- **THEN** another agent or the orchestrator can resume this session by passing the recorded ID

### Requirement: Sub-agent supports explore mode via read-only sandbox

The sub-agent SHALL support read-only (explore) delegation using `codex exec -s read-only`. The orchestrator SHALL NOT pass `--auto` for read-only runs.

#### Scenario: Explore delegation
- **WHEN** the orchestrator delegates an exploration task
- **THEN** the orchestrator passes `--sandbox read-only` to prevent file edits
