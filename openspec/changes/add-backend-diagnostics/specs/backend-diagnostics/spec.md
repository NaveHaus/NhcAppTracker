## ADDED Requirements

### Requirement: Backend diagnostics exposes bounded local usage inspection
The system SHALL provide backend-only local inspection queries for recently persisted foreground observations, completed attention spans, and tracking gaps through application-level contracts over the local usage persistence boundary.

#### Scenario: Recent observations are inspected locally
- **WHEN** a diagnostics caller requests recent foreground observations with an explicit limit or time window
- **THEN** the system MUST return matching locally persisted observations with observed UTC timestamp, app identity, window context, and opaque user/session identity without requiring UI, project/category assignment, export, sync, telemetry, analytics, or network access

#### Scenario: Recent spans are inspected locally
- **WHEN** a diagnostics caller requests recent completed attention spans with an explicit limit or time window
- **THEN** the system MUST return matching locally persisted spans with UTC time range, app identity, and window context without categorizing, editing, exporting, syncing, or transmitting the data

#### Scenario: Recent gaps are inspected locally
- **WHEN** a diagnostics caller requests recent tracking gaps with an explicit limit or time window
- **THEN** the system MUST return matching locally persisted gaps with UTC time range and constrained gap reason without requiring front-end timeline behavior or project/category assignment

#### Scenario: Unbounded usage inspection is rejected
- **WHEN** a diagnostics caller requests persisted usage records without an explicit limit or time window
- **THEN** the system MUST reject the request or apply a documented conservative default bound and emit a local diagnostic describing the applied bound

### Requirement: Backend diagnostics exposes current runtime state
The system SHALL expose a local point-in-time snapshot of the per-user agent runtime state for backend verification.

#### Scenario: Runtime state snapshot is requested
- **WHEN** a diagnostics caller requests current runtime state
- **THEN** the system MUST return a snapshot containing lifecycle state, sampling state, attended-time tracking state, pending observation presence, pending gap presence, last successful sample timestamp when known, last accepted persistence timestamp when known, and snapshot UTC timestamp

#### Scenario: Runtime state inspection remains read-only
- **WHEN** runtime state is inspected
- **THEN** the system MUST NOT start, stop, restart, repair, reconfigure, register startup behavior, create foreground observations, complete spans, create gaps, or mutate persistence as part of the inspection

#### Scenario: Runtime state changes during inspection
- **WHEN** runtime state changes while a diagnostics snapshot is being collected
- **THEN** the system MUST return a coherent point-in-time or internally consistent best-effort snapshot with its own UTC timestamp rather than combining values without indicating when the snapshot was taken

### Requirement: Backend diagnostics exposes local health status
The system SHALL provide local health status for backend runtime dependencies and recent backend outcomes without adding watchdog, recovery, or remote monitoring behavior.

#### Scenario: Health status reports dependency availability
- **WHEN** a diagnostics caller requests backend health
- **THEN** the system MUST report observable status for the runtime, foreground capture dependency, activity signal dependency, local usage persistence dependency, scheduler, and diagnostic sink using stable status values

#### Scenario: Health status reports recent failures
- **WHEN** recent sampling, signal handling, persistence, span completion, gap creation, or diagnostics failures are known to the runtime diagnostics provider
- **THEN** the health status MUST include local failure summaries with timestamps, severity, component, and failure classification without reporting the backend as healthy for the affected component

#### Scenario: Health inspection does not perform recovery
- **WHEN** backend health is inspected
- **THEN** the system MUST NOT restart the runtime, recreate storage, register startup tasks, repair records, upload telemetry, contact remote endpoints, or otherwise change backend behavior

### Requirement: Backend diagnostics exposes recent structured diagnostics locally
The system SHALL allow recent structured backend diagnostic events to be inspected locally through bounded queries.

#### Scenario: Recent diagnostic events are inspected
- **WHEN** a diagnostics caller requests recent structured diagnostic events with an explicit limit or time window
- **THEN** the system MUST return matching local diagnostic events with stable event name, UTC timestamp, severity, component, runtime state when available, correlation fields when available, and failure details when applicable

