## ADDED Requirements

### Requirement: Capture current Windows foreground window
The system SHALL provide a Windows platform adapter that captures the current foreground window in the interactive user session without adding capture behavior to host-neutral projects.

#### Scenario: Foreground window is available
- **WHEN** the Windows foreground capture adapter samples the active interactive session and a foreground window is available
- **THEN** it captures the foreground window identity, exact current window title, owning process ID, and capture timestamp for translation into a foreground observation

#### Scenario: No foreground window is available
- **WHEN** the Windows foreground capture adapter samples the active interactive session and Windows reports no foreground window
- **THEN** it returns no foreground observation and does not fabricate an app identity, window context, attention span, or tracking gap

### Requirement: Capture process identity and executable metadata
The system SHALL capture the owning process identity for the foreground window, including process name, executable path/name when accessible, and available app metadata such as display name, product name, or file description.

#### Scenario: Process metadata is accessible
- **WHEN** the foreground window owning process can be opened and its executable metadata can be read
- **THEN** the adapter includes the process ID, process name, executable path, executable file name, and available display/product metadata in the captured Windows snapshot

#### Scenario: Optional executable metadata is unavailable
- **WHEN** the foreground window owning process is accessible but executable metadata such as display name or product name is missing or unreadable
- **THEN** the adapter still captures a valid Windows snapshot using the available process identity and window title

#### Scenario: Owning process cannot be resolved
- **WHEN** the foreground window owning process cannot be resolved well enough to produce a valid app identity
- **THEN** the adapter returns no foreground observation and leaves gap/span decisions to later application logic

### Requirement: Translate Windows data into foreground observations
The system SHALL translate captured Windows foreground-window data into the host-neutral `ForegroundObservation` contract from the attention-domain contracts.

#### Scenario: Captured Windows snapshot is valid
- **WHEN** a captured Windows snapshot contains a capture timestamp, app identity signals, user/session identity, and window title
- **THEN** the mapper creates a `ForegroundObservation` with a valid `AppIdentity`, valid `WindowContext`, and opaque `UserSessionIdentity`

#### Scenario: Empty window title is captured
- **WHEN** Windows reports an empty title for an otherwise valid foreground window
- **THEN** the mapper creates a foreground observation whose window context preserves the empty title as captured context

#### Scenario: Window title contains whitespace or punctuation
- **WHEN** Windows reports a non-empty title containing whitespace, punctuation, or mixed casing
- **THEN** the mapper preserves the exact title value in the foreground observation window context

### Requirement: Keep Windows-specific identifiers at the adapter edge
The system SHALL keep Win32 handles, process IDs, and Windows API types out of host-neutral project APIs while allowing Windows-specific identifiers to be retained only as adapter-local data or opaque domain source identifiers.

#### Scenario: Domain observation is created
- **WHEN** the Windows adapter translates a captured snapshot into a foreground observation
- **THEN** host-neutral domain values do not expose Win32 handle, process, window, or Windows API types

#### Scenario: Platform source identifiers are preserved
- **WHEN** process ID, window handle, executable path, or other Windows-specific identifiers are useful for source identity
- **THEN** the adapter maps them only into domain-supported opaque source identifier values or keeps them inside Windows-edge types

### Requirement: Avoid span-building and persistence
The system SHALL limit the Windows foreground capture adapter to sampling and domain-observation translation, and SHALL NOT build attention spans, create tracking gaps, persist captured data, transmit captured data, or categorize observations.

#### Scenario: Observation is captured
- **WHEN** the Windows foreground capture adapter successfully captures and translates foreground context
- **THEN** it returns a foreground observation without creating an attention span, tracking gap, persisted record, category assignment, telemetry event, or external transmission

#### Scenario: Capture fails
- **WHEN** the Windows foreground capture adapter cannot produce a valid foreground observation
- **THEN** it returns a failure or empty result without persisting failure state or creating a tracking gap

### Requirement: Windows capture remains isolated from host-neutral projects
The system SHALL implement foreground-window capture only in Windows-specific project boundaries and SHALL keep host-neutral domain and application projects free of Windows-specific API references.

#### Scenario: Project references are inspected
- **WHEN** project references are inspected after implementing the Windows foreground capture adapter
- **THEN** host-neutral projects do not reference Windows-specific adapter projects

#### Scenario: Windows API usage is inspected
- **WHEN** Windows API imports and platform-specific capture code are inspected
- **THEN** they exist only in Windows-specific platform adapter code and tests
