## ADDED Requirements

### Requirement: Build completed spans from ordered observations
The system SHALL provide host-neutral application logic that converts a strictly ordered sequence of `ForegroundObservation` values into completed `AttentionSpan` records.

#### Scenario: Empty observation sequence produces no spans
- **WHEN** the span builder receives an empty observation sequence
- **THEN** it MUST return no completed attention spans

#### Scenario: Single observation produces no completed span
- **WHEN** the span builder receives exactly one foreground observation
- **THEN** it MUST return no completed attention spans

#### Scenario: Same context observations remain pending
- **WHEN** the span builder receives multiple foreground observations with the same app identity and window context
- **THEN** it MUST return no completed attention spans because no later context boundary exists

#### Scenario: Context change completes previous span
- **WHEN** the span builder receives a foreground observation followed by a later observation with a different app identity or window context
- **THEN** it MUST return one completed attention span from the first observation timestamp through the later boundary observation timestamp

### Requirement: Detect app and window context boundaries
The system SHALL treat app identity and window context together as the attention context used to decide when a pending span ends.

#### Scenario: App identity change completes previous span
- **WHEN** two consecutive observations have different app identity values and valid increasing timestamps
- **THEN** the span builder MUST complete the previous span at the second observation timestamp

#### Scenario: Window context change completes previous span
- **WHEN** two consecutive observations have the same app identity but different window context values and valid increasing timestamps
- **THEN** the span builder MUST complete the previous span at the second observation timestamp

#### Scenario: Completed span preserves starting context
- **WHEN** an observation is followed by a later observation with a different attention context
- **THEN** the completed span MUST preserve the app identity and window context from the starting observation

### Requirement: Validate observation ordering
The system SHALL require input observations to be in strictly increasing `ObservedAtUtc` order before producing completed spans.

#### Scenario: Equal timestamps are rejected
- **WHEN** the span builder receives consecutive observations with equal `ObservedAtUtc` timestamps
- **THEN** it MUST reject the sequence and produce no completed spans from that input

#### Scenario: Decreasing timestamps are rejected
- **WHEN** the span builder receives an observation whose `ObservedAtUtc` timestamp is earlier than the previous observation timestamp
- **THEN** it MUST reject the sequence and produce no completed spans from that input

#### Scenario: Increasing timestamps are accepted
- **WHEN** the span builder receives observations with strictly increasing `ObservedAtUtc` timestamps
- **THEN** it MUST evaluate context boundaries and return all completed spans implied by the sequence

### Requirement: Remain host-neutral
The system SHALL implement span building without persistence, Windows APIs, runtime sampling, hosted services, UI, telemetry, sync, startup registration, or `LocalService` infrastructure.

#### Scenario: Builder uses domain contracts only
- **WHEN** the span builder is implemented
- **THEN** it MUST depend on the host-neutral attention domain contracts and MUST NOT depend on Windows-specific APIs, persistence packages, UI projects, telemetry, sync, startup registration, or `LocalService` infrastructure
