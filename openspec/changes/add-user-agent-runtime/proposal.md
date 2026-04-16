## Why

The backend MVP needs a per-user headless process that connects the completed capture, activity, span-building, gap-modeling, and local persistence pieces into a running tracker. Without this runtime, the existing components remain independently testable but cannot produce durable local usage records from real user-session activity.

## What Changes

- Add a per-user headless agent runtime that samples foreground context in the interactive user session.
- Wire Windows foreground capture into host-neutral observation persistence.
- Wire activity signals into runtime state transitions for idle, active, locked, unlocked, suspending, resumed, logging out, and shutting down.
- Use the host-neutral span builder to complete attention spans from ordered observations.
- Use tracking-gap modeling to record explicit gaps for controlled stop, sleep, lock, logout, shutdown, and unclassified untracked time known to the runtime.
- Persist foreground observations, completed attention spans, and tracking gaps through the local usage sink.
- Handle cancellation and controlled shutdown without fabricating attended time.
- Produce structured backend diagnostics for runtime health, sampling, persistence, signal handling, span completion, gap creation, and shutdown outcomes.
- Keep the runtime backend-only with no UI, startup registration, network sync, telemetry upload, or `LocalService` infrastructure.

## Capabilities

### New Capabilities
- `user-agent-runtime`: Per-user headless runtime orchestration for foreground sampling, activity signals, span and gap production, local persistence, cancellation, controlled shutdown, and structured backend diagnostics.

### Modified Capabilities

## Impact

- Adds an executable or hosted process for the per-user backend agent.
- Adds application-level runtime orchestration that depends on existing domain, span builder, gap modeling, local persistence boundary, Windows foreground capture, and Windows activity signal capabilities.
- Adds tests under the runtime project test category using deterministic clocks, fake capture sources, fake activity signals, fake span/gap/persistence boundaries, and cancellation controls.
- Preserves the backend/front-end boundary by avoiding UI, project/category assignment, external transmission, startup registration, and `LocalService` behavior.
