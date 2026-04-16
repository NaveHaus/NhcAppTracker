## ADDED Requirements

### Requirement: Per-user agent startup registration is supported
The system SHALL provide startup registration that launches the per-user backend agent runtime when the current interactive user signs in.

#### Scenario: Startup registration targets current user
- **WHEN** startup registration is installed
- **THEN** the registration MUST be scoped to the current user and MUST NOT require machine-wide installation, elevation, a Windows service, `LocalService`, or service-control-manager registration

#### Scenario: Registered command launches normal agent runtime
- **WHEN** the current user signs in after startup registration is installed
- **THEN** Windows MUST be able to invoke a registered command that starts the backend agent runtime in normal runtime mode for that user session

#### Scenario: Startup registration uses runtime artifact from dependency
- **WHEN** the registered command is constructed
- **THEN** it MUST target the per-user agent runtime executable or host produced by the `add-user-agent-runtime` dependency and MUST NOT register a separate service host

### Requirement: Startup install is reversible and idempotent
The system SHALL provide an install path that creates or updates current-user startup registration and an uninstall path that removes that registration.

#### Scenario: Install creates missing registration
- **WHEN** startup registration is absent and the install path is executed
- **THEN** the system MUST create current-user startup registration with the canonical agent launch command and report success through a local command result or diagnostic

#### Scenario: Install replaces stale registration
- **WHEN** startup registration exists with a missing, stale, or non-canonical launch command and the install path is executed
- **THEN** the system MUST replace it with the canonical agent launch command and report that registration was updated

#### Scenario: Install is idempotent
- **WHEN** startup registration already exists with the canonical agent launch command and the install path is executed
- **THEN** the system MUST leave registration equivalent to the existing canonical value and report success without creating duplicate startup entries

#### Scenario: Uninstall removes existing registration
- **WHEN** startup registration exists and the uninstall path is executed
- **THEN** the system MUST remove that current-user startup registration and report success through a local command result or diagnostic

#### Scenario: Uninstall is idempotent
- **WHEN** startup registration is already absent and the uninstall path is executed
- **THEN** the system MUST report success or an absent state without failing and MUST NOT create any registration

### Requirement: Startup registration status is observable
The system SHALL provide a way to inspect whether startup registration is absent, installed with the canonical command, or present with a mismatched command.

#### Scenario: Status reports absent registration
- **WHEN** no current-user startup registration exists for the agent and status is requested
- **THEN** the system MUST report an absent state without modifying registration

#### Scenario: Status reports installed registration
- **WHEN** current-user startup registration exists with the canonical agent launch command and status is requested
- **THEN** the system MUST report an installed state and include local-only details sufficient to verify the registered command

#### Scenario: Status reports mismatched registration
- **WHEN** current-user startup registration exists for the agent but points to a different executable path or launch arguments and status is requested
- **THEN** the system MUST report a mismatched or stale state without modifying registration

### Requirement: Startup registration command rendering is deterministic
The system SHALL render the registered launch command deterministically from the resolved agent runtime path and normal runtime arguments.

#### Scenario: Executable path with spaces is quoted
- **WHEN** the agent executable path contains spaces and install constructs the registered command
- **THEN** the command MUST quote the executable path so Windows sign-in launch can invoke the correct runtime

#### Scenario: Runtime arguments are preserved
- **WHEN** normal runtime startup requires launch arguments
- **THEN** the registered command MUST include those arguments in deterministic order with quoting or escaping sufficient for Windows command-line parsing

#### Scenario: Startup management commands are not registered
- **WHEN** install constructs the registered command
- **THEN** the command MUST NOT register `install-startup`, `uninstall-startup`, `startup-status`, or other startup-management arguments as the sign-in runtime command

### Requirement: Startup registration remains local and backend-only
The system SHALL keep startup registration local, backend-only, and separate from UI, services, network transmission, telemetry upload, and unrelated runtime behavior.

#### Scenario: Registration does not add service infrastructure
- **WHEN** the change is reviewed
- **THEN** it MUST contain no Windows service implementation, `LocalService` host, service installer, service recovery configuration, or service-control-manager integration

#### Scenario: Registration does not add UI or remote communication
- **WHEN** startup registration install, uninstall, status, or sign-in launch behavior executes
- **THEN** it MUST NOT require tray UI, timeline UI, network sync, telemetry upload, analytics clients, HTTP clients, sockets, or remote endpoints

#### Scenario: Host-neutral boundary remains free of Windows APIs
- **WHEN** host-neutral registration abstractions and command behavior are inspected
- **THEN** they MUST NOT reference Windows registry APIs, Windows service APIs, UI frameworks, network clients, telemetry clients, or persistence implementation details

#### Scenario: Windows-specific registration stays at the edge
- **WHEN** the Windows startup registration implementation is inspected
- **THEN** Windows-specific APIs MUST be limited to the edge implementation that adapts the host-neutral registration boundary to current-user sign-in registration

### Requirement: Startup registration failures are reported without fabricating success
The system SHALL report registration failures locally and MUST NOT claim successful install, uninstall, or status results when the underlying registration operation fails.

#### Scenario: Install write fails
- **WHEN** the current-user registration store rejects or fails the install write
- **THEN** the system MUST report a failed install result with local diagnostic details and MUST NOT report registration as successfully installed

#### Scenario: Uninstall delete fails
- **WHEN** the current-user registration store rejects or fails the uninstall delete
- **THEN** the system MUST report a failed uninstall result with local diagnostic details and MUST NOT report registration as successfully removed

#### Scenario: Status read fails
- **WHEN** the current-user registration store rejects or fails a status read
- **THEN** the system MUST report a failed status result with local diagnostic details and MUST NOT infer absent, installed, or mismatched state
