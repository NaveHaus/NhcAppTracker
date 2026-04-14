## ADDED Requirements

### Requirement: Cross-platform C# solution scaffold
The repository SHALL provide a C# solution scaffold that can be restored, built, and tested through .NET CLI commands from the repository root on supported development platforms.

#### Scenario: Developer builds the scaffold
- **WHEN** a developer runs the documented build command from the repository root with the required .NET SDK installed
- **THEN** the command succeeds without requiring Windows-only shell features

#### Scenario: Developer tests the scaffold
- **WHEN** a developer runs the documented test command from the repository root with the required .NET SDK installed
- **THEN** the command succeeds and discovers the scaffolded test projects

### Requirement: Host-neutral projects avoid Windows dependencies
The scaffold SHALL separate host-neutral code from Windows-specific code by placing domain and application projects in projects that do not reference Windows-specific projects or Windows-only APIs.

#### Scenario: Host-neutral dependency direction is inspected
- **WHEN** the solution project references are inspected
- **THEN** host-neutral projects have no references to Windows-specific agent or platform projects

#### Scenario: Windows-specific projects depend inward
- **WHEN** Windows-specific project references are inspected
- **THEN** Windows-specific projects only depend inward on host-neutral projects or other Windows-specific edge projects

### Requirement: Windows MVP has an isolated project location
The scaffold SHALL provide an isolated project location for the Windows-only user-agent MVP so future foreground-window capture can be implemented without moving Windows API dependencies into host-neutral projects.

#### Scenario: Windows-specific work is added later
- **WHEN** a future change adds Windows foreground-window capture
- **THEN** the implementation has a Windows-specific project location available outside the host-neutral domain and application projects

### Requirement: Tests follow repository layout rules
The scaffold SHALL place test projects under `tests/<category>`, where `<category>` matches the library or tool containing the feature or class under test.

#### Scenario: Test layout is inspected
- **WHEN** the scaffolded test directories are inspected
- **THEN** each test project is located under `tests/<category>` with a category name matching the project under test

### Requirement: Scaffold excludes runtime tracking behavior
The scaffold SHALL NOT implement foreground-window capture, idle detection, attention-span persistence, categorization UI, installer/startup registration, or `LocalService` infrastructure.

#### Scenario: Scaffold scope is reviewed
- **WHEN** the change is reviewed
- **THEN** it contains build/project/test layout only and no runtime app-usage tracking behavior

### Requirement: Incremental implementation sequence is documented
The scaffold change SHALL document the intended follow-on implementation sequence for the user-agent MVP.

#### Scenario: Future work is planned
- **WHEN** a developer reviews the scaffold change artifacts or associated notes
- **THEN** they can identify the next incremental changes after the scaffold without treating the entire user-agent MVP as one giant change
