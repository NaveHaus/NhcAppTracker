## 1. Diagnostics Query Contracts

- [ ] 1.1 RED: Add unit tests for bounded recent-observation diagnostics queries, including explicit limit/time-window behavior and rejection or conservative defaulting for unbounded requests.
- [ ] 1.2 GREEN: Define host-neutral diagnostics query contracts and DTOs for recent foreground observations over the local usage persistence read boundary.
- [ ] 1.3 REFACTOR: Keep observation diagnostics contracts free of Windows-specific APIs, persistence implementation types, UI references, and network-related dependencies.
- [ ] 1.4 RED: Add unit tests for bounded recent-span and recent-gap diagnostics queries, including preservation of UTC ranges, app/window context, and constrained gap reasons.
- [ ] 1.5 GREEN: Implement recent attention-span and tracking-gap diagnostics query handlers over the local usage persistence read boundary.
- [ ] 1.6 REFACTOR: Consolidate shared query-bound validation so observation, span, gap, and diagnostic-event queries apply consistent limits and diagnostics.

## 2. Runtime State And Health Inspection

- [ ] 2.1 RED: Add unit tests for current runtime-state snapshots covering lifecycle state, sampling state, attended-time tracking state, pending observation presence, pending gap presence, last successful sample timestamp, last accepted persistence timestamp, and snapshot UTC timestamp.
- [ ] 2.2 GREEN: Add a runtime-state provider contract and snapshot implementation that reads current user-agent runtime state without mutating runtime or persistence.
- [ ] 2.3 REFACTOR: Ensure runtime-state snapshots are point-in-time or internally consistent best-effort values with a clear snapshot timestamp.
- [ ] 2.4 RED: Add unit tests for backend health snapshots covering runtime, foreground capture dependency, activity signal dependency, local usage persistence dependency, scheduler, diagnostic sink, and recent failure summaries.
- [ ] 2.5 GREEN: Implement local health query handling with stable status values and component-level recent failure summaries.
- [ ] 2.6 REFACTOR: Keep health inspection read-only and separate from watchdog, restart, repair, startup registration, and recovery behavior.

## 3. Structured Diagnostic Event Inspection

- [ ] 3.1 RED: Add unit tests for bounded recent structured diagnostic event queries, including event name, UTC timestamp, severity, component, runtime state, correlation fields, and failure details.
- [ ] 3.2 GREEN: Implement a local diagnostic-event inspection provider or query handler over the runtime diagnostics collector or local diagnostic sink.
- [ ] 3.3 REFACTOR: Normalize diagnostic event DTOs so tests and local commands do not depend on internal logging or persistence implementation entities.
- [ ] 3.4 RED: Add tests that unbounded diagnostic-event inspection is rejected or conservatively bounded and emits a local diagnostic describing the applied bound.
- [ ] 3.5 GREEN: Apply bounded-query validation to diagnostic-event inspection.

## 4. Developer-Facing Local Inspection Commands

- [ ] 4.1 RED: Add command or handler tests proving local observations, spans, gaps, health, current-state, and diagnostics inspection delegate to diagnostics query contracts rather than directly reading storage tables.
- [ ] 4.2 GREEN: Add thin developer-facing local command or handler paths for recent observations, recent spans, recent gaps, health, current state, and diagnostic events.
- [ ] 4.3 REFACTOR: Keep command rendering local, deterministic, and small, with no runtime mutation and no categorization, editing, export, sync, telemetry, or analytics behavior.
- [ ] 4.4 RED: Add tests that local health/current-state commands do not start, stop, restart, repair, or reconfigure the runtime.
- [ ] 4.5 GREEN: Ensure command handlers are read-only adapters over query contracts.

## 5. Local-Only And Boundary Verification

- [ ] 5.1 RED: Add source or project-reference inspection tests that fail if diagnostics code introduces HTTP clients, sockets, telemetry clients, sync clients, analytics clients, remote endpoints, UI frameworks, startup registration, installer behavior, or `LocalService` infrastructure.
- [ ] 5.2 GREEN: Keep diagnostics implementation references limited to host-neutral application contracts, runtime state providers, local usage persistence read boundaries, and executable or adapter edge composition where concrete dependencies are required.
- [ ] 5.3 RED: Add tests that captured window titles and app identity appear only in explicit local diagnostics responses or command output and are never sent to remote sinks.
- [ ] 5.4 GREEN: Route diagnostics output only through return values, console output, local files, in-process collectors, or local store reads.
- [ ] 5.5 REFACTOR: Review dependency direction so host-neutral diagnostics contracts do not reference Windows-specific projects, local persistence implementation packages, network clients, telemetry clients, UI references, or service infrastructure.

## 6. End-To-End Verification

- [ ] 6.1 Run the diagnostics unit test suite and fix failures using the TDD red/green/refactor loop.
- [ ] 6.2 Run the relevant runtime and local persistence test suites to verify diagnostics integration does not regress dependency changes.
- [ ] 6.3 Run the repository build command for the assigned worktree.
- [ ] 6.4 Run the repository test command for the assigned worktree.
- [ ] 6.5 Run `openspec status --change "add-backend-diagnostics"` and confirm the change is ready for implementation.
