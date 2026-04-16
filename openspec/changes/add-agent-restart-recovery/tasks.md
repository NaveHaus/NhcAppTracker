## 1. Recovery Test Harness

- [ ] 1.1 Create or update the recovery/runtime test project under `tests/<runtime-or-recovery-project-name>` for deterministic recovery tests.
- [ ] 1.2 Add fake local usage persistence, fake recovery checkpoint storage, fake clock, fake runtime identity, fake platform restart facts, and fake diagnostic sink test helpers.
- [ ] 1.3 Add RED tests proving agent startup runs recovery before sampling or activity subscription begins.
- [ ] 1.4 Add RED tests proving recovery failure prevents attended-time sampling and emits structured error diagnostics.

## 2. Recovery Checkpoint Contract

- [ ] 2.1 Add RED tests for persisting pending foreground context, pending untracked range, clean shutdown marker, runtime instance identity, checkpoint version, and local-only behavior.
- [ ] 2.2 Implement host-neutral recovery checkpoint models and store boundary in application/runtime code.
- [ ] 2.3 Implement local checkpoint persistence or extend the local usage persistence implementation with recovery-scoped metadata.
- [ ] 2.4 Refactor checkpoint tests and implementation to keep domain/application code free of persistence package details and Windows APIs.

## 3. Startup Recovery Coordinator

- [ ] 3.1 Add RED tests for clean prior shutdown, missing checkpoint with no prior usage, unclean crash, reboot restart classification, and agent restart classification.
- [ ] 3.2 Implement the recovery coordinator that reads checkpoint state, local usage state, current clock, runtime identity, and optional platform restart facts.
- [ ] 3.3 Wire the per-user agent host so recovery completes before normal foreground sampling or activity subscriptions start.
- [ ] 3.4 Refactor startup wiring to keep Windows-specific boot, process, or session facts behind injectable host-edge ports.

## 4. Unknown Gap Recovery

- [ ] 4.1 Add RED tests for persisting an unknown unavailable-tracking gap after unclean termination when no stronger pending reason applies.
- [ ] 4.2 Add RED tests proving pending foreground context is not extended through crash, reboot, restart, or unavailable agent time.
- [ ] 4.3 Implement unknown gap creation through the local usage sink with deterministic recovery provenance.
- [ ] 4.4 Add RED tests proving no open-ended or guessed gap is fabricated when the unavailable range start cannot be identified.

## 5. Pending Range Recovery

- [ ] 5.1 Add RED tests for recovering pending locked, idle, sleep, stopped, logout, or shutdown ranges with constrained gap reasons.
- [ ] 5.2 Add RED tests for rejecting pending range recovery when the recovery end timestamp is not strictly after the start timestamp.
- [ ] 5.3 Implement pending untracked range completion using existing tracking gap validation and local persistence.
- [ ] 5.4 Refactor reason mapping so recovery uses the same constrained gap reason vocabulary as the runtime gap model.

## 6. Abandoned Pending State

- [ ] 6.1 Add RED tests proving pending pre-crash foreground context is marked abandoned or superseded without creating an attended span.
- [ ] 6.2 Add RED tests proving the first post-recovery observation becomes a new pending context and does not complete a pre-crash span.
- [ ] 6.3 Implement recovery status metadata for abandoned or superseded pending state where supported by the persistence shape.
- [ ] 6.4 Emit structured diagnostics for abandoned state, skipped span fabrication, and post-recovery context reset.

## 7. Idempotent Recovery

- [ ] 7.1 Add RED tests proving repeated recovery for the same runtime instance, checkpoint version, range, and reason creates exactly one recovery gap or status update.
- [ ] 7.2 Add RED tests proving recovery can resume after a crash during recovery without duplicating prior successful writes.
- [ ] 7.3 Implement deterministic recovery identifiers or upsert keys for recovery-created gaps, status updates, and checkpoint transitions.
- [ ] 7.4 Emit diagnostics that distinguish idempotent skipped writes from newly persisted recovery facts.

## 8. Local Diagnostics And Boundaries

- [ ] 8.1 Add RED tests for structured diagnostics covering recovery lifecycle, classification, repair decisions, skipped fabrication, persistence outcomes, idempotence, and failures.
- [ ] 8.2 Add RED tests or inspections proving recovery code has no UI, project/category assignment, startup registration, network sync, telemetry upload, external analytics, remote diagnostics, or `LocalService` behavior.
- [ ] 8.3 Add RED tests or inspections proving host-neutral recovery projects have no Windows-only APIs, UI framework references, Windows adapter references, network clients, telemetry clients, sync clients, analytics clients, or service infrastructure.
- [ ] 8.4 Implement local diagnostic event names and payload fields with stable UTC timestamps, severity, runtime identifiers where available, and recovery state.

## 9. Verification

- [ ] 9.1 Run the focused recovery/runtime unit tests and confirm they fail before implementation and pass after each implementation slice.
- [ ] 9.2 Run the relevant integration tests for local persistence and runtime startup recovery.
- [ ] 9.3 Run the repository build command and full test command required by the worktree.
- [ ] 9.4 Run `openspec status --change "add-agent-restart-recovery"` and confirm the change is ready for implementation.
