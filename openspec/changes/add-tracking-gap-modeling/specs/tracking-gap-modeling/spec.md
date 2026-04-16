## ADDED Requirements

### Requirement: Tracking gap creation uses completed UTC time ranges
The system SHALL create tracking gaps only from completed UTC time ranges with an end timestamp strictly after the start timestamp.

#### Scenario: Valid completed range is accepted
- **WHEN** a tracking gap is created with UTC start and end timestamps and the end timestamp is after the start timestamp
- **THEN** the system accepts the gap as a valid completed untracked-time range

#### Scenario: Equal range endpoints are rejected
- **WHEN** a tracking gap is created with the same UTC start and end timestamp
- **THEN** the system rejects the gap as an invalid untracked-time range

#### Scenario: Reversed range endpoints are rejected
- **WHEN** a tracking gap is created with a UTC end timestamp before the start timestamp
- **THEN** the system rejects the gap as an invalid untracked-time range

### Requirement: Tracking gap reasons are constrained
The system SHALL constrain tracking gap reasons to stopped, crashed, system sleep, session locked, logged out, and unknown.

#### Scenario: Stopped untracked time is represented
- **WHEN** a tracking gap is created for time after tracking was intentionally stopped
- **THEN** the gap reason is represented as stopped

#### Scenario: Crashed untracked time is represented
- **WHEN** a tracking gap is created for time after the tracker process failed unexpectedly
- **THEN** the gap reason is represented as crashed

#### Scenario: System sleep untracked time is represented
- **WHEN** a tracking gap is created for time where the host was suspended or asleep
- **THEN** the gap reason is represented as system sleep

#### Scenario: Session locked untracked time is represented
- **WHEN** a tracking gap is created for time where the user session was locked
- **THEN** the gap reason is represented as session locked

#### Scenario: Logged out untracked time is represented
- **WHEN** a tracking gap is created for time where the user was logged out
- **THEN** the gap reason is represented as logged out

#### Scenario: Unknown untracked time is represented
- **WHEN** a tracking gap is created for a known untracked time range whose cause cannot be classified
- **THEN** the gap reason is represented as unknown

#### Scenario: Unsupported reasons are rejected
- **WHEN** a tracking gap is created with a reason outside the constrained domain reason set
- **THEN** the system rejects the reason as invalid tracking-gap data

### Requirement: Tracking gap creation remains host-neutral
The system SHALL keep tracking gap creation independent of Windows-specific signal types, handles, event identifiers, session messages, foreground-window APIs, persistence APIs, UI APIs, and service infrastructure.

#### Scenario: Future Windows stop signal maps to host-neutral stopped reason
- **WHEN** a future Windows adapter detects that tracking was intentionally stopped
- **THEN** the domain accepts only the host-neutral stopped reason and does not require a Windows-specific stop signal type

#### Scenario: Future Windows crash signal maps to host-neutral crashed reason
- **WHEN** a future Windows adapter detects that tracking ended because the tracker process failed unexpectedly
- **THEN** the domain accepts only the host-neutral crashed reason and does not require a Windows process, handle, or event type

#### Scenario: Future Windows power signal maps to host-neutral system sleep reason
- **WHEN** a future Windows adapter detects host suspend or sleep time
- **THEN** the domain accepts only the host-neutral system sleep reason and does not require a Windows power-event type

#### Scenario: Future Windows session signal maps to host-neutral session locked reason
- **WHEN** a future Windows adapter detects that the user session was locked
- **THEN** the domain accepts only the host-neutral session locked reason and does not require a Windows session-message type

#### Scenario: Future Windows logout signal maps to host-neutral logged out reason
- **WHEN** a future Windows adapter detects that the user logged out
- **THEN** the domain accepts only the host-neutral logged out reason and does not require a Windows logoff-event type

### Requirement: Tracking gap creation is separate from signal capture
The system SHALL define tracking gap creation rules without implementing OS signal capture, foreground-window capture, idle detection, runtime loops, persistence, categorization UI, startup registration, telemetry, sync, network behavior, or `LocalService` behavior.

#### Scenario: Domain code is reviewed for platform capture
- **WHEN** the tracking-gap modeling implementation is reviewed
- **THEN** it contains domain contracts and tests only, with no Windows API capture, OS signal subscriptions, or foreground-window capture behavior

#### Scenario: Domain dependencies are inspected
- **WHEN** project references and package references for the tracking-gap modeling implementation are inspected
- **THEN** they do not introduce Windows-specific adapter projects, persistence packages, UI packages, telemetry packages, sync packages, network packages, or `LocalService` infrastructure
