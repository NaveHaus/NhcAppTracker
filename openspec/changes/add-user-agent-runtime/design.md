## Context

The backend MVP already has planned or copied local dependency artifacts for host-neutral attention contracts, span building, tracking-gap modeling, local usage persistence, Windows foreground capture, and Windows activity signals. This change is the first change that turns those components into a running per-user backend process.

The agent must run in the interactive user's session because foreground window and title capture cannot be performed correctly from a Windows `LocalService` process. The runtime must remain backend-only and local-only: it records observations, completed spans, and gaps, and it emits structured diagnostics for verification, but it does not provide UI, startup registration, project/category assignment, network sync, telemetry upload, or service infrastructure.

## Goals / Non-Goals

**Goals:**

- Add a per-user headless agent executable or hosted process.
- Compose foreground sampling, activity signal capture, span building, gap creation, and local persistence through explicit runtime boundaries.
- Keep runtime behavior deterministic in tests with fake clocks, fake capture sources, fake signal sources, fake persistence, and controllable cancellation.
- Persist every accepted foreground observation locally before it is used to complete spans.
- Complete attention spans only from valid ordered observations and persist completed spans through the local usage sink.
- Convert known runtime interruptions and lifecycle transitions into explicit tracking gaps without fabricating attended time.
- Emit structured backend diagnostics for lifecycle, sampling, persistence, signal handling, span completion, gap creation, cancellation, and shutdown.

**Non-Goals:**

- No tray UI, timeline UI, user-facing controls, or front-end behavior.
- No startup registration, installer behavior, Windows service, or `LocalService` infrastructure.
- No project/category assignment, categorization rules, timeline editing, export, purge, or sync.
- No network transmission, external analytics, telemetry upload, or remote diagnostics.
- No reimplementation of span-building, gap validation, foreground capture, activity-signal translation, or local persistence rules already owned by dependency changes.
- No restart/crash recovery beyond controlled runtime shutdown and currently observed lifecycle transitions; deterministic restart recovery belongs to `add-agent-restart-recovery`.

## Decisions

### Runtime Uses Explicit Ports Around Existing Capabilities

The runtime should depend on application-level ports for foreground sampling, activity events, span building, gap creation, local usage writes, clocks, delay/scheduling, and diagnostics. The production host wires those ports to the Windows foreground capture adapter, Windows activity signal adapter, host-neutral builders, and local persistence implementation.

Rationale: this keeps the runtime testable without real Windows foreground changes, lock/unlock, suspend/resume, logout, shutdown, or real disk writes in every test. It also preserves dependency direction: Windows-specific projects depend inward on host-neutral contracts, while the runtime composes concrete edge implementations at the executable boundary.

Alternative considered: make the runtime call concrete Windows and storage implementations directly. That is simpler initially but makes cancellation, timing, signal, and failure paths difficult to test and risks coupling host-neutral orchestration to platform details.

### One Ordered Runtime Pipeline Owns Observation State

The agent runtime should maintain the last accepted foreground observation as runtime state. Each successful sample is persisted as an observation, then passed with the previous accepted observation to the span builder. Completed spans are persisted, and the new observation becomes the pending context. Failed or empty samples do not fabricate observations, spans, or gaps by themselves.

Rationale: span completion depends on ordered observations and a later boundary. Persisting observations before span completion preserves raw capture facts even if later span persistence fails or shutdown occurs.

Alternative considered: batch observations and build spans periodically. Batching reduces write frequency, but it increases loss risk during shutdown and complicates gap boundaries.

### Activity Events Gate Sampling And Create Known Gaps

The runtime should react to host-neutral activity events by updating tracking state. Idle or locked states stop foreground sampling for attended-time purposes and start a pending gap range. Active or unlocked/resumed transitions close the pending gap with the appropriate host-neutral reason, persist it, and then resume sampling. Suspending, logging out, and shutting down close runtime state as controlled lifecycle exits where possible.

Rationale: explicit gaps are more accurate than pretending the foreground app remained attended during idle, locked, sleep, logout, or shutdown periods. The runtime should use the existing gap model for validation and reason constraints.

Alternative considered: continue sampling foreground windows through idle and lock. That overstates attended time and hides important untracked intervals from the future timeline UI.

### Controlled Shutdown Flushes Known State

On cancellation or controlled shutdown, the runtime should stop scheduling new samples, drain already accepted activity events, record a stopped or lifecycle-specific gap when there is a completed known untracked range, flush completed persistence writes, emit shutdown diagnostics, and then exit. The runtime must not complete a final attention span without a later valid observation boundary.

Rationale: a controlled shutdown can record why tracking stopped, but it cannot infer how long the current foreground context was attended after the last observation. Restart and crash recovery will later reconcile open-ended state.

Alternative considered: close the pending span at shutdown time. That fabricates attended time between the last observation and shutdown even if the user was idle, locked, or the sample was stale.

### Diagnostics Are Structured And Local

Diagnostics should be emitted as structured local backend events with stable event names, timestamps, severity, correlation fields where useful, and exception details when applicable. The runtime should emit diagnostics through an injectable sink so tests can assert them and production can route them to console, file, or local logging without network transmission.

Rationale: this backend phase needs verification without a UI, and structured diagnostics make sampling, signal, and persistence behavior auditable during manual and automated testing.

Alternative considered: rely on ad hoc console strings. That is harder to test and less useful for later local inspection tools.

## Risks / Trade-offs

- Sampling and activity events can race -> serialize runtime state transitions through a single runtime loop or command queue so observation, span, and gap state changes are ordered.
- Persistence failure can leave observations written without spans or gaps -> emit structured diagnostics, stop or degrade deterministically, and avoid marking fabricated success; later recovery can reconcile durable records.
- Activity transitions can arrive without a clean matching resume/unlock event -> keep pending gap state explicit and only persist completed gaps with valid end timestamps.
- Windows signal delivery during process exit can be incomplete -> handle cancellation and process lifetime notifications as controlled shutdown signals, but leave crash/restart reconciliation to a later recovery change.
- Short sampling intervals improve fidelity but increase writes -> make the interval configurable with a conservative default and deterministic scheduler abstraction for tests.
- The runtime executable may need Windows-only references -> keep those references in the Windows host/executable boundary and keep host-neutral application contracts free of Windows APIs.
