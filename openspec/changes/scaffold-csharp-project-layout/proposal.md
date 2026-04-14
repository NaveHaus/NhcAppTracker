## Why

The user-agent MVP needs a buildable C# foundation before foreground-app tracking, persistence, or Windows-specific adapters are added. Scaffolding the project layout as its own increment keeps the later implementation sequence small and prevents the initial architecture from locking the product into Windows-only code paths.

## What Changes

- Add a cross-platform .NET solution and C# project layout for the local user-agent MVP.
- Add host-neutral core and application projects where future attention-tracking abstractions can live without depending on Windows APIs.
- Add a Windows-specific adapter/agent project location for the MVP implementation without making Windows the dependency root.
- Add test project layout under `tests/<category>` so future code changes can follow the required TDD red/green/refactor workflow.
- Add build/test entry points that can run on non-Windows systems while allowing Windows-only projects to be conditionally included or targeted.
- Document the intended incremental implementation sequence so this scaffold remains the first small change rather than a giant user-agent MVP change.
- Do not implement foreground-window capture, idle detection, persistence, categorization UI, installer/startup registration, or `LocalService` infrastructure in this change.

## Capabilities

### New Capabilities
- `csharp-project-layout`: Defines the repository's cross-platform C# solution and project layout for the Windows-only user-agent MVP.

### Modified Capabilities

None.

## Impact

- Adds solution/project files, source directories, and test directories.
- Establishes dependency direction between host-neutral projects and Windows-specific projects.
- Adds or updates developer-facing build documentation/scripts as needed.
- Does not introduce runtime behavior, captured user data, external services, or production dependencies beyond the build/test scaffold.
