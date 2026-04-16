## ADDED Requirements

### Requirement: Backend hardening verification is repeatable
The system SHALL provide a repeatable backend-only hardening verification process that records the environment, commands, timestamps, expected outcomes, actual outcomes, diagnostics, persistence evidence, and pass/fail status for each backend MVP scenario.

#### Scenario: Hardening run records environment and scope
- **WHEN** the backend hardening verification process starts
- **THEN** it MUST record the repository worktree path, change name, Windows version, current user context, backend executable or command under test, local storage location under test, verification start timestamp, and the explicit statement that the run is backend-only and local-only

#### Scenario: Hardening run records scenario evidence
- **WHEN** a hardening scenario is executed
- **THEN** the process MUST record the scenario name, input actions or commands, relevant UTC timestamps, expected persisted records or diagnostics, actual persisted records or diagnostics, and pass/fail status

#### Scenario: Hardening run fails on missing evidence
- **WHEN** a scenario cannot produce required diagnostics, local command output, persisted record summaries, or database integrity output
- **THEN** the scenario MUST be marked failed or inconclusive with the missing evidence identified and MUST NOT be reported as passed

#### Scenario: Verification additions are tested first
- **WHEN** code is added or changed to implement hardening harnesses, parsers, integrity checks, local-only guards, or evidence rendering
- **THEN** tests MUST be written or updated first using a TDD red/green/refactor workflow and stored under the relevant `tests/<category>` path

### Requirement: Foreground capture is verified end to end
The system SHALL verify that real Windows foreground app switches and window-title changes are captured locally and reflected through persisted records and backend diagnostics without requiring a front-end UI.

#### Scenario: App switch creates observable local usage evidence
- **WHEN** the backend agent is running and the verifier switches between at least two distinct foreground applications long enough to be sampled
- **THEN** diagnostics and local persistence MUST show foreground observations for each app identity and completed attention span evidence for the app transition when the configured span-building rules allow completion

#### Scenario: Window title change is captured locally
- **WHEN** the foreground application remains the same but its window title changes during a tracked interval
- **THEN** diagnostics and local persistence MUST show the changed window context in observations or spans according to the backend capture and span-building model

#### Scenario: Capture evidence preserves local-only boundary
- **WHEN** captured app identity or window title evidence is inspected during hardening
- **THEN** it MUST be available only through local diagnostics, local command output, local files, or local database reads and MUST NOT require telemetry, sync, analytics, HTTP, sockets, or remote endpoints

### Requirement: Activity transition handling is verified end to end
The system SHALL verify that idle, session lock/unlock, sleep/resume, logout or controlled shutdown where practical, and resumed foreground tracking produce correct local gaps, spans, runtime state, and diagnostics.

#### Scenario: Idle creates constrained tracking gap evidence
- **WHEN** the backend agent observes an idle interval that meets the configured idle threshold
- **THEN** local persistence and diagnostics MUST show a tracking gap or equivalent pending gap state with the idle reason and MUST NOT count idle time as attended foreground time

#### Scenario: Session lock and unlock create constrained tracking evidence
- **WHEN** the Windows session is locked and later unlocked while the backend agent is installed or running
- **THEN** local persistence and diagnostics MUST show a session-locked tracking gap or equivalent constrained state for the unavailable interval and foreground tracking MUST resume after unlock without extending a pre-lock attended span through the locked interval

#### Scenario: Sleep and resume create constrained tracking evidence
- **WHEN** the Windows system sleeps or suspends and later resumes while backend tracking is installed or running
- **THEN** local persistence and diagnostics MUST show a system-sleep, resume, reboot-compatible, or unknown unavailable-tracking gap according to the recovery and gap model and MUST NOT fabricate attended time across the unavailable interval

#### Scenario: Controlled stop records bounded shutdown evidence
- **WHEN** the backend agent is stopped through a controlled shutdown path during hardening
- **THEN** diagnostics, checkpoints, and local usage records MUST show flushed accepted records and clean shutdown or stopped-tracking state sufficient for the next startup to avoid crash-style fabrication

### Requirement: Restart and crash recovery are verified end to end
The system SHALL verify that restart, process interruption, reboot where practical, and crash recovery reconcile local records deterministically and do not fabricate attended time.

#### Scenario: Agent restart runs recovery before normal tracking
- **WHEN** the backend agent is stopped and started again during hardening
- **THEN** local diagnostics MUST show recovery ran before normal sampling resumed and local persistence MUST NOT contain duplicate recovery-created records for the same prior runtime state

#### Scenario: Crash recovery records unavailable tracking gap
- **WHEN** the backend agent process is terminated uncleanly and then restarted
- **THEN** recovery diagnostics and local persistence MUST show either a constrained pending-gap completion or an unknown unavailable-tracking gap for the valid unavailable interval and MUST NOT create an attended span across the crash downtime

