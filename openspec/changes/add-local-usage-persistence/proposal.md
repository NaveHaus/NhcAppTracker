## Why

The tracker needs a local sink boundary so captured attention data can survive process restarts without introducing sync, telemetry, or project-assignment behavior. This change builds on the attention domain contracts by defining strictly local persistence for observations, attention spans, and tracking gaps.

## What Changes

- Add a local usage persistence capability for saving and reading foreground observations, completed attention spans, and tracking gaps.
- Define a storage boundary that depends on host-neutral attention-domain contracts and does not require Windows capture or runtime agent behavior.
- Keep persisted usage data strictly local, with no network transmission, telemetry, sync, analytics, export, or remote service behavior.
- Preserve captured app/window context, including window titles, as local data for later user-controlled categorization.
- Avoid project/category assignment tables and relationships unless a storage engine requires a minimal schema compatibility artifact.
- Add tests through a TDD red/green/refactor workflow for persistence behavior, local-only boundaries, and schema shape.

## Capabilities

### New Capabilities
- `local-usage-persistence`: Defines strictly local persistence for foreground observations, attention spans, and tracking gaps using the attention domain contracts.

### Modified Capabilities

None.

## Impact

- Adds a local persistence boundary and storage implementation under application/infrastructure code, plus matching unit or integration tests.
- Depends on `add-attention-domain-contracts` for the domain types that are persisted.
- May add a local storage package or schema migration support if needed by the selected implementation.
- Does not add Windows foreground capture, span-building runtime loops, project/category assignment, UI, sync, telemetry, analytics, external network calls, or `LocalService` behavior.
