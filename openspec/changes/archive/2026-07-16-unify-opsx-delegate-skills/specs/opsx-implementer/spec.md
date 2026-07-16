## ADDED Requirements

### Requirement: Unified skill entry
The system SHALL provide a single `opsx-implementer` skill that combines the capabilities of the former `oc-delegate-opsx` and `cx-delegate-opsx` skills.

#### Scenario: Invoke unified skill
- **WHEN** the orchestrator loads `opsx-implementer`
- **THEN** the skill SHALL ask the user which implementer to use (OpenCode or Codex)

#### Scenario: Implementer choice persists
- **WHEN** the user has chosen an implementer at entry
- **THEN** the same implementer SHALL be used for the duration of the delegation loop, including all parallel or sequential task groups

### Requirement: CLI-specific commands in isolated references
The skill SHALL separate CLI-specific commands from shared flow logic. Commands for each implementer SHALL live in dedicated reference files (`opencode-operations.md`, `codex-operations.md`).

#### Scenario: Command lookup by name
- **WHEN** the flow requires a CLI-specific operation (dispatch, session extraction, resume)
- **THEN** the flow SHALL reference the operation by a named key (e.g., `extractSessionId`) that the orchestrator looks up in the chosen implementer's operations file

#### Scenario: Commands are pure snippets
- **WHEN** the orchestrator reads an operations file
- **THEN** each entry SHALL contain only the shell command(s) with no flow logic, explanations, or branching

### Requirement: Shared references for agent-agnostic content
The skill SHALL retain shared reference files for content that does not vary by implementer: `dispatch-and-poll.md`, `writing-the-brief.md`, `review-and-land.md`, `multi-task-queues.md`.

#### Scenario: Dispatch-and-poll retained as shared flow
- **WHEN** the orchestrator follows the dispatch flow
- **THEN** `dispatch-and-poll.md` SHALL describe the sequence of steps, referencing CLI-specific operations by their named keys

### Requirement: No auto-install
The skill SHALL NOT attempt to install the implementer CLI. If the chosen CLI is missing or unauthenticated, the skill SHALL report the error and stop.

#### Scenario: CLI not found
- **WHEN** the chosen implementer CLI is not on PATH or is unauthenticated
- **THEN** the skill SHALL report the specific error to the user and stop, without attempting installation

### Requirement: No cost tracking
The skill SHALL NOT include cost extraction in the delegation flow.

#### Scenario: Cost tracking absent
- **WHEN** the orchestrator runs the delegation loop
- **THEN** no step SHALL extract or record run cost
