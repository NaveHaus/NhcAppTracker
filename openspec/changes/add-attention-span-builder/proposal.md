## Why

The attention domain contracts define observations and completed spans, but the application layer still needs deterministic logic that converts ordered foreground observations into completed attention spans. Adding this logic as a host-neutral increment keeps span construction out of Windows capture, persistence, and runtime sampling code.

## What Changes

- Add application-layer span-building logic that consumes ordered `ForegroundObservation` values and emits completed `AttentionSpan` records.
- Complete a span when the observed app or window context changes.
- Preserve the app identity and window context from the observations that define each completed span.
- Ignore or reject invalid observation sequences according to explicit ordering and timestamp rules.
- Add focused unit tests through the required TDD red/green/refactor workflow.
- Depend on the already-created `add-attention-domain-contracts` change for `ForegroundObservation`, `AttentionSpan`, app identity, and window context contracts.
- Do not add persistence, Windows APIs, foreground-window capture, idle detection, runtime sampling loops, background services, categorization UI, startup registration, telemetry, sync, or `LocalService` infrastructure.

## Capabilities

### New Capabilities
- `attention-span-builder`: Defines host-neutral application logic for converting ordered foreground observations into completed attention spans when application or window context changes.

### Modified Capabilities

None.

## Impact

- Adds application logic and tests in the application layer and matching test project once the C# project layout and attention domain contracts exist.
- Uses domain contracts from `add-attention-domain-contracts` without changing persistence, capture, UI, or platform adapter behavior.
- Establishes deterministic span construction behavior for future runtime sampling and storage changes.
