## Context

This change follows `scaffold-csharp-project-layout`. It assumes the scaffolded `Nhc.AppTracker.Domain` project and matching test project exist, but it does not implement Windows capture, storage, or the agent runtime loop.

The product is a strictly local self-tracking and project-management tool. The model must preserve enough foreground-window context, including titles, for a later UI to help users categorize time by project. The model must not require Windows APIs so later platform adapters can translate their observations into the same domain vocabulary.

## Goals / Non-Goals

**Goals:**
- Define host-neutral domain contracts for foreground observations, app identity, window context, attention spans, and tracking gaps.
- Encode simple invariants for UTC time ranges, required identity/context fields, and valid gap reasons.
- Preserve sensitive context such as window title as local domain data without introducing transmission, sync, or analytics behavior.
- Keep platform-specific identifiers opaque so Windows adapters do not leak Win32 concepts into the domain model.
- Add focused unit tests through a TDD red/green/refactor loop.

**Non-Goals:**
- Implement foreground-window capture, Win32 interop, idle/lock detection, span-building algorithms, storage, categorization, startup registration, tray UI, or `LocalService` infrastructure.
- Define project/category assignment behavior.
- Add external dependencies for date/time, persistence, or platform APIs.

## Decisions

### Decision: Model capture input as immutable foreground observations

Define `ForegroundObservation` as a host-neutral snapshot of what the user was attending to at a specific UTC timestamp. It should include app identity, window context, and opaque user/session context where needed.

Candidate shape:

```text
ForegroundObservation
  - ObservedAtUtc
  - AppIdentity
  - WindowContext
  - UserSessionIdentity
```

Rationale: the Windows adapter can produce snapshots without deciding how long the user attended to the app. Span construction can be added later as application logic using the same contracts.

Alternatives considered:
- Model only completed spans: simpler storage shape, but it forces the capture adapter to own duration and gap behavior too early.
- Model raw Windows handles/process IDs directly: useful for capture, but it locks the domain to Windows concepts and makes non-Windows adapters harder.

### Decision: Keep app and session identifiers opaque

Represent platform-specific identity values as strings or small value objects with neutral names, such as `ProcessId`, `ExecutablePath`, `ProcessName`, `DisplayName`, and `UserSessionIdentity`. Avoid Win32-specific type names in the domain layer.

Rationale: the domain needs enough identity information for future categorization and diagnostics, but it should not know whether the source was Win32, a future macOS API, or imported data.

Alternatives considered:
- Use strongly typed Windows identifiers in the domain: clearer for the Windows MVP, but it violates the no-Windows-lock-in constraint.
- Store only a normalized app key: cleaner categorization key, but it throws away source context that may be needed for later rules and debugging.

### Decision: Represent completed attention as validated time ranges

Define `AttentionSpan` as a contiguous interval with a start UTC timestamp, end UTC timestamp, app identity, and window context. Define a shared time-range/value-object invariant where end time must be after start time.

Rationale: categorization will operate on editable time blocks, so the domain should make invalid spans unrepresentable or reject them at construction.

Alternatives considered:
- Store duration only: loses the ability to split, merge, or align spans with clock time later.
- Allow open-ended spans in the domain: useful for an active runtime loop, but that can be represented later by application state rather than persisted completed spans.

### Decision: Represent untracked time explicitly as tracking gaps

Define `TrackingGap` as a time range with a constrained reason, such as paused, stopped, crashed, system sleep, session locked, logged out, or unknown.

Rationale: the product should help users reconstruct their own work without pretending that missing data is tracked attention. Explicit gaps also support later UI behavior and optional service health monitoring.

Alternatives considered:
- Omit gaps and infer missing time from holes: simpler model, but ambiguous when rendering timelines and debugging capture behavior.
- Treat gaps as special attention spans: reduces type count, but mixes "known attention" with "known absence of tracking."

### Decision: Keep privacy as a domain constraint, not a transmission feature

Window titles are part of the domain context because future categorization needs them. This change should not add any external sink, network transmission, sync, telemetry, or analytics behavior.

Rationale: local-only capture is a product boundary. The domain can hold sensitive context while leaving purge/redaction and UI controls to later changes.

Alternatives considered:
- Exclude titles from the domain until the UI exists: safer by default, but it would undermine the stated categorization use case.
- Normalize or redact titles immediately: premature without user-facing rules and may destroy useful categorization context.

## Risks / Trade-offs

- Domain types may overfit the first Windows adapter -> use neutral names and opaque identifiers.
- Window titles are sensitive -> keep this change local-only and avoid any persistence, sync, telemetry, or export behavior.
- Too many value objects can slow implementation -> start with the smallest types needed to enforce time and identity invariants.
- Future categorization may need more context -> preserve raw app/window identity fields instead of reducing observations to a single normalized key.

## Migration Plan

There is no runtime migration. This change adds new domain contracts after the scaffold exists.

If `scaffold-csharp-project-layout` has not been applied yet, implement and archive that change first or apply this change immediately after the scaffold exists in the working tree.

## Open Questions

- Whether to use `DateTimeOffset` directly or wrap it in a small UTC-only value type should be decided during TDD based on the smallest useful invariant.
- The final enum names for gap reasons can be refined during implementation, but the model must distinguish known untracked time from attended app time.
