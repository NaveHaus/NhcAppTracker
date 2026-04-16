## Context

The copied dependency artifacts define a per-user headless runtime that samples foreground context, converts activity signals into spans and gaps, persists local usage records, and performs controlled shutdown without fabricating final spans. They also define local persistence for foreground observations, completed attention spans, and tracking gaps.

This change covers what happens before the runtime resumes normal sampling after a crash, reboot, process kill, or agent restart. Recovery must use only local durable facts and explicit runtime checkpoint metadata. It cannot infer attended time from wall-clock elapsed time alone, because the agent was unavailable during that interval.

## Goals / Non-Goals

**Goals:**

- Run deterministic recovery before normal runtime sampling resumes.
- Persist runtime recovery checkpoints that describe pending foreground context, pending untracked ranges, last clean shutdown state, and runtime identity.
- Detect unclean termination and distinguish it from clean controlled shutdown.
- Reconcile durable local observations, completed spans, tracking gaps, and recovery checkpoints without duplicating records.
- Record unknown gaps for time ranges where the agent was unavailable and no specific reason is known.
- Mark or close open runtime state without creating attended foreground spans from unavailable time.
- Emit structured local diagnostics for every recovery decision.
- Make recovery fully testable with fake clocks, fake checkpoint storage, fake local usage storage, fake boot/session identity, and diagnostic collectors.

**Non-Goals:**

- No timeline UI, tray UI, manual repair UI, or user-facing conflict resolution.
- No startup registration, installer behavior, Windows service, or `LocalService` implementation.
- No project/category assignment, categorization rules, sync, telemetry upload, external analytics, export, purge, or remote diagnostics.
- No reconstruction of precise user attention while the agent was not running.
- No mutation of unrelated durable records except recovery-owned checkpoint state and recovery-required gap/span status fields.

## Decisions

### Decision: Recovery Runs Before Runtime Sampling

The per-user agent host should invoke a recovery coordinator immediately after constructing local persistence and diagnostics, and before starting foreground sampling or activity subscription. Normal runtime orchestration starts only after recovery reports a completed or failed outcome.

Rationale: sampling before recovery can interleave new observations with unreconciled pending state, making crash gaps and open spans nondeterministic.

Alternative considered: let the runtime loop recover lazily on the first sample. That couples recovery decisions to sampling cadence and makes tests dependent on scheduler timing.

### Decision: Store A Local Runtime Recovery Checkpoint

The runtime should persist a small local checkpoint record whenever recovery-relevant state changes. The checkpoint should include a stable runtime instance identifier, last heartbeat timestamp, last accepted observation reference or value, pending untracked range start and reason, clean shutdown marker, shutdown timestamp when known, process/boot/session identity when available, and schema version.

Rationale: observations, spans, and gaps are usage facts, but they do not fully describe whether the runtime exited cleanly or had an open pending range. A checkpoint makes recovery explicit without changing usage facts into process-state records.

Alternatives considered:
- Infer everything from the latest usage rows. Rejected because the latest observation does not prove whether shutdown was clean, whether a gap was pending, or whether a crash occurred after a write.
- Persist process state in every usage row. Rejected because it mixes runtime metadata into domain usage records and increases duplication.

### Decision: Treat Unclean Downtime As Unknown Gap Unless A Stronger Reason Exists

When the previous runtime did not record a clean shutdown, recovery should compute the unavailable interval from the last durable recovery timestamp to the current recovery timestamp or a reliable restart boundary. If a pending untracked range already has a constrained reason, recovery should complete or mark that gap with that reason when the end is known. Otherwise it should persist an unknown tracking gap for the unavailable interval.

Rationale: unknown gaps make unavailable tracking explicit without pretending that the foreground app remained attended.

Alternative considered: close the last pending foreground context at restart time. Rejected because it fabricates attended time across crash, reboot, or process downtime.

### Decision: Recovery Is Idempotent

Recovery should write records with deterministic recovery identifiers or deduplication keys derived from the previous runtime instance, checkpoint version, recovered range, and recovery reason. Re-running recovery over the same durable state must not create duplicate gaps, duplicate status updates, or duplicate diagnostics that imply new data repair.

Rationale: recovery itself can be interrupted. Idempotence keeps repeated startup attempts safe.

Alternative considered: append every recovery attempt as a new gap. Rejected because repeated failed restarts would inflate unavailable time.

### Decision: Open State Is Marked With Provenance Instead Of Fabricated Completion

If the implementation needs to represent an abandoned pending foreground context or pending gap, it should use recovery provenance such as `RecoveryAbandoned`, `RecoveredUnknownGap`, or equivalent status metadata. It must not create a completed attention span unless there is a later valid observation boundary captured while the runtime was available.

Rationale: abandoned pending state is operational metadata, not evidence of attended use.

Alternative considered: write zero-duration spans or open-ended gaps. Rejected because zero-duration spans are misleading and open-ended usage records make later queries ambiguous.

### Decision: Recovery Failures Stop Normal Runtime Start

If recovery cannot read checkpoint state, cannot inspect required local records, or cannot persist required recovery output, the agent should emit an error diagnostic and fail startup or enter a non-sampling degraded state. It must not begin attended-time sampling on top of unreconciled state.

Rationale: continuing after partial recovery risks mixing new records with ambiguous old state.

Alternative considered: log and continue. Rejected for deterministic recovery because the system would become dependent on manual inspection.

## Risks / Trade-offs

- Checkpoint write can fail after usage writes -> emit diagnostics and stop or degrade deterministically; do not report a clean checkpoint.
- Crash during recovery can leave partial recovery output -> use deterministic ids and idempotent upserts for recovery-created gaps and status updates.
- Boot/session identity may be unavailable or platform-specific -> keep it optional metadata at the host edge and base correctness on timestamps plus clean shutdown markers.
- Local clock changes can make ranges non-increasing -> reject invalid recovery ranges, emit diagnostics, and avoid fabricated gaps.
- Unknown gaps may be conservative -> prefer explicit unavailable time over invented attended time.
- Adding checkpoint persistence expands local schema -> keep it local-only, recovery-scoped, and free of project/category, sync, telemetry, or network concepts.

## Migration Plan

There is no existing runtime recovery checkpoint. On first startup after this change, recovery should initialize recovery metadata and treat the absence of a checkpoint as "no prior runtime state" unless existing local usage records prove a prior runtime session that needs reconciliation.

Rollback removes the recovery coordinator, checkpoint metadata, tests, and host wiring. Existing usage records and recovery-created unknown gaps remain valid local usage facts; schema cleanup can be handled by a later explicit migration if needed.

## Open Questions

- Whether recovery checkpoint metadata belongs in the same local persistence implementation as usage records or a separate local runtime-state store can be decided during implementation, as long as dependency direction and local-only constraints hold.
- The exact status field names for abandoned pending state depend on the domain and persistence shape produced by dependency implementation.
