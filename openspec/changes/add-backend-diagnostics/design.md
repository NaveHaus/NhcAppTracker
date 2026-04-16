## Context

The backend MVP already has local dependency artifacts for the per-user agent runtime and local usage persistence. The runtime emits structured diagnostics and maintains orchestration state; the persistence boundary stores foreground observations, completed attention spans, and tracking gaps. This change adds a backend-only way to inspect those records and runtime health before a front-end timeline UI exists.

Diagnostics must remain local and must not become a remote monitoring feature. The inspection paths are developer-facing or backend-facing verification surfaces for this phase, not product UI, categorization, export, sync, telemetry, or `LocalService` behavior.

## Goals / Non-Goals

**Goals:**

- Provide local inspection of recent foreground observations, completed attention spans, and tracking gaps.
- Provide local inspection of current per-user runtime state, including lifecycle, sampling, pending observation, pending gap, and recent diagnostic events.
- Provide local health reporting for the runtime and its backend dependencies.
- Keep inspection deterministic in tests with fake stores, fake runtime state providers, fake clocks, and in-process diagnostic collectors.
- Keep all diagnostics and inspection output local, privacy-aware, and free of remote transmission.
- Add focused TDD tasks that cover query behavior, boundary constraints, health state, and local-only guarantees.

**Non-Goals:**

- No tray UI, timeline UI, web UI, or user-facing controls.
- No project/category assignment, categorization rules, visual editing, purge workflow, export workflow, or sync workflow.
- No startup registration, installer behavior, Windows service, or `LocalService` infrastructure.
- No remote diagnostics, telemetry upload, analytics clients, HTTP endpoints, sockets, cloud sync, or external network transmission.
- No restart/crash recovery changes beyond reporting the state already exposed by the runtime.
- No changes to the storage schema beyond read/query support needed by local inspection.

## Decisions

### Decision: Use Application-Level Query Contracts For Inspection

Define host-neutral query contracts for backend diagnostics, such as recent observations, recent spans, recent gaps, runtime state snapshot, health snapshot, and recent structured diagnostics. Implementations should compose the local usage persistence read API and the user-agent runtime's in-process state/diagnostic providers.

Rationale: query contracts keep diagnostics testable without binding the implementation to console output, Windows APIs, or a future UI. They also give the later front end a stable backend boundary without forcing front-end behavior into this change.

Alternatives considered:
- Read the database directly from a command: rejected because it bypasses application boundaries and duplicates persistence mapping.
- Add UI screens now: rejected because the backend MVP needs verification surfaces without committing to timeline UX.

### Decision: Expose Developer-Facing Local Commands As A Thin Shell

If the solution has or adds a per-user agent executable, add local command handlers or CLI commands as a thin shell over the query contracts. Commands should support bounded queries, such as recent observations, spans, gaps, health, current state, and diagnostics, and should print structured local output suitable for manual verification.

Rationale: a thin command layer provides practical inspection without becoming a product UI. Tests can focus on the query handlers and output boundaries while the command shell remains small.

Alternatives considered:
- Only expose in-process services: lower implementation cost, but weaker manual verification before the UI exists.
- Add a local HTTP endpoint: rejected because it creates unnecessary network surface area and conflicts with the local-only diagnostics requirement.

### Decision: Return Bounded, Privacy-Aware Inspection Results

Inspection queries should require explicit bounds, such as a limit and/or time window, and should use stable DTOs. Results may include locally captured app identity and window title where needed to verify capture correctness, but output must remain local and must not redact by transmitting elsewhere or by logging to remote sinks.

Rationale: bounded queries avoid accidentally dumping large local histories while still allowing backend verification. Stable DTOs also prevent command output from depending on internal persistence entities.

Alternatives considered:
- Return all history by default: rejected because it is poor verification ergonomics and increases accidental disclosure risk.
- Always redact window titles: rejected for this backend verification phase because title capture correctness must be inspectable locally.

### Decision: Health Is A Snapshot, Not A Watchdog

Health inspection should report current backend status from existing runtime and persistence dependencies, including lifecycle state, last successful sample, last accepted write, last diagnostics event, sampling enabled/paused state, and dependency availability where observable. It should not restart the agent, register startup tasks, or repair persistence.

Rationale: this change is for inspection. Recovery, startup registration, and repair workflows are separate changes with different risk profiles.

Alternatives considered:
- Add watchdog or auto-repair behavior: rejected because it changes runtime control semantics and belongs in hardening or recovery work.

### Decision: Local-Only Is Enforced In Design And Tests

Tests should verify the diagnostics implementation and command layer do not introduce HTTP clients, sockets, telemetry clients, analytics clients, sync clients, remote endpoints, or network transmission of usage data. Source/reference inspection tests are acceptable alongside functional tests.

Rationale: the local-only boundary is a core product constraint and must be visible at implementation time, not left to manual review.

Alternatives considered:
- Rely on code review only: rejected because the same local-only constraint is already treated as testable in persistence and runtime changes.

## Risks / Trade-offs

- Query contracts may duplicate future UI needs -> Keep DTOs focused on backend records and snapshots, not timeline editing or categorization.
- Command output can expose sensitive window titles locally -> Require explicit local invocation and bounded results; do not transmit output to remote sinks.
- Runtime state can change while inspection runs -> Return point-in-time snapshots with timestamps instead of trying to provide consistency across all stores.
- Persistence read APIs may be too limited -> Extend only the read methods needed for bounded recent-record inspection.
- Health checks can become control behavior -> Keep health reporting read-only and do not add restart, repair, or startup registration behavior.
- Local-only source inspections can be brittle -> Assert forbidden dependency families and network primitives at project/source boundaries instead of exact implementation layout.
