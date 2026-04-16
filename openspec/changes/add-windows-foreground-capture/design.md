## Context

This change follows the C# project-layout scaffold and depends on the attention-domain contracts change. The Windows adapter belongs at the platform edge, where Windows-only APIs are allowed to observe the interactive user's current foreground window and owning process. Host-neutral domain and application code must remain free of Win32 API references.

The adapter produces `ForegroundObservation` values. It does not decide how long attention lasted, build `AttentionSpan` values, persist observations, categorize apps, detect idle/lock state, host a tray UI, register startup behavior, or introduce `LocalService` infrastructure.

## Goals / Non-Goals

**Goals:**
- Capture the current Windows foreground window handle, title, owning process ID, process name, executable path/name, and available app metadata.
- Map Windows capture data into host-neutral `ForegroundObservation`, `AppIdentity`, `WindowContext`, and `UserSessionIdentity` values.
- Keep Win32 handles, process IDs, and other Windows-specific identifiers out of host-neutral project APIs except as opaque source identifiers accepted by the domain contracts.
- Add TDD coverage for mapping behavior and capture failure handling before implementation.

**Non-Goals:**
- Attention-span construction or coalescing observations into time ranges.
- Persistence, analytics, sync, telemetry, or external transmission of captured titles/process data.
- Idle detection, lock/logoff detection, service hosting, tray UI, installer/startup registration, or categorization behavior.
- Changing the attention-domain contract requirements introduced by `add-attention-domain-contracts`.

## Decisions

### Decision: Split native capture from domain translation

Represent native capture as a Windows-edge snapshot type that contains raw foreground-window and process data. Use a separate mapper to translate that snapshot into a domain `ForegroundObservation`.

Rationale: this keeps Win32 interop and process/file metadata lookup testable without requiring every domain-mapping test to call live Windows APIs. It also prevents the adapter from growing span-building or persistence responsibilities.

Alternatives considered:
- Return `ForegroundObservation` directly from every Win32 call path: simpler surface, but it couples native error handling to domain mapping and makes edge cases harder to test.
- Put Win32 handles/process IDs directly into the domain model: easier debugging, but it violates the platform-neutral contract boundary.

### Decision: Treat process and window identifiers as adapter data or opaque source identifiers

The adapter may capture process ID, window handle, executable path, and executable name for diagnostics and mapping, but the domain observation receives only supported identity fields and opaque source identifiers.

Rationale: the dependency change already defines app identity as host-neutral data. Preserving Windows IDs as opaque source values allows later troubleshooting or categorization without forcing Win32 types into the domain.

Alternatives considered:
- Drop process ID/window handle entirely after mapping: reduces sensitive data, but loses useful source identity that can distinguish apps with ambiguous names.
- Add Windows-specific domain fields: useful for the MVP only, but it creates Windows lock-in.

### Decision: Metadata lookup is best-effort

Executable file metadata such as display name, product name, or file description should be captured when available, but missing or inaccessible metadata must not prevent an observation when process identity and window context are otherwise valid.

Rationale: Windows processes can exit between handle lookup and metadata reads, run with restricted access, or lack version metadata. The adapter should still produce the richest valid observation it can.

Alternatives considered:
- Require executable path and metadata for every observation: precise when available, but brittle for protected or short-lived processes.
- Capture only process name and title: robust, but too weak for later categorization and app identity matching.

### Decision: Surface capture absence explicitly

When no foreground window is available, the owning process cannot be resolved, or required domain values cannot be created, the adapter should return a no-observation result rather than persisting a placeholder or fabricating attention data.

Rationale: missing foreground context is not attended time. Later runtime logic can decide whether to turn repeated capture absence into tracking gaps.

Alternatives considered:
- Emit an observation with unknown app/window values: keeps a continuous stream, but pollutes attention data and hides capture failures.
- Build tracking gaps inside the adapter: useful later, but span/gap construction is outside this change.

## Risks / Trade-offs

- Protected, elevated, or short-lived processes may deny path/metadata access -> keep metadata best-effort and test fallback mapping.
- Window titles may contain sensitive local context -> keep data local, avoid persistence/transmission in this change, and do not add logging of captured titles.
- Win32 interop is hard to exercise in deterministic unit tests -> separate native capture from mapping and cover interop through narrow adapter tests where practical.
- Domain contracts are being developed in a dependent unarchived change -> implement against the accepted contract names and adjust during application if the dependency lands with minor API differences.

## Migration Plan

1. Add focused failing tests for Windows snapshot-to-domain mapping and capture failure cases.
2. Implement the minimal Windows-edge capture and mapping code needed to pass the tests.
3. Refactor interop boundaries so host-neutral projects do not reference Windows-specific APIs.
4. Run the documented build and test commands for the solution, including Windows platform adapter tests.

Rollback is file-level removal of the Windows adapter implementation and tests because this change introduces no persistence schema, external service, or user data migration.

## Open Questions

- Which exact domain API names will be available after `add-attention-domain-contracts` is applied and archived?
- Should the implementation use `System.Diagnostics.Process` metadata only, or also read executable version info through `FileVersionInfo` for display/product fields?
