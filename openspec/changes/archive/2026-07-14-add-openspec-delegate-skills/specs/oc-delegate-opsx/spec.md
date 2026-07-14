## ADDED Requirements

### Requirement: Orchestrator delegates task groups via direct prompt

The orchestrator SHALL delegate task groups by constructing a prompt inline and piping it to `opencode run`, without using a relay script or brief.txt file.

#### Scenario: Delegate a task group
- **WHEN** the orchestrator wants to delegate task group N from an OpenSpec change
- **THEN** the orchestrator reads the spec, design, and task descriptions from the change directory
- **THEN** the orchestrator constructs a prompt containing the spec content and supplementary instructions
- **THEN** the orchestrator pipes the prompt to `opencode run --agent build --model <model> --format json --auto -- -`

### Requirement: Sub-agent uses appropriate OpenCode agent

The sub-agent SHALL use the `build` agent for write-capable delegation and the `plan` agent for read-only review. The orchestrator SHALL specify the agent and model in the command.

#### Scenario: Write-capable delegation
- **WHEN** the orchestrator delegates implementation work
- **THEN** the orchestrator passes `--agent build --auto` to enable unattended edits

#### Scenario: Read-only review
- **WHEN** the orchestrator wants the sub-agent to review code without editing
- **THEN** the orchestrator passes `--agent plan` (no `--auto`) for read-only operation

### Requirement: Sub-agent reads OpenSpec artifacts for context

The OpenCode sub-agent SHALL receive the specs, design, and tasks context in its prompt. It SHALL use `openspec status` and `openspec instructions` to read additional context when instructed.

#### Scenario: Read spec from prompt
- **WHEN** the prompt includes the spec content from `openspec/changes/<name>/specs/<capability>/spec.md`
- **THEN** the sub-agent uses that content to understand what to implement

### Requirement: Sub-agent updates OpenSpec artifacts after implementation

After implementing, the OpenCode sub-agent SHALL update the change artifacts in the working tree:
- Mark completed tasks in `tasks.md` by changing `- [ ]` to `- [x]`
- Add implementation decisions to `design.md` if not already documented
- Update delta specs if implementation reveals spec inaccuracies

#### Scenario: Mark task done
- **WHEN** the sub-agent completes a task
- **THEN** it edits `tasks.md` to mark that task as done

### Requirement: Orchestrator captures session ID from CLI output

The orchestrator SHALL capture the OpenCode session ID from the CLI's JSON event stream, not from sub-agent self-report. The orchestrator SHALL redirect stdout to a file with a fixed naming convention and parse the session ID from it after the run completes.

#### Scenario: Capture session from output
- **WHEN** the orchestrator delegates to `opencode run --format json`
- **THEN** the orchestrator redirects stdout to `/tmp/delegate_<capability>_<task-id>.jsonl`
- **THEN** the orchestrator extracts `sessionID` from the JSON event stream after completion

### Requirement: Orchestrator records session in tasks.md for resume

The orchestrator SHALL write the captured session ID into `tasks.md` next to the delegated task. The format SHALL be `delegate: opencode <session-id>` appended to the task line.

#### Scenario: Record session in tasks
- **WHEN** the orchestrator captures a session ID from a delegation run
- **THEN** the orchestrator edits `tasks.md` to append `delegate: opencode ses_abc` to the task line
- **THEN** another agent or the orchestrator can resume this session by passing `--session ses_abc`

### Requirement: Sub-agent supports explore mode via plan agent

The sub-agent SHALL support read-only (explore) delegation using `--agent plan`. The orchestrator SHALL NOT pass `--auto` for plan agent runs.

#### Scenario: Explore delegation
- **WHEN** the orchestrator delegates an exploration task
- **THEN** the orchestrator passes `--agent plan` (no `--auto`) to prevent file edits

### Requirement: Model selection is required for fresh runs

The orchestrator MUST pass `--model provider/model` on every fresh `opencode run`. A resumed run inherits the original session's model.

#### Scenario: Fresh run requires model
- **WHEN** the orchestrator starts a new delegation without `--session` or `--continue`
- **THEN** the orchestrator MUST include `--model <provider/model>`
