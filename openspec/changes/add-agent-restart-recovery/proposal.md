## Why

The per-user backend agent can be interrupted by crash, reboot, process kill, or agent restart after it has already persisted observations, spans, gaps, or pending runtime state. Recovery must reconcile those durable facts deterministically, close or mark open ranges correctly, and record unknown gaps where tracking was unavailable without inventing attended time.

## What Changes

- Add deterministic startup recovery for the per-user agent runtime after crash, reboot, or restart.
- Persist enough runtime checkpoint state to distinguish pending foreground context, pending untracked ranges, controlled shutdown, and unclean termination.
- Reconcile open observations, spans, and gaps using durable local usage records and recovery checkpoints.
- Record explicit unknown tracking gaps for intervals where the agent was unavailable and the reason cannot be proven.
- Close or mark open spans and gaps using constrained reasons and provenance instead of fabricating attended foreground time.
- Emit structured local diagnostics for recovery start, recovery decisions, repaired records, unknown gaps, skipped fabrication, and recovery completion.
- Keep recovery backend-only and local-only with no UI, startup registration, sync, telemetry upload, external service, or `LocalService` behavior.
- Add tests through a TDD red/green/refactor workflow for clean restart, crash restart, reboot restart, pending spans, pending gaps, unknown gaps, idempotence, and persistence failures.

## Capabilities

### New Capabilities
- `agent-restart-recovery`: Deterministic backend recovery for unclean runtime termination, reboot, and agent restart using local persisted usage records, runtime checkpoints, unknown gaps, and structured diagnostics.

### Modified Capabilities

## Impact

- Adds application-level recovery orchestration used by the per-user agent runtime before normal sampling resumes.
- Extends local persistence needs with runtime recovery checkpoints or equivalent local recovery metadata while preserving local-only behavior.
- Depends on the planned user agent runtime and local usage persistence capabilities copied into this worktree.
- Adds tests under the runtime or recovery project test category using deterministic clocks, fake local usage stores, fake checkpoint stores, controlled boot/session identifiers, and diagnostic collectors.
- Preserves backend scope by avoiding UI, project/category assignment, startup registration, network sync, telemetry upload, external analytics, and `LocalService` infrastructure.
