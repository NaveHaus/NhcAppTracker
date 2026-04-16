## ADDED Requirements

### Requirement: Per-user headless agent runtime composes backend capabilities
The system SHALL provide a per-user headless agent runtime that composes foreground sampling, activity signal handling, attention span building, tracking gap creation, local usage persistence, cancellation, controlled shutdown, and structured backend diagnostics.

#### Scenario: Runtime starts with configured backend dependencies
- **WHEN** the runtime is started with a foreground observation source, activity signal source, span builder, gap factory, local usage sink, clock, scheduler, cancellation token, and diagnostic sink
- **THEN** it MUST initialize those dependencies and begin runtime orchestration without requiring UI, startup registration, network access, external services, or `LocalService` infrastructure

#### Scenario: Runtime uses production Windows and local adapters only at the host edge
- **WHEN** the production per-user agent process is composed
- **THEN** it MUST wire the Windows foreground capture adapter, Windows activity signal adapter, host-neutral span builder, host-neutral tracking gap rules, and local usage persistence implementation at the executable or host boundary

#### Scenario: Runtime remains testable with fakes
- **WHEN** automated tests run the runtime with fake foreground sources, fake activity sources, fake persistence, fake diagnostics, fake clocks, fake schedulers, and controllable cancellation
- **THEN** the runtime MUST exercise sampling, signal handling, span persistence, gap persistence, diagnostics, and shutdown behavior without requiring real foreground-window changes, session lock, suspend, resume, logout, shutdown, disk writes, UI, or network access

### Requirement: Runtime samples and persists foreground observations
The runtime SHALL sample foreground context on the configured schedule and persist each valid foreground observation through the local usage sink before using it to complete spans.

#### Scenario: Valid foreground sample is persisted
- **WHEN** the foreground source returns a valid `ForegroundObservation` during a scheduled sample
- **THEN** the runtime MUST write that observation to the local usage sink and emit a structured diagnostic indicating the sample was accepted

#### Scenario: Empty foreground sample is ignored without fabrication
- **WHEN** the foreground source reports that no valid foreground observation is available
- **THEN** the runtime MUST NOT create a foreground observation, attention span, or tracking gap from that sample and MUST emit a structured diagnostic indicating the sample was empty or unavailable

#### Scenario: Invalid or failed foreground sample is diagnosed
- **WHEN** foreground capture fails or returns data that cannot become a valid `ForegroundObservation`
- **THEN** the runtime MUST NOT persist fabricated usage data and MUST emit a structured diagnostic that includes the failure classification and sampling timestamp

#### Scenario: Sampling interval is scheduler controlled
- **WHEN** the runtime is configured with a sampling interval and scheduler
- **THEN** it MUST request samples through the scheduler so tests can advance time deterministically and production can use real elapsed time

### Requirement: Runtime completes and persists spans from ordered observations
The runtime SHALL maintain pending observation state and use the host-neutral span builder to persist completed attention spans only when a later valid observation creates a boundary.

#### Scenario: First valid observation becomes pending context
- **WHEN** the runtime accepts its first valid foreground observation
- **THEN** it MUST persist the observation and keep it as pending context without persisting a completed attention span

#### Scenario: Later context boundary completes previous span
- **WHEN** the runtime accepts a later valid observation whose app identity or window context differs from the pending observation
- **THEN** it MUST use the span builder to create the completed span ending at the later observation timestamp and persist the completed span through the local usage sink

#### Scenario: Same context remains pending
- **WHEN** the runtime accepts a later valid observation with the same app identity and window context as the pending observation
- **THEN** it MUST persist the observation but MUST NOT persist a completed attention span for that unchanged context

#### Scenario: Non-increasing observation order is rejected
- **WHEN** a newly accepted foreground observation has an `ObservedAtUtc` timestamp equal to or earlier than the pending observation timestamp
- **THEN** the runtime MUST NOT pass the invalid sequence to span persistence, MUST leave existing durable records intact, and MUST emit a structured diagnostic describing the ordering violation

#### Scenario: Shutdown does not fabricate a final span
- **WHEN** runtime cancellation or controlled shutdown occurs while an observation is pending without a later valid boundary observation
- **THEN** the runtime MUST NOT complete that pending context as an attention span at shutdown time

### Requirement: Runtime converts activity signals into sampling state and tracking gaps
The runtime SHALL consume host-neutral activity events and use them to pause or resume attended-time sampling and to persist completed tracking gaps for known untracked ranges.

#### Scenario: Idle transition starts untracked range
- **WHEN** the runtime receives an idle activity event while attended-time sampling is active
- **THEN** it MUST stop treating foreground samples as attended time, remember the idle event timestamp as the start of a pending untracked range, and emit a structured diagnostic for the transition

#### Scenario: Active transition closes idle gap
- **WHEN** the runtime receives an active activity event after a pending idle untracked range
- **THEN** it MUST create and persist a completed tracking gap from the idle timestamp through the active timestamp using a constrained host-neutral gap reason and resume attended-time sampling

#### Scenario: Session lock closes attended-time tracking
- **WHEN** the runtime receives a session locked activity event while attended-time sampling is active
- **THEN** it MUST stop attended-time sampling and start a pending untracked range with the session locked reason

