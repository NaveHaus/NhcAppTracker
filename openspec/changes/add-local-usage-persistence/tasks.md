## 1. Prerequisite And Boundary Checks

- [ ] 1.1 Confirm `add-attention-domain-contracts` has been applied or that the required attention domain contracts exist in `src/Nhc.AppTracker.Domain`.
- [ ] 1.2 Confirm the scaffolded solution contains host-neutral application/domain projects and matching test locations under `tests/<category>`.
- [ ] 1.3 Confirm this change will not add Windows capture, runtime span-building loops, UI, `LocalService`, sync, telemetry, analytics, export, or project/category assignment behavior.

## 2. TDD Application Sink Contract

- [ ] 2.1 RED: using the `tdd` skill, add failing application tests for a host-neutral usage sink contract that accepts foreground observations, attention spans, and tracking gaps from the domain model.
- [ ] 2.2 GREEN: implement the minimal application-level sink/read boundary needed for the contract tests to compile and pass.
- [ ] 2.3 RED: add failing tests that invalid domain values or invalid sink inputs are rejected before persisted records are created.
- [ ] 2.4 GREEN: implement minimal input validation or rely on domain validation so invalid usage values cannot be stored.
- [ ] 2.5 REFACTOR: keep the contract names host-neutral and free of storage-engine, Windows, UI, or project/category assignment concepts.

## 3. TDD Local Persistence Project And Schema

- [ ] 3.1 RED: add failing local persistence tests that opening a new store path creates storage for foreground observations, attention spans, and tracking gaps.
- [ ] 3.2 GREEN: add the local persistence implementation project and minimal storage package/reference needed to create the local schema.
- [ ] 3.3 RED: add failing schema inspection tests proving project, category, assignment, sync, telemetry, analytics, export, and remote endpoint tables are absent.
- [ ] 3.4 GREEN: limit schema creation to foreground observations, attention spans, tracking gaps, and minimal schema metadata if required by the storage engine.
- [ ] 3.5 REFACTOR: keep storage package details inside the local persistence project and out of domain/application projects.

## 4. TDD Usage Round Trips

- [ ] 4.1 RED: add failing tests for writing and reading a foreground observation with UTC timestamp, app identity, window context, opaque source identifiers, and user/session identity.
- [ ] 4.2 GREEN: implement observation persistence and mapping until the round-trip test passes.
- [ ] 4.3 RED: add failing tests proving observations survive closing and reopening the same local store path.
- [ ] 4.4 GREEN: implement durable local observation storage for reopened store instances.
- [ ] 4.5 RED: add failing tests for writing and reading attention spans with UTC start/end, app identity, and window context preserved.
- [ ] 4.6 GREEN: implement attention span persistence and mapping until the round-trip tests pass.
- [ ] 4.7 RED: add failing tests for writing and reading tracking gaps with UTC start/end and constrained gap reasons preserved.
- [ ] 4.8 GREEN: implement tracking gap persistence and mapping until the round-trip tests pass.
- [ ] 4.9 REFACTOR: remove duplicate persistence mapping/test setup while preserving explicit coverage for observations, spans, and gaps.

## 5. TDD Local-Only And Dependency Direction

- [ ] 5.1 RED: add failing tests or inspections proving the local persistence project does not reference Windows-specific projects.
- [ ] 5.2 GREEN: adjust project references so persistence depends only on host-neutral application/domain projects and storage packages.
- [ ] 5.3 RED: add failing tests or inspections proving domain/application projects do not reference the local persistence implementation.
- [ ] 5.4 GREEN: adjust project references so dependency direction remains inward and host-neutral.
- [ ] 5.5 RED: add failing tests or inspections proving persistence has no HTTP client, remote endpoint, telemetry, sync, analytics, export, or network transport dependency used to transmit usage data.
- [ ] 5.6 GREEN: remove or avoid any network/transmission-related dependency or code path from local persistence.

## 6. Final Validation

- [ ] 6.1 Run the focused application and local persistence test projects from the assigned worktree root.
- [ ] 6.2 Run the documented full solution build command from the assigned worktree root.
- [ ] 6.3 Run the documented full solution test command from the assigned worktree root.
- [ ] 6.4 Inspect changed files and verify no generated local database files, secrets, gitignored artifacts, or files outside the assigned worktree are included.
- [ ] 6.5 Run `openspec status --change add-local-usage-persistence` and confirm the change is ready for implementation verification.
