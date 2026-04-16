## Context

This change follows `add-attention-domain-contracts` and assumes the project layout can keep host-neutral contracts separate from Windows-specific adapters. The backend MVP needs Windows activity signals so later runtime and gap-modeling changes can explain why tracking stopped or why attended time should not be inferred.

Windows exposes the needed signals through several sources rather than one API: idle time from last-input information, session lifecycle notifications for lock/unlock/logout, power notifications for suspend/resume, and process/control notifications for shutdown paths. This change should isolate those APIs behind testable adapter boundaries and translate them into host-neutral events before they cross into application code.

## Goals / Non-Goals

**Goals:**
- Capture Windows idle, active, locked, unlocked, sleep, resume, logout, and shutdown signals.
- Translate raw Windows signals into host-neutral activity events with UTC timestamps and constrained event kinds.
- Keep signal translation deterministic, ordered, and testable using fakes without requiring real OS session transitions in unit tests.
- Make duplicate or repeated raw signals harmless so downstream runtime logic receives stable event streams.
- Use TDD for new code and cover the translation rules and adapter boundaries.

**Non-Goals:**
- Build the user-agent runtime loop, attention span builder, tracking-gap policy, persistence, diagnostics UI, tray UI, startup registration, foreground-window capture, or `LocalService` infrastructure.
- Decide how long an idle period must be before it becomes a tracking gap beyond exposing a configurable threshold or poll input needed to detect idle state.
- Store activity events or transmit them outside the local process.
- Implement cross-platform activity-signal capture.

## Decisions

### Decision: Define a host-neutral activity event boundary

Introduce a small application/domain-facing event contract for activity lifecycle changes, using neutral event kinds such as `BecameIdle`, `BecameActive`, `SessionLocked`, `SessionUnlocked`, `SystemSuspending`, `SystemResumed`, `UserLoggingOut`, and `SystemShuttingDown`.

Rationale: later runtime and gap logic can consume one vocabulary without depending on Win32 session, power, or control-handler details.

Alternatives considered:
- Emit raw Windows notification names: easier initially, but it leaks platform semantics into later changes.
- Directly create `TrackingGap` values in the Windows adapter: premature because gap policy and runtime state belong to later host-neutral application logic.

### Decision: Separate raw signal sources from translation

Model Windows signal capture as small sources for idle polling, session notifications, power notifications, and shutdown/logoff notifications. Feed those raw signals into a translator that emits host-neutral events.

Rationale: the APIs have different subscription and polling models. Keeping capture and translation separate makes unit tests deterministic and avoids requiring real lock, sleep, or shutdown transitions during normal test runs.

Alternatives considered:
- One monolithic Windows watcher: less code, but harder to test and more likely to mix OS interop with application behavior.
- Runtime-only integration tests: useful later, but too slow and brittle as the primary correctness mechanism for this change.

### Decision: Treat timestamps as adapter-observed UTC times

Every emitted host-neutral activity event carries an observed UTC timestamp supplied by an injectable clock or captured at the adapter boundary. The translator must preserve event ordering for equal-source input and reject or normalize non-UTC timestamps before emission.

Rationale: later span and gap logic needs stable time anchors, and tests need deterministic timestamps.

Alternatives considered:
- Use local time to match Windows notifications: easier for debugging, but inconsistent with existing attention contracts.
- Let downstream code timestamp events: loses the actual observation time and makes suspend/resume ordering ambiguous.

### Decision: Detect idle transitions, not continuous idle samples

The idle source may poll Windows last-input information, but the emitted neutral stream should include state transitions only: active to idle and idle to active. Repeated samples that do not change state should not emit duplicate events.

Rationale: downstream code should react to lifecycle changes, not noisy polling samples.

Alternatives considered:
- Emit every idle sample: simpler capture, but creates unnecessary downstream deduplication.
- Leave idle deduplication to the runtime: duplicates are an adapter concern because the adapter controls polling cadence.

### Decision: Keep Windows dependencies quarantined

Windows API calls and any package references needed for them belong only in Windows-specific projects. Host-neutral contracts must not reference Windows Forms UI behavior, WPF UI behavior, `LocalService`, persistence, telemetry, or foreground-window capture.

Rationale: this preserves the backend architecture and keeps the later per-user runtime free to compose adapters without inheriting UI or service infrastructure.

Alternatives considered:
- Use UI framework session events directly from application code: convenient, but it ties activity capture to UI process behavior.
- Add a service now for reliable shutdown notifications: outside scope and contrary to the MVP plan, which explicitly excludes `LocalService`.

## Risks / Trade-offs

- Windows signal availability varies by process type and hosting model -> keep the adapter boundary narrow and document which notifications the implementation subscribes to.
- Shutdown/logoff notifications can have limited execution time -> emit a best-effort event and leave durable flushing/recovery to later runtime and persistence changes.
- Idle detection requires polling last-input state -> make the threshold and polling source testable and avoid emitting duplicate transition events.
- Suspend/resume ordering can be imperfect around clock changes -> use UTC timestamps at observation time and preserve source order when timestamps are equal.
- Unit tests cannot prove real OS notification registration works end to end -> cover translation with unit tests and leave manual/backend hardening verification to later changes.

## Migration Plan

There is no runtime migration. This change adds Windows-specific activity-signal capture and host-neutral event contracts after `add-attention-domain-contracts` is available.

If the scaffolded Windows adapter project does not exist when implementation starts, apply the project-layout change first or add only the minimal Windows-specific project wiring required by the established scaffold.

## Open Questions

- The exact Windows project and namespace names should follow the scaffold once implementation starts.
- The final idle threshold default can be chosen during implementation, but tests should keep it configurable.
- The implementation may need to choose between direct Win32 interop and available .NET/Windows notification wrappers based on the scaffolded target framework.
