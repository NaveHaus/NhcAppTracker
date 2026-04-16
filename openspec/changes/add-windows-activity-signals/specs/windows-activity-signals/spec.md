## ADDED Requirements

### Requirement: Windows activity signals are captured behind a Windows-only adapter
The system SHALL provide Windows-specific activity signal capture for idle state, active state, session lock, session unlock, system suspend, system resume, user logout, and system shutdown without exposing Windows API types through host-neutral application contracts.

#### Scenario: Windows idle signal is captured
- **WHEN** the Windows idle source observes that the configured idle threshold has been reached
- **THEN** the adapter emits a host-neutral activity event representing that the user became idle

#### Scenario: Windows active signal is captured
- **WHEN** the Windows idle source observes user input after a previously idle state
- **THEN** the adapter emits a host-neutral activity event representing that the user became active

#### Scenario: Windows session lock signal is captured
- **WHEN** Windows reports that the current user session has been locked
- **THEN** the adapter emits a host-neutral activity event representing a locked session

#### Scenario: Windows session unlock signal is captured
- **WHEN** Windows reports that the current user session has been unlocked
- **THEN** the adapter emits a host-neutral activity event representing an unlocked session

#### Scenario: Windows power lifecycle signal is captured
- **WHEN** Windows reports system suspend or resume
- **THEN** the adapter emits a host-neutral activity event representing system suspending or system resumed

#### Scenario: Windows exit lifecycle signal is captured
- **WHEN** Windows reports user logout or system shutdown
- **THEN** the adapter emits a host-neutral activity event representing user logging out or system shutting down

### Requirement: Activity events use a host-neutral event contract
The system SHALL translate all captured Windows activity signals into a host-neutral event contract with a constrained event kind and an observed UTC timestamp.

#### Scenario: Raw Windows signal is translated
- **WHEN** a raw Windows session, power, idle, logout, or shutdown signal is received
- **THEN** the translated event contains no Win32 handles, Windows message IDs, UI framework event args, or other Windows-specific API types

#### Scenario: Event kind is constrained
- **WHEN** a host-neutral activity event is created
- **THEN** the event kind is limited to known activity lifecycle values for idle, active, locked, unlocked, suspending, resumed, logging out, shutting down, and unknown only when the source signal cannot be classified

#### Scenario: Event timestamp is UTC
- **WHEN** a host-neutral activity event is emitted
- **THEN** the event contains the UTC timestamp observed at the adapter boundary

### Requirement: Activity signal streams suppress duplicate state transitions
The system SHALL emit activity events for state transitions and MUST NOT emit duplicate events for repeated raw signals that do not change the effective activity state.

#### Scenario: Repeated idle samples are suppressed
- **WHEN** the idle source produces multiple samples while the user remains idle
- **THEN** only the first idle transition emits a host-neutral idle event

#### Scenario: Repeated active samples are suppressed
- **WHEN** the idle source produces multiple samples while the user remains active after an active transition
- **THEN** only the first active transition emits a host-neutral active event

#### Scenario: Distinct lifecycle transitions are preserved
- **WHEN** the adapter observes different lifecycle transitions in sequence
- **THEN** each distinct transition is emitted in source order with its observed UTC timestamp

### Requirement: Activity signal translation is testable without real OS transitions
The system SHALL make Windows activity signal translation testable with deterministic clocks and fake raw signal sources so automated tests do not need to lock, unlock, suspend, resume, log out, or shut down the development machine.

#### Scenario: Translation is tested with fake signals
- **WHEN** tests provide fake raw Windows idle, session, power, logout, or shutdown signals
- **THEN** the translator emits the same host-neutral events that a real adapter source would emit

#### Scenario: Timestamps are deterministic in tests
- **WHEN** tests provide an explicit UTC clock or timestamp source
- **THEN** emitted host-neutral activity events use the provided timestamps

### Requirement: Windows activity signal capture remains scoped to adapter behavior
The system SHALL NOT add UI behavior, persistence, foreground-window capture, attention span building, runtime orchestration, startup registration, network transmission, telemetry, or `LocalService` infrastructure as part of Windows activity signal capture.

#### Scenario: No direct UI behavior is added
- **WHEN** this change is reviewed
- **THEN** it contains no tray UI, timeline UI, user-facing controls, WPF UI behavior, or Windows Forms UI behavior beyond any minimal non-UI notification hook required by the adapter boundary

#### Scenario: No persistence or runtime orchestration is added
- **WHEN** this change is reviewed
- **THEN** it contains no storage writes, span-building runtime loop, foreground-window capture, startup registration, network transmission, telemetry, or `LocalService` behavior

#### Scenario: Host-neutral projects remain free of Windows dependencies
- **WHEN** host-neutral project references are inspected
- **THEN** they contain no Windows-only API references, Windows-specific adapter references, UI framework references, persistence packages, or `LocalService` infrastructure