#### Scenario: Session unlock persists locked gap
- **WHEN** the runtime receives a session unlocked activity event after a pending locked untracked range
- **THEN** it MUST create and persist a completed tracking gap from the lock timestamp through the unlock timestamp using the session locked reason and resume attended-time sampling

#### Scenario: System suspend and resume persist sleep gap
- **WHEN** the runtime receives a suspending activity event followed by a resumed activity event
- **THEN** it MUST create and persist a completed tracking gap from the suspend timestamp through the resume timestamp using the system sleep reason before resuming attended-time sampling

#### Scenario: Logout or shutdown stops runtime tracking
- **WHEN** the runtime receives a logging out or shutting down activity event
- **THEN** it MUST stop scheduling new foreground samples, record controlled shutdown diagnostics, and persist any completed known gap range that has a valid end timestamp

#### Scenario: Duplicate activity state does not create duplicate gaps
- **WHEN** the runtime receives repeated activity events that do not change the effective runtime tracking state
- **THEN** it MUST NOT persist duplicate tracking gaps and MUST emit at most diagnostic state confirmations for the repeated events

### Requirement: Runtime handles cancellation and controlled shutdown deterministically
The runtime SHALL stop predictably on cancellation or controlled shutdown, avoid new samples after stop begins, flush accepted writes, and report shutdown outcome through structured diagnostics.

#### Scenario: Cancellation stops new sampling
- **WHEN** the runtime cancellation token is canceled
- **THEN** the runtime MUST stop scheduling new foreground samples and begin controlled shutdown

#### Scenario: In-flight accepted records are flushed
- **WHEN** cancellation occurs after an observation, completed span, or completed gap has been accepted for persistence
- **THEN** the runtime MUST await or complete the corresponding local usage sink write before reporting shutdown complete unless the sink reports a failure

#### Scenario: Controlled stop records stopped gap only for completed range
- **WHEN** the runtime is stopped intentionally and has a known untracked range with an end timestamp strictly after its start timestamp
- **THEN** it MUST persist a tracking gap using the stopped reason

#### Scenario: Controlled stop avoids open-ended gap fabrication
- **WHEN** the runtime is stopped intentionally but only knows the start of an untracked range or only has a pending foreground observation
- **THEN** it MUST NOT persist an open-ended or fabricated tracking gap and MUST emit diagnostics describing the pending state left for later recovery

#### Scenario: Persistence failure during shutdown is diagnosed
- **WHEN** the local usage sink rejects or fails a write during controlled shutdown
- **THEN** the runtime MUST emit a structured error diagnostic with the record kind and failure details and MUST NOT report that record as successfully persisted

### Requirement: Runtime emits structured local backend diagnostics
The runtime SHALL emit structured local backend diagnostics for lifecycle, sampling, activity handling, span completion, gap creation, persistence, cancellation, and shutdown outcomes without transmitting captured usage data externally.

#### Scenario: Lifecycle diagnostics are emitted
- **WHEN** the runtime starts, becomes ready, begins shutdown, or completes shutdown
- **THEN** it MUST emit structured diagnostics with stable event names, UTC timestamps, severity, and runtime state

#### Scenario: Sampling diagnostics are emitted
- **WHEN** a foreground sample succeeds, returns empty, is rejected, or fails
- **THEN** the runtime MUST emit structured diagnostics that identify the sample outcome without sending data to a network endpoint

#### Scenario: Persistence diagnostics are emitted
- **WHEN** an observation, attention span, or tracking gap write succeeds or fails
- **THEN** the runtime MUST emit structured diagnostics that identify the persisted record kind, outcome, and failure details when applicable

#### Scenario: Diagnostics remain local
- **WHEN** runtime diagnostics are configured for production
- **THEN** diagnostics MUST be written only to local sinks such as console, local logs, or in-process test collectors and MUST NOT use HTTP clients, sockets, telemetry uploaders, analytics clients, sync clients, or remote endpoints to transmit captured usage data

### Requirement: Runtime preserves backend scope and dependency boundaries
The runtime SHALL keep backend orchestration separate from UI, categorization, startup registration, network transmission, telemetry upload, and `LocalService` behavior while preserving host-neutral dependency direction.

#### Scenario: Runtime scope is reviewed
- **WHEN** the change is reviewed
- **THEN** it MUST contain no tray UI, timeline UI, project/category assignment, categorization rules, startup registration, installer behavior, network sync, telemetry upload, external analytics, or `LocalService` implementation

#### Scenario: Host-neutral projects remain free of Windows dependencies
- **WHEN** host-neutral domain and application project references are inspected
- **THEN** they MUST NOT reference Windows-specific adapter projects, Windows-only APIs, UI framework references, persistence implementation packages, network clients, telemetry clients, or service infrastructure

#### Scenario: Windows-specific dependencies stay at the edge
- **WHEN** the per-user Windows agent executable or Windows host project references are inspected
- **THEN** Windows-specific references MUST be limited to the executable or adapter edge that composes host-neutral runtime contracts with Windows capture and signal implementations
