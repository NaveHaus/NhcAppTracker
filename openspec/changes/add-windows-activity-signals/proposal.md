## Why

The user-agent MVP needs Windows activity-state signals before the runtime can distinguish attended time from known untracked time around idle periods, session locks, sleep/resume, logout, and shutdown. Capturing those signals as host-neutral events keeps Windows APIs isolated from the domain/application boundary established by `add-attention-domain-contracts`.

## What Changes

- Add Windows-only signal capture for user idle state, session lock/unlock, suspend/resume, logout, and shutdown notifications.
- Translate captured platform signals into host-neutral agent/application events that later runtime and gap logic can consume.
- Define timestamp, ordering, and duplicate-handling expectations for emitted activity signals.
- Add tests through the required TDD red/green/refactor workflow, using abstractions or fakes so signal translation can be tested without forcing real OS session transitions.
- Keep this change out of direct UI behavior, persistence, foreground-window capture, attention span building, runtime orchestration, startup registration, and `LocalService` infrastructure.
- Treat `add-attention-domain-contracts` as a prerequisite because emitted events must align with the existing host-neutral attention and gap vocabulary.

## Capabilities

### New Capabilities
- `windows-activity-signals`: Captures Windows idle and session/power lifecycle signals and translates them into host-neutral activity events.

### Modified Capabilities

None.

## Impact

- Adds Windows-specific adapter code and matching tests in the scaffolded Windows platform/agent area once the project layout exists.
- Introduces a host-neutral activity-signal contract or application-facing event type that can be consumed by later runtime and tracking-gap changes.
- May use Windows APIs for last-input, session change, power mode, console control, or equivalent OS notifications behind adapter boundaries.
- Does not add storage, network behavior, analytics, UI controls, foreground-window capture, span-building loops, startup registration, or `LocalService` behavior.
