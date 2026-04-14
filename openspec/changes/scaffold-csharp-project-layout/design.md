## Context

The repository currently needs a buildable foundation for a local user-agent MVP that tracks user attention on foreground apps. The MVP runtime is Windows-only because foreground-window and title capture are platform APIs, but the build system and core abstractions should not make Windows the dependency root.

The prior exploration established that a future `LocalService` service would not replace the per-user agent. Foreground capture remains in the interactive user session. Service-like responsibilities, if added later, would sit behind persistence/coordination boundaries.

## Goals / Non-Goals

**Goals:**
- Establish a buildable C# solution layout for the first implementation increment.
- Keep domain and application projects host-neutral and platform-neutral.
- Isolate Windows-only projects at the edge of the dependency graph.
- Make the default build/test path usable from cross-platform .NET tooling.
- Add test project locations that satisfy the repo rule: tests live under `tests/<category>`.
- Capture the larger user-agent MVP as a sequence of incremental changes without implementing later increments in this change.

**Non-Goals:**
- Implement foreground-window capture, idle detection, session tracking, local persistence, categorization, tray UI, packaging, startup registration, or `LocalService` infrastructure.
- Add external telemetry, network sync, or corporate-control enforcement.
- Decide the future UI architecture.

## Decisions

### Decision: Use a layered C# project layout

Create a solution with host-neutral projects for core model/application code and Windows-specific projects only at the edge.

Candidate layout:

```text
src/
  Nhc.AppTracker.Domain/
  Nhc.AppTracker.Application/
  Nhc.AppTracker.Agent.Windows/
  Nhc.AppTracker.Platform.Windows/
tests/
  Nhc.AppTracker.Domain.Tests/
  Nhc.AppTracker.Application.Tests/
```

Dependency direction:

```text
Windows Agent  ->  Windows Platform Adapter
      |                    |
      v                    v
Application  ----------> Domain
```

Rationale: domain and application code can evolve without depending on Win32 APIs, tray integration, installer behavior, or service hosting. Windows-specific projects are allowed to reference host-neutral projects, but host-neutral projects must not reference Windows-specific projects.

Alternatives considered:
- Single `Agent` project: simpler at first, but it would mix capture, orchestration, and storage concerns and make later extraction to a `LocalService` or non-Windows adapter harder.
- Platform abstraction project first: premature until there is real platform behavior to abstract. The scaffold should leave room for it without inventing broad interfaces before implementation pressure exists.

### Decision: Keep the build entry points based on `dotnet`

Use .NET solution/project files and `dotnet build` / `dotnet test` as the primary build/test interface. Any helper scripts should wrap these commands without becoming required for normal development.

Rationale: `dotnet` is cross-platform and avoids tying the repo to Windows-only shell behavior. This is consistent with a C# project that has a Windows-only runtime target but host-neutral libraries and tests.

Alternatives considered:
- PowerShell-only scripts as the primary build system: acceptable for convenience, but not as the only path because it weakens cross-platform expectations.
- MSBuild custom targets up front: unnecessary until conditional Windows packaging or installer behavior exists.

### Decision: Treat Windows targeting as an edge concern

The Windows agent and Windows platform adapter may target a Windows-specific framework moniker when implementation requires Windows APIs. Host-neutral projects and tests should remain buildable on non-Windows platforms.

Rationale: the MVP is Windows-only at runtime, but compile/test feedback for the core model and orchestration should not require a Windows machine. This also makes later platform expansion a bounded adapter change instead of a domain rewrite.

Alternatives considered:
- Make the entire solution Windows-targeted: simpler solution file, but it creates avoidable Windows lock-in.
- Make the MVP fully cross-platform immediately: over-scopes the first implementation sequence before the Windows capture behavior is proven.

### Decision: Capture the implementation as multiple OpenSpec changes

This change should only scaffold the project layout. Later changes should add behavior in small increments.

Suggested sequence:

1. Scaffold the C# project layout.
2. Add host-neutral domain contracts for foreground observations, attention spans, and tracking gaps.
3. Add local persistence boundaries and an initial local store.
4. Add Windows foreground-window and process-identity capture adapter.
5. Add per-user agent runtime loop and span-building behavior.
6. Add idle/lock/sleep gap accounting.
7. Add user-facing pause/stop/status controls.
8. Add packaging/startup registration for the per-user agent.
9. Consider optional `LocalService` infrastructure only if resilience, multi-user coordination, or maintenance needs justify it.

Rationale: this reduces review size and keeps each test-driven change focused.

Alternatives considered:
- One large user-agent MVP change: faster to describe, but harder to test, review, and correct when platform assumptions change.
- A `LocalService` scaffold first: wrong center of gravity because foreground capture must remain in the interactive user session.

## Risks / Trade-offs

- Windows-only projects can accidentally leak into host-neutral dependencies -> enforce dependency direction through project references and tests.
- A cross-platform build may skip Windows-specific compile errors if conditionalized too aggressively -> keep a documented Windows build/test command for Windows-specific projects once those projects contain runtime code.
- Too much architecture can appear before behavior exists -> keep scaffolding minimal and add abstractions only where they protect known future boundaries.
- Exact .NET SDK/framework choices can become stale -> prefer repository-level SDK/build configuration that is explicit enough for reproducibility but does not depend on local machine paths.

## Migration Plan

This is an initial scaffold, so there is no runtime migration. Rollback is removal of the added solution/project/test scaffold and any associated build documentation.

Future migration toward `LocalService`, if needed, should keep foreground capture in the per-user agent and split persistence/coordination behind the usage-sink boundary described in `docs/notes/localservice-migration-path.md`.

## Open Questions

- Which exact .NET target framework should the scaffold use? The implementation should choose a supported .NET target compatible with the repo's C# 10 requirement and the installed SDK baseline.
- Should the first scaffold include a Windows agent project immediately, or defer it until the first Windows adapter behavior is added? The default assumption is to include the project location now but keep it empty/minimal.
