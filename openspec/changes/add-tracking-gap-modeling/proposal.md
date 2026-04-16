## Why

The attention domain already needs explicit gaps so missing time is not confused with attended app time. This change defines host-neutral rules for creating those gaps before Windows signal capture is added, so platform adapters can report lifecycle signals without owning domain classification.

## What Changes

- Add host-neutral tracking-gap creation rules for known untracked time ranges.
- Represent stopped, crashed, system sleep, session locked, logged out, and unknown untracked time as constrained domain reasons.
- Keep gap creation separate from Windows signal capture, idle detection, persistence, runtime loops, UI, and service infrastructure.
- Define validation behavior for gap time ranges and reason mapping.
- Add focused tests through the required TDD red/green/refactor workflow.
- Treat `add-attention-domain-contracts` as a dependency because this change extends the tracking-gap concept introduced there.

## Capabilities

### New Capabilities
- `tracking-gap-modeling`: Defines host-neutral rules for explicit `TrackingGap` creation and constrained untracked-time reasons.

### Modified Capabilities

None.

## Impact

- Adds or refines domain model code and tests under the scaffolded domain project and matching test project.
- Establishes a stable domain boundary that future Windows capture, session, sleep, and process-health adapters can target.
- Does not add Windows API capture, OS signal subscriptions, storage, categorization UI, startup registration, `LocalService` behavior, telemetry, sync, or network behavior.
