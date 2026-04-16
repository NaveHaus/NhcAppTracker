## ADDED Requirements

### Requirement: Local usage sink boundary persists attention domain values
The system SHALL provide a host-neutral local usage sink boundary that accepts the foreground observation, attention span, and tracking gap domain contracts from `add-attention-domain-contracts` without requiring Windows-specific capture types.

#### Scenario: Usage sink accepts domain values
- **WHEN** application code writes a valid foreground observation, a valid attention span, and a valid tracking gap through the local usage sink boundary
- **THEN** the boundary accepts those domain values without requiring Windows API types, foreground-window handles, runtime agent state, UI state, or project/category assignments

#### Scenario: Usage sink rejects invalid domain values before storage
- **WHEN** application code attempts to write an invalid observation, invalid attention span, or invalid tracking gap
- **THEN** the value is rejected by existing domain validation or sink input validation before a persisted row is created

### Requirement: Local store round-trips foreground observations
The system SHALL persist foreground observations locally and read them back with their observed UTC timestamp, app identity, window context, and opaque user/session identity preserved.

#### Scenario: Observation is written and read back
- **WHEN** a valid foreground observation with app identity, window title, opaque source identifiers, and user/session identity is written to a new local store and then read back from the same store path
- **THEN** the returned observation matches the persisted domain value for timestamp, app identity fields, window context fields, and user/session identity

#### Scenario: Observation survives store reopen
- **WHEN** a valid foreground observation is written, the store is disposed or closed, and a new store instance opens the same local path
- **THEN** the observation remains readable without requiring network access or external services

### Requirement: Local store round-trips attention spans
The system SHALL persist completed attention spans locally and read them back with their UTC time range, app identity, and window context preserved.

#### Scenario: Attention span is written and read back
- **WHEN** a valid attention span with a UTC start, UTC end after start, app identity, and window title is written to a local store and then read back
- **THEN** the returned attention span matches the persisted domain value for time range, app identity fields, and window context fields

#### Scenario: Multiple attention spans are preserved independently
- **WHEN** two valid attention spans with different time ranges or app/window context are written to the same local store
- **THEN** both spans are readable as distinct persisted records without merging, categorizing, or assigning them to projects

### Requirement: Local store round-trips tracking gaps
The system SHALL persist tracking gaps locally and read them back with their UTC time range and constrained gap reason preserved.

#### Scenario: Tracking gap is written and read back
- **WHEN** a valid tracking gap with a UTC start, UTC end after start, and constrained gap reason is written to a local store and then read back
- **THEN** the returned tracking gap matches the persisted domain value for time range and reason

#### Scenario: Multiple gap reasons are preserved
- **WHEN** tracking gaps with different valid reasons are written to the same local store
- **THEN** each returned gap preserves its original constrained reason value

### Requirement: Local persistence schema excludes categorization assignments
The system SHALL create only the local storage structures required for usage observations, attention spans, tracking gaps, and minimal storage metadata, and MUST NOT create project/category assignment tables or relationships in this change.

#### Scenario: New local schema is created
- **WHEN** a local store is opened at an empty path and initializes its schema
- **THEN** the schema contains storage for foreground observations, attention spans, and tracking gaps

#### Scenario: Assignment schema is absent
- **WHEN** the initialized local schema is inspected
- **THEN** it contains no project tables, category tables, assignment tables, join tables linking usage to projects/categories, sync tables, telemetry tables, analytics tables, export tables, or remote endpoint tables

#### Scenario: Minimal metadata is allowed
- **WHEN** the selected storage engine requires schema metadata for compatibility or migration checks
- **THEN** the implementation may create minimal schema metadata while still excluding project/category assignment and network-related tables

### Requirement: Persisted usage data remains strictly local
The system SHALL keep persisted usage data strictly local and MUST NOT transmit observations, attention spans, tracking gaps, app identity, window titles, or session identity over a network.

#### Scenario: Persistence writes local storage only
- **WHEN** domain usage values are written through the local persistence implementation
- **THEN** the implementation writes only to the configured local store path and performs no HTTP calls, socket connections, telemetry emission, sync operation, analytics upload, export, or remote service call

#### Scenario: Persistence dependencies are inspected
- **WHEN** the local persistence project references and source are inspected
- **THEN** they contain no dependency or code path for HTTP clients, remote endpoints, telemetry clients, sync clients, analytics clients, or network transport used to transmit usage data

### Requirement: Persistence dependency direction remains host-neutral
The system SHALL keep the local persistence boundary and implementation independent of Windows-specific projects and APIs.

#### Scenario: Project references are inspected
- **WHEN** project references for the domain, application, local persistence, and Windows-specific projects are inspected
- **THEN** domain and application projects do not reference the local persistence implementation, and the local persistence implementation does not reference Windows-specific agent or platform projects

#### Scenario: Runtime scope is reviewed
- **WHEN** this change is reviewed
- **THEN** it contains local persistence boundary and storage behavior only, with no foreground-window capture, idle detection, span-building runtime loop, project/category assignment UI, startup registration, installer behavior, or `LocalService` behavior
