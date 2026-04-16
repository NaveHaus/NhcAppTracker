## Context

This change follows `add-attention-domain-contracts`, which defines host-neutral foreground observations, attention spans, tracking gaps, app identity, window context, and user/session identity. That change deliberately excludes storage. This change adds the next boundary: a strictly local usage sink that can persist those domain values without adding Windows capture, span-building runtime loops, categorization, UI, sync, telemetry, or `LocalService` behavior.

The scaffolded solution keeps domain and application code host-neutral and isolates Windows-specific work at the edge. Local persistence should follow the same direction: application code owns the sink contract, while the storage implementation sits behind that contract and depends inward on application/domain types.

## Goals / Non-Goals

**Goals:**
- Define an application-level local usage sink boundary for storing and reading foreground observations, attention spans, and tracking gaps.
- Implement an initial strictly local store with deterministic schema creation for development and tests.
- Persist enough app/window/session context to reconstruct observed usage and support future categorization.
- Keep storage local-only, with no network clients, telemetry, sync, analytics, export, or remote service integration.
- Keep schema limited to usage facts and storage metadata; do not add project/category assignment tables.
- Add focused tests through a TDD red/green/refactor loop, including persistence round-trips and schema inspection.

**Non-Goals:**
- Foreground-window capture, idle/lock/sleep detection, span-building runtime loops, project/category assignment, categorization UI, startup registration, installer behavior, `LocalService` infrastructure, purge/redaction UX, cloud sync, telemetry, analytics, or data export.
- Changing the attention domain contract requirements from `add-attention-domain-contracts`.
- Optimizing for high-volume analytics queries before the runtime capture loop exists.

## Decisions

### Decision: Put the sink contract in application code

Define a host-neutral application boundary, such as `IUsageSink` or `ILocalUsageStore`, that accepts domain values for foreground observations, attention spans, and tracking gaps. The contract should expose append/write operations and read/query operations needed by tests and later agent orchestration, using cancellation tokens where asynchronous I/O is used.

Rationale: the runtime agent and future capture adapter should depend on an application boundary rather than on a concrete database package. Keeping the contract in application code preserves dependency direction and makes the local-only guarantee testable at the boundary.

Alternatives considered:
- Put persistence directly in the domain project: rejected because storage is infrastructure behavior and would pollute domain contracts.
- Let the Windows agent write directly to a database: rejected because it would couple capture/runtime behavior to storage and make later testing harder.

### Decision: Add a local persistence implementation project if the scaffold has no suitable edge project

Implement the store in a host-neutral edge project, for example `src/Nhc.AppTracker.Persistence.Local`, with matching tests under `tests/Nhc.AppTracker.Persistence.Local.Tests`. The project may reference `Nhc.AppTracker.Application` and `Nhc.AppTracker.Domain`, but host-neutral domain/application projects must not reference the persistence implementation.

Rationale: a dedicated local persistence project keeps external storage packages and schema code out of domain/application while remaining independent of Windows-specific projects.

Alternatives considered:
- Place the implementation in `Nhc.AppTracker.Application`: simpler initially, but it makes the application layer own storage-package details.
- Reuse a Windows platform project: rejected because local storage is not Windows-specific and should be testable without Windows APIs.

### Decision: Use a file-backed local database with three usage tables

Use a local file-backed database implementation suitable for .NET tests, with a schema limited to:

```text
foreground_observations
attention_spans
tracking_gaps
```

Each table should store the domain time range or timestamp plus serialized scalar columns for app identity, window context, opaque session/source identifiers, and gap reason where applicable. The implementation may include a small schema metadata table if required for migrations/version checks, but it must not create project, category, assignment, sync, telemetry, or remote endpoint tables.

Rationale: observations, spans, and gaps have different semantics and validation rules, so separate tables keep the schema explicit without introducing future categorization concepts prematurely. A local file-backed database supports process restarts and integration tests better than in-memory collections.

Alternatives considered:
- JSON lines files: easy to append, but harder to query, validate schema, and evolve safely.
- A single event table: flexible, but it hides invariants and makes tests less direct for the three required persisted concepts.
- Project/category tables now: rejected because assignment behavior is explicitly out of scope.

### Decision: Treat local-only as a verifiable implementation constraint

The persistence implementation must not reference network client packages, create HTTP clients, open sockets, configure remote endpoints, or expose sync/export behavior. Tests should inspect project references and source/schema shape where practical, alongside functional round-trip tests.

Rationale: "local only" is a product boundary, not just a deployment preference. Making it visible in tests reduces the chance that future implementation adds a remote dependency while building persistence.

Alternatives considered:
- Rely on code review only: insufficient for a hard privacy boundary.
- Add runtime network blocking: premature before there is a runtime host.

## Risks / Trade-offs

- Storage schema may not match future categorization needs -> Persist raw app/window/session context without creating assignment tables, so later categorization can add its own change.
- Database package choice can leak into application code -> Keep package references and schema code in the local persistence implementation project.
- Tests can become brittle if they assert every column name too early -> Assert required tables and forbidden table families, plus round-trip behavior for domain values.
- Local database migrations may be overbuilt -> Start with deterministic schema creation and a minimal schema version marker only if required by the implementation.
- Sensitive window titles are stored locally -> Do not add sync/export/telemetry behavior in this change; future purge/redaction controls should be separate changes.

## Migration Plan

There is no existing runtime usage store to migrate. Implementation should create the local schema when opening a new store path and preserve existing rows when reopening the same path.

Rollback is removal of the local persistence project, tests, project references, and generated local database files used only by tests. Test-created database files must stay under test temporary directories and must not be committed.

## Open Questions

- The exact local database package should be chosen during implementation based on the current solution and available .NET SDK, while preserving the local-only and host-neutral constraints.
- Whether the read API returns all rows or filtered time windows can be decided during TDD; it must at least support persistence round-trip verification and should not add categorization queries yet.
