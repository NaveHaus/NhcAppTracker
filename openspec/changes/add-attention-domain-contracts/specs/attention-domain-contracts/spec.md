## ADDED Requirements

### Requirement: Host-neutral foreground observation contract
The system SHALL provide a host-neutral foreground observation contract that represents a timestamped snapshot of the app and window context receiving user attention.

#### Scenario: Observation is created from valid captured context
- **WHEN** a foreground observation is created with a UTC observation timestamp, valid app identity, valid window context, and opaque user/session identity
- **THEN** the observation is accepted without requiring Windows-specific API types

#### Scenario: Observation rejects invalid captured context
- **WHEN** a foreground observation is created without a valid app identity or without window context
- **THEN** the observation is rejected before it can be used by later span-building logic

### Requirement: App identity preserves source context without Windows lock-in
The system SHALL provide an app identity contract that can preserve captured app context such as process name, executable path, display name, product name, and opaque source identifiers without exposing Windows-specific types in the domain model.

#### Scenario: App identity contains at least one identifying signal
- **WHEN** an app identity is created with at least one non-empty identifying signal
- **THEN** the app identity is accepted as valid domain data

#### Scenario: App identity rejects empty identity
- **WHEN** an app identity is created with no non-empty identifying signals
- **THEN** the app identity is rejected as invalid domain data

#### Scenario: Platform identifiers remain opaque
- **WHEN** platform-specific identifiers are captured by a future adapter
- **THEN** the domain model stores them only as opaque values and does not require Win32 handle, process, or window API types

### Requirement: Window context preserves titles for categorization
The system SHALL provide a window context contract that preserves the captured foreground window title as local domain data for future project categorization.

#### Scenario: Window title is captured
- **WHEN** a future foreground-window adapter provides a non-empty title
- **THEN** the domain window context retains the exact captured title value for later categorization

#### Scenario: Empty window title is captured
- **WHEN** a future foreground-window adapter provides an empty title
- **THEN** the domain window context accepts the empty title as valid captured context

### Requirement: Attention span represents completed attended time
The system SHALL provide an attention span contract that represents a completed contiguous time range of user attention for an app and window context.

#### Scenario: Valid attention span is created
- **WHEN** an attention span is created with UTC start and end timestamps, end after start, valid app identity, and valid window context
- **THEN** the attention span is accepted as valid attended time

#### Scenario: Attention span rejects invalid time range
- **WHEN** an attention span is created with an end timestamp equal to or before the start timestamp
- **THEN** the attention span is rejected as invalid attended time

#### Scenario: Attention span retains categorization context
- **WHEN** an attention span is created from foreground observation context
- **THEN** the attention span retains the app identity and window title needed by future categorization behavior

### Requirement: Tracking gap represents known untracked time
The system SHALL provide a tracking gap contract that represents a completed time range where attention was not tracked and a constrained reason explaining the gap.

#### Scenario: Valid tracking gap is created
- **WHEN** a tracking gap is created with UTC start and end timestamps, end after start, and a valid gap reason
- **THEN** the tracking gap is accepted as known untracked time

#### Scenario: Tracking gap rejects invalid time range
- **WHEN** a tracking gap is created with an end timestamp equal to or before the start timestamp
- **THEN** the tracking gap is rejected as invalid untracked time

#### Scenario: Tracking gap reason is constrained
- **WHEN** a tracking gap is created
- **THEN** the gap reason is limited to known domain reasons such as paused, stopped, crashed, system sleep, session locked, logged out, or unknown

### Requirement: Domain contracts remain platform-neutral
The system SHALL keep the attention-domain contracts in host-neutral domain code with no dependency on Windows-specific projects, Windows-only APIs, persistence packages, agent runtime loops, UI, or `LocalService` infrastructure.

#### Scenario: Domain dependencies are inspected
- **WHEN** the domain project references for attention-domain contracts are inspected
- **THEN** they contain no references to Windows-specific agent or platform projects

#### Scenario: Runtime behavior is reviewed
- **WHEN** this change is reviewed
- **THEN** it contains domain contracts and tests only, with no foreground-window capture, idle detection, persistence, categorization UI, startup registration, or `LocalService` behavior