#### Scenario: Repeated recovery remains idempotent
- **WHEN** recovery is invoked more than once over equivalent durable prior runtime state during hardening
- **THEN** local persistence MUST contain exactly one equivalent recovery-created gap or status update per recovery range and diagnostics MUST identify duplicate skips as idempotent decisions

#### Scenario: Startup registration launches normal runtime
- **WHEN** current-user startup registration is installed and the verifier signs in or invokes the registered launch path equivalently for hardening
- **THEN** the registered command MUST launch the normal per-user backend runtime, diagnostics MUST identify normal runtime startup, and startup management commands MUST NOT be registered as the sign-in runtime command

### Requirement: Local database integrity is verified
The system SHALL verify local usage persistence integrity after normal capture, activity transitions, restart recovery, crash recovery, and diagnostics inspection.

#### Scenario: Integrity checks pass after hardening scenarios
- **WHEN** the hardening scenario sequence completes
- **THEN** the local database or persistence store MUST pass the project's integrity check mechanism and the hardening evidence MUST include the integrity command or API result

#### Scenario: Persisted usage records have valid time ordering
- **WHEN** hardening inspects persisted observations, attention spans, tracking gaps, recovery records, or checkpoints
- **THEN** every completed range MUST have an end timestamp strictly after its start timestamp, observations MUST have valid observed timestamps, and invalid or skipped ranges MUST be represented only by diagnostics rather than fabricated usage records

#### Scenario: Diagnostics inspection does not mutate usage records
- **WHEN** hardening runs backend diagnostics queries repeatedly after capture and recovery scenarios
- **THEN** diagnostics inspection MUST NOT create, edit, delete, duplicate, or reorder persisted observations, spans, gaps, checkpoints, or recovery records

#### Scenario: Restart does not corrupt local persistence
- **WHEN** the backend agent is restarted after local records have been written
- **THEN** previously persisted valid observations, spans, gaps, checkpoints, and recovery provenance MUST remain readable through local diagnostics or integrity inspection

### Requirement: Absence of network transmission is verified
The system SHALL verify that backend hardening scenarios do not transmit captured usage data, window titles, app identity, session identity, runtime state, diagnostics, health state, checkpoints, or recovery metadata over a network.

#### Scenario: Static network dependency inspection passes
- **WHEN** backend projects, command handlers, diagnostics handlers, startup registration, recovery, and hardening support code are inspected
- **THEN** they MUST contain no HTTP clients, sockets, telemetry clients, sync clients, analytics clients, remote endpoints, or network transport paths used to transmit captured usage data or backend diagnostics

#### Scenario: Runtime network observation shows no backend transmission
- **WHEN** foreground capture, activity transition, recovery, startup, diagnostics, and integrity hardening scenarios execute
- **THEN** process-scoped runtime observation or an equivalent local network check MUST show no outbound backend network transmission of captured usage data, diagnostics, runtime state, health state, checkpoints, or recovery metadata

#### Scenario: Network check failures are not ignored
- **WHEN** static inspection or runtime network observation cannot be completed or produces ambiguous process attribution
- **THEN** the network absence result MUST be marked failed or inconclusive with the reason recorded and MUST NOT be reported as passed

### Requirement: Backend readiness for front-end round is explicitly decided
The system SHALL produce an explicit backend readiness decision for the separate front-end UI round based on hardening evidence and documented residual risk.

#### Scenario: Readiness requires required scenarios to pass
- **WHEN** the hardening run is evaluated for front-end readiness
- **THEN** the backend MUST be marked ready only if required capture, activity transition, restart recovery, startup registration, backend diagnostics, local database integrity, and network absence scenarios pass or have documented non-blocking exceptions

#### Scenario: Readiness report identifies front-end usable boundaries
- **WHEN** the backend is marked ready for the front-end round
- **THEN** the readiness evidence MUST identify the local backend data and diagnostic boundaries available to the UI round, including observations, spans, gaps, health, current runtime state, and recent diagnostic events

#### Scenario: Readiness report blocks UI dependency on failed backend behavior
- **WHEN** a hardening scenario fails in a way that affects UI assumptions about captured data, gaps, recovery, diagnostics, or local-only behavior
- **THEN** the backend MUST NOT be marked ready for the front-end round until the failure is fixed or explicitly accepted as a documented non-blocking limitation

#### Scenario: Backend scope remains unchanged by readiness
- **WHEN** the readiness report is produced
- **THEN** it MUST NOT introduce tray UI, timeline UI, project/category assignment, categorization rules, export/sync workflow, telemetry upload, remote analytics, `LocalService`, service installation, or product front-end behavior
