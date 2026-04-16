## ADDED Requirements

### Requirement: Agent startup runs recovery before normal tracking
The system SHALL run deterministic recovery for the per-user backend agent before starting foreground sampling, activity subscriptions, span completion, or normal runtime tracking after agent process start.

#### Scenario: Recovery runs before sampling starts
- **WHEN** the per-user agent process starts with local usage persistence, recovery checkpoint storage, clock, runtime identity, and diagnostic sink configured
- **THEN** it MUST run recovery to completion before scheduling any foreground sample or subscribing to normal activity events

#### Scenario: Clean prior shutdown allows normal startup
- **WHEN** recovery finds a prior checkpoint marked as cleanly shut down with no pending foreground context or pending untracked range requiring repair
- **THEN** it MUST emit a structured recovery diagnostic and allow normal runtime sampling to start without creating recovery gaps or spans

#### Scenario: Missing checkpoint with no prior usage initializes recovery state
- **WHEN** recovery finds no prior checkpoint and no local usage records requiring reconciliation
- **THEN** it MUST initialize recovery checkpoint state for the new runtime instance and allow normal runtime sampling to start

#### Scenario: Recovery failure prevents attended tracking
- **WHEN** recovery cannot read required checkpoint state, inspect required local usage records, or persist required recovery output
- **THEN** it MUST emit a structured error diagnostic and MUST NOT start attended-time foreground sampling on top of unreconciled state

### Requirement: Runtime persists recovery checkpoints for restart reconciliation
The system SHALL persist local recovery checkpoint metadata whenever runtime state changes in a way that affects crash, reboot, or restart recovery.

#### Scenario: Checkpoint records pending foreground context
- **WHEN** the runtime accepts and persists a valid foreground observation as the pending context
- **THEN** it MUST persist checkpoint metadata identifying the runtime instance, checkpoint version, last heartbeat or state timestamp, and the pending foreground observation reference or value needed for recovery

#### Scenario: Checkpoint records pending untracked range
- **WHEN** the runtime starts a pending untracked range because of idle, session lock, suspend, controlled stop, logout, shutdown, or another constrained tracking interruption
- **THEN** it MUST persist checkpoint metadata with the pending range start timestamp and constrained pending reason

#### Scenario: Checkpoint records clean shutdown
- **WHEN** the runtime completes controlled shutdown after flushing accepted observation, span, gap, and checkpoint writes
- **THEN** it MUST mark the checkpoint as cleanly shut down with the shutdown timestamp and no unreconciled pending state except state explicitly left for later recovery diagnostics

#### Scenario: Checkpoint remains local
- **WHEN** recovery checkpoint metadata is persisted
- **THEN** it MUST be written only to local storage and MUST NOT use HTTP clients, sockets, telemetry uploaders, sync clients, analytics clients, or remote endpoints

### Requirement: Recovery detects unclean termination deterministically
The system SHALL classify prior runtime termination as clean or unclean from durable checkpoint state and local usage state without relying on in-memory process state.

#### Scenario: Unclean crash is detected
- **WHEN** recovery finds a prior checkpoint that lacks a clean shutdown marker and has a last heartbeat or state timestamp before the current recovery timestamp
- **THEN** it MUST classify the prior runtime termination as unclean and emit a structured diagnostic with the previous runtime instance identifier and recovery timestamp

#### Scenario: Reboot restart is detected when boot identity changes
- **WHEN** recovery finds a prior checkpoint whose optional boot identity differs from the current host boot identity
- **THEN** it MUST classify the recovery as crossing a reboot boundary and use that classification in diagnostics and recovery provenance

#### Scenario: Agent restart without reboot is detected
- **WHEN** recovery finds a prior unclean checkpoint whose optional boot identity matches the current host boot identity
- **THEN** it MUST classify the recovery as an agent restart or process interruption without requiring a reboot-specific repair path

#### Scenario: Invalid recovery timestamps are rejected
- **WHEN** checkpoint or usage timestamps would produce a recovery range whose end timestamp is equal to or earlier than its start timestamp
- **THEN** recovery MUST NOT persist a fabricated gap or span for that range and MUST emit a structured diagnostic describing the invalid ordering

### Requirement: Recovery records unknown gaps for unavailable tracking
The system SHALL persist an explicit tracking gap for intervals where tracking was unavailable after unclean termination and no more specific constrained reason can be proven.

#### Scenario: Crash downtime becomes unknown gap
- **WHEN** recovery detects unclean termination after the last durable runtime state timestamp and no pending constrained untracked reason applies
- **THEN** it MUST persist a tracking gap from the last durable runtime state timestamp to the recovery timestamp using the unknown or unclassified unavailable-tracking reason

#### Scenario: Pending foreground context is not extended through downtime
- **WHEN** recovery detects a pending foreground observation from the previous runtime and no later valid foreground observation was captured while the runtime was available
- **THEN** it MUST NOT create an attention span from the pending observation through the recovery timestamp and MUST record unavailable time as a tracking gap when the range is valid

#### Scenario: Unknown gap uses local usage sink
- **WHEN** recovery persists an unknown unavailable-tracking gap
- **THEN** it MUST write the gap through the local usage persistence boundary with recovery provenance and emit a structured persistence diagnostic

#### Scenario: No gap is fabricated for unknown start
- **WHEN** recovery cannot identify a reliable start timestamp for unavailable tracking
- **THEN** it MUST NOT persist an open-ended or guessed tracking gap and MUST emit diagnostics describing the unrecoverable missing boundary

