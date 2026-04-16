## 1. Runtime Test Harness And Contracts

- [ ] 1.1 Add failing unit tests under `tests/<runtime-category>` for starting the runtime with fake foreground, activity, span, gap, persistence, clock, scheduler, cancellation, and diagnostics dependencies.
- [ ] 1.2 Add the minimal runtime project structure and host-neutral runtime ports needed to make the startup/composition tests compile.
- [ ] 1.3 Implement the runtime start/ready lifecycle behavior until the startup tests pass.
- [ ] 1.4 Refactor runtime contracts for clear dependency direction and verify host-neutral projects do not reference Windows-specific adapters or UI/service infrastructure.

## 2. Foreground Sampling And Observation Persistence

- [ ] 2.1 Add failing unit tests for scheduled sampling of valid observations, empty samples, invalid samples, capture failures, and deterministic scheduler advancement.
- [ ] 2.2 Implement scheduled foreground sampling through the runtime foreground source port.
- [ ] 2.3 Persist each valid `ForegroundObservation` through the local usage sink before span processing.
- [ ] 2.4 Emit structured diagnostics for accepted, empty, invalid, and failed foreground samples.
- [ ] 2.5 Refactor sampling state to keep tests deterministic and run the relevant runtime tests.

## 3. Span Completion Pipeline

- [ ] 3.1 Add failing unit tests for first observation pending state, same-context observations, context-boundary span completion, non-increasing timestamps, and no span fabrication at shutdown.
- [ ] 3.2 Implement pending observation tracking and ordered-observation validation in the runtime.
- [ ] 3.3 Call the host-neutral span builder when a later valid observation creates a context boundary.
- [ ] 3.4 Persist completed `AttentionSpan` records through the local usage sink and emit persistence diagnostics.
- [ ] 3.5 Refactor span pipeline code to keep span-building rules delegated to the existing span builder.

## 4. Activity Signals And Tracking Gaps

- [ ] 4.1 Add failing unit tests for idle/active, lock/unlock, suspend/resume, logout, shutdown, and duplicate activity event handling.
- [ ] 4.2 Implement runtime tracking-state transitions from host-neutral activity events.
- [ ] 4.3 Create and persist completed `TrackingGap` records for known untracked ranges using existing gap modeling rules and constrained reasons.
- [ ] 4.4 Prevent duplicate gaps for repeated effective activity states and diagnose ignored duplicate transitions.
- [ ] 4.5 Refactor activity handling so Windows signal translation remains outside the runtime state machine.

## 5. Cancellation And Controlled Shutdown

- [ ] 5.1 Add failing unit tests for cancellation stopping new samples, flushing accepted writes, controlled stopped gaps for completed ranges, avoiding open-ended gaps, and shutdown persistence failures.
- [ ] 5.2 Implement cancellation handling that stops scheduling new foreground samples and begins controlled shutdown.
- [ ] 5.3 Await or complete accepted observation, span, and gap writes before reporting shutdown complete.
- [ ] 5.4 Record stopped or lifecycle-specific gaps only when the runtime has valid completed gap ranges.
- [ ] 5.5 Emit structured shutdown diagnostics for complete, canceled, pending-state, and persistence-failure outcomes.

## 6. Structured Local Diagnostics

- [ ] 6.1 Add failing unit tests that assert stable diagnostic event names, UTC timestamps, severity, runtime state fields, record kind fields, and exception/failure details.
- [ ] 6.2 Implement the runtime diagnostic event model and diagnostic sink port.
- [ ] 6.3 Route lifecycle, sampling, activity, span, gap, persistence, cancellation, and shutdown outcomes through the diagnostic sink.
- [ ] 6.4 Add tests or static inspections proving diagnostics do not use HTTP clients, sockets, telemetry uploaders, analytics clients, sync clients, or remote endpoints.

## 7. Production Host Composition

- [ ] 7.1 Add failing composition tests or project-structure inspections for the per-user Windows agent executable or hosted process.
- [ ] 7.2 Add the production host that wires Windows foreground capture, Windows activity signals, host-neutral span builder, tracking gap modeling, local persistence, scheduler, cancellation, and local diagnostics.
- [ ] 7.3 Ensure Windows-specific references remain limited to the executable or Windows adapter edge.
- [ ] 7.4 Verify the host adds no tray UI, timeline UI, project/category assignment, startup registration, installer behavior, network sync, telemetry upload, analytics upload, or `LocalService` implementation.

## 8. Verification

- [ ] 8.1 Run the focused runtime unit tests and fix failures through the red/green/refactor loop.
- [ ] 8.2 Run the full repository test command from the assigned worktree.
- [ ] 8.3 Run the repository build command from the assigned worktree.
- [ ] 8.4 Run OpenSpec verification for `add-user-agent-runtime` and resolve any artifact or implementation mismatches.
- [ ] 8.5 Review changed files to confirm no modifications occurred outside `D:\dev\nhc\oss\NhcAppTracker\working.git\.subagents\worktrees\add-user-agent-runtime`.
