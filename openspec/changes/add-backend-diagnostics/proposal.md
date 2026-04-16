## Why

The backend MVP needs local inspection paths before a front-end UI exists, so captured observations, completed spans, gaps, runtime health, and current agent state can be verified directly. This change adds backend-only diagnostics access while preserving the local-only privacy boundary and avoiding any network transmission of captured usage data.

## What Changes

- Add backend-only inspection paths for recent foreground observations, completed attention spans, and tracking gaps from the local usage store.
- Add inspection of current per-user agent runtime state, including lifecycle status, sampling state, pending observation or gap state, and recent diagnostic events.
- Add local health reporting for runtime dependencies such as foreground capture, activity signals, local persistence, scheduler, and diagnostics sink.
- Provide deterministic local interfaces and developer-facing command or query handlers that can be tested without UI, Windows session changes, disk writes beyond test stores, or network access.
- Keep diagnostics and inspection strictly local, with no HTTP clients, sockets, telemetry upload, analytics, sync, remote endpoints, tray UI, timeline UI, categorization UI, or project/category assignment.
- Add tests through a TDD red/green/refactor workflow for inspection queries, health state, local-only boundaries, privacy-safe output, and runtime-state snapshots.

## Capabilities

### New Capabilities
- `backend-diagnostics`: Backend-only local inspection of recent observations, spans, gaps, runtime health, current state, and structured diagnostics without a front-end UI or network transmission.

### Modified Capabilities

None.

## Impact

- Adds application-level diagnostics query contracts and/or developer-facing local commands that depend on `add-user-agent-runtime` state and `add-local-usage-persistence` read APIs.
- May add a backend diagnostics project or extend the per-user agent host with local inspection handlers, while keeping host-neutral contracts free of Windows-specific implementation references.
- Adds tests under the relevant diagnostics or runtime test category using fake usage stores, fake runtime state providers, fake clocks, and in-process diagnostic collectors.
- Preserves backend scope by excluding UI, startup registration changes, project/category assignment, export, sync, telemetry, external analytics, remote diagnostics, and `LocalService` behavior.
