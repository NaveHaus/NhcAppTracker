## 1. Solution And Project Scaffold

- [ ] 1.1 Choose and document the .NET target framework and explicit C# 10 language-version configuration for the scaffold.
- [ ] 1.2 Create the solution and host-neutral `src/Nhc.AppTracker.Domain` and `src/Nhc.AppTracker.Application` projects.
- [ ] 1.3 Create the Windows-edge `src/Nhc.AppTracker.Agent.Windows` and `src/Nhc.AppTracker.Platform.Windows` project locations with references that point inward to host-neutral projects only.
- [ ] 1.4 Add cross-platform `dotnet` build/test entry point documentation and avoid making Windows-only shell scripts required for normal development.

## 2. TDD Scaffold Verification

- [ ] 2.1 RED: using the `tdd` skill, add scaffold verification tests under `tests/Nhc.AppTracker.Domain.Tests` and `tests/Nhc.AppTracker.Application.Tests` that fail until the host-neutral assemblies and references exist.
- [ ] 2.2 GREEN: add only the minimal host-neutral assembly markers or equivalent scaffold code needed for the new tests to pass.
- [ ] 2.3 REFACTOR: remove unnecessary placeholder code and confirm host-neutral projects do not reference Windows-specific projects or Windows-only APIs.

## 3. Incremental Plan Documentation

- [ ] 3.1 Document the intended follow-on implementation sequence for the user-agent MVP in a developer-facing markdown note or README section.
- [ ] 3.2 Confirm the documented sequence keeps foreground capture in the per-user agent and defers optional `LocalService` infrastructure until a later justified change.

## 4. Validation

- [ ] 4.1 Run the documented restore/build/test commands from the repository root.
- [ ] 4.2 Inspect solution and project references to verify host-neutral projects have no Windows-specific dependencies and Windows-specific projects depend inward.
- [ ] 4.3 Verify the change contains no runtime implementation for foreground-window capture, idle detection, attention-span persistence, categorization UI, installer/startup registration, or `LocalService` infrastructure.