#### Scenario: Diagnostic event inspection is bounded
- **WHEN** a diagnostics caller requests diagnostic events without an explicit limit or time window
- **THEN** the system MUST reject the request or apply a documented conservative default bound and emit a local diagnostic describing the applied bound

#### Scenario: Diagnostic inspection avoids remote sinks
- **WHEN** diagnostic events are inspected in production or tests
- **THEN** the system MUST use only local sinks such as in-process collectors, console output, local files, or local store reads and MUST NOT use HTTP clients, sockets, telemetry uploaders, analytics clients, sync clients, or remote endpoints

### Requirement: Developer-facing inspection commands are thin local adapters
The system SHALL provide developer-facing local command or handler paths for backend diagnostics as thin adapters over the diagnostics query contracts when a command surface is added.

#### Scenario: Local observations command delegates to query contract
- **WHEN** a developer invokes a local command or handler to inspect recent observations
- **THEN** the command MUST delegate to the bounded observations diagnostics query and render local output without directly reading storage tables, mutating runtime state, or using network transport

#### Scenario: Local spans command delegates to query contract
- **WHEN** a developer invokes a local command or handler to inspect recent spans
- **THEN** the command MUST delegate to the bounded spans diagnostics query and render local output without categorizing, editing, exporting, syncing, or transmitting the data

#### Scenario: Local gaps command delegates to query contract
- **WHEN** a developer invokes a local command or handler to inspect recent gaps
- **THEN** the command MUST delegate to the bounded gaps diagnostics query and render local output without front-end timeline behavior or project/category assignment

#### Scenario: Local health command delegates to query contract
- **WHEN** a developer invokes a local command or handler to inspect health or current state
- **THEN** the command MUST delegate to the health or runtime-state diagnostics query and render local output without starting, stopping, restarting, or repairing the runtime

### Requirement: Backend diagnostics preserves local-only privacy boundary
The system SHALL keep backend diagnostic inspection local-only and MUST NOT transmit observations, spans, gaps, window titles, app identity, session identity, runtime state, health state, or diagnostic events over a network.

#### Scenario: Diagnostics dependencies are inspected
- **WHEN** diagnostics project references, command handlers, query handlers, and source code are inspected
- **THEN** they MUST contain no dependency or code path for HTTP clients, sockets, telemetry clients, sync clients, analytics clients, remote endpoints, or network transport used to transmit captured usage data or backend diagnostics

#### Scenario: Diagnostics output remains local
- **WHEN** backend diagnostics are queried or rendered by a local command
- **THEN** output MUST be written only to local sinks such as return values, console output, local files, in-process collectors, or local store reads and MUST NOT be uploaded, synchronized, exported to remote services, or sent to external analytics

#### Scenario: Captured window titles are exposed only through explicit local inspection
- **WHEN** diagnostics output includes window titles or app identity for verification
- **THEN** those values MUST be returned only through explicit local diagnostics requests and MUST NOT be included in remote telemetry, analytics, sync, or network payloads

### Requirement: Backend diagnostics preserves backend scope and dependency boundaries
The system SHALL keep diagnostics inspection separate from UI, categorization, startup registration, service infrastructure, and Windows-specific implementation details except at the executable or adapter edge.

#### Scenario: Diagnostics scope is reviewed
- **WHEN** this change is reviewed
- **THEN** it MUST contain no tray UI, timeline UI, web UI, project/category assignment, categorization rules, visual editing, purge workflow, export workflow, sync workflow, startup registration, installer behavior, `LocalService` implementation, remote diagnostics, telemetry upload, or external analytics behavior

#### Scenario: Host-neutral diagnostics contracts avoid Windows dependencies
- **WHEN** host-neutral diagnostics query contracts and DTOs are inspected
- **THEN** they MUST NOT reference Windows-specific adapter projects, Windows-only APIs, UI framework references, local persistence implementation packages, network clients, telemetry clients, or service infrastructure

#### Scenario: Edge commands may compose concrete dependencies
- **WHEN** a per-user agent executable or backend command surface composes diagnostics dependencies
- **THEN** Windows-specific or persistence implementation references MUST remain limited to the executable or adapter edge and MUST NOT be introduced into host-neutral domain or application contracts