### Requirement: Recovery completes pending untracked ranges using constrained reasons
The system SHALL recover pending untracked ranges with their original constrained reason when the prior runtime checkpoint proves the reason and a valid recovery end timestamp is available.

#### Scenario: Pending locked range is completed on restart
- **WHEN** recovery finds a pending session-locked untracked range from a prior unclean runtime and the current recovery timestamp is after the range start
- **THEN** it MUST persist a tracking gap from the locked timestamp through the recovery timestamp using the session locked reason and recovery provenance

#### Scenario: Pending idle range is completed on restart
- **WHEN** recovery finds a pending idle untracked range from a prior unclean runtime and the current recovery timestamp is after the range start
- **THEN** it MUST persist a tracking gap from the idle timestamp through the recovery timestamp using the idle reason and recovery provenance

#### Scenario: Pending sleep range may cross reboot
- **WHEN** recovery finds a pending system-sleep untracked range and the optional boot identity changed before recovery
- **THEN** it MUST persist a tracking gap using the system sleep or reboot-compatible constrained reason selected by the gap model and MUST record recovery provenance

#### Scenario: Pending range with invalid end is not persisted
- **WHEN** recovery finds a pending untracked range whose recovery end timestamp is not strictly after its start timestamp
- **THEN** it MUST NOT persist a completed gap and MUST emit a structured diagnostic for the skipped recovery action

### Requirement: Recovery marks abandoned pending state without fabricating attended time
The system SHALL mark or checkpoint abandoned pending runtime state with recovery provenance when needed, but MUST NOT create attended attention spans from time when the agent was unavailable.

#### Scenario: Pending span remains uncompleted after crash
- **WHEN** recovery finds a pending foreground observation from an unclean prior runtime
- **THEN** it MUST mark the prior pending context as abandoned or superseded by recovery metadata when supported and MUST NOT persist a completed attention span without a later valid observation boundary captured during runtime availability

#### Scenario: Later post-recovery observation starts new context
- **WHEN** recovery completes and the runtime accepts its first valid foreground observation after restart
- **THEN** that observation MUST become the new pending context for normal span building and MUST NOT be used to complete a pre-crash attended span across the unavailable interval

#### Scenario: Recovery status updates are local and auditable
- **WHEN** recovery marks pending state as abandoned, superseded, or recovered
- **THEN** the status update MUST be stored locally with recovery provenance and MUST emit a structured diagnostic identifying the action without transmitting usage data externally

### Requirement: Recovery is idempotent
The system SHALL make recovery-created gaps, status updates, and checkpoint transitions idempotent so repeated recovery over the same durable state does not duplicate records or change outcomes.

#### Scenario: Repeated recovery does not duplicate unknown gap
- **WHEN** recovery is run more than once for the same prior runtime instance, checkpoint version, unavailable range, and recovery reason
- **THEN** it MUST leave exactly one corresponding recovery-created tracking gap or equivalent idempotent record

#### Scenario: Crash during recovery can be retried
- **WHEN** the agent crashes after persisting part of a recovery result and starts again
- **THEN** recovery MUST detect already-applied recovery output and complete any remaining required recovery actions without duplicating successful prior writes

#### Scenario: Idempotent recovery emits accurate diagnostics
- **WHEN** recovery skips a write because the equivalent recovery output already exists
- **THEN** it MUST emit a structured diagnostic identifying the skipped duplicate as an idempotent recovery decision rather than a newly persisted usage fact

### Requirement: Recovery preserves local-only backend boundaries
The system SHALL keep restart recovery backend-only, local-only, and separated from UI, categorization, startup registration, network transmission, telemetry upload, and Windows service infrastructure.

#### Scenario: Recovery scope is reviewed
- **WHEN** the recovery change is reviewed
- **THEN** it MUST contain no tray UI, timeline UI, project/category assignment, categorization rules, startup registration, installer behavior, network sync, telemetry upload, external analytics, remote diagnostics, or `LocalService` implementation

#### Scenario: Host-neutral recovery has no Windows dependencies
- **WHEN** host-neutral recovery, domain, application, and persistence boundary projects are inspected
- **THEN** they MUST NOT reference Windows-only APIs, UI framework references, Windows adapter projects, network clients, telemetry clients, sync clients, analytics clients, or service infrastructure

#### Scenario: Platform restart facts stay at the edge
- **WHEN** optional platform facts such as boot identity, process identity, or session identity are needed for recovery classification
- **THEN** they MUST be provided through injectable host-edge ports so recovery logic remains testable without Windows APIs

### Requirement: Recovery emits structured local diagnostics
The system SHALL emit structured local backend diagnostics for recovery lifecycle, classification, repair decisions, skipped fabrication, persistence outcomes, idempotence, and failure states.

#### Scenario: Recovery lifecycle diagnostics are emitted
- **WHEN** recovery starts, classifies prior state, applies recovery output, skips a recovery action, completes, or fails
- **THEN** it MUST emit structured diagnostics with stable event names, UTC timestamps, severity, runtime instance identifiers where available, and recovery state

#### Scenario: Recovery diagnostics explain no-fabrication decisions
- **WHEN** recovery declines to create an attention span, tracking gap, or status update because the required durable evidence is missing or invalid
- **THEN** it MUST emit a structured diagnostic identifying the missing or invalid boundary and the reason no usage fact was fabricated

#### Scenario: Diagnostics remain local
- **WHEN** recovery diagnostics are configured for production
- **THEN** diagnostics MUST be written only to local sinks such as console, local logs, or in-process collectors and MUST NOT transmit captured usage data, window titles, app identity, session identity, or recovery metadata to remote endpoints
