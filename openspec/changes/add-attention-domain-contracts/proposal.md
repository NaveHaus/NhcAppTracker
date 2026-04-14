## Why

The user-agent MVP needs a host-neutral domain vocabulary before adding Windows foreground-window capture or local persistence. Defining the attention-tracking contracts as a separate increment keeps platform adapters from shaping the core model and gives later Windows-only work a stable boundary to target.

## What Changes

- Add host-neutral domain contracts for foreground observations, app identity, window context, attention spans, and tracking gaps.
- Define validation rules and invariants for time ranges, observed window context, and gap reasons.
- Keep captured context rich enough for future project categorization, including window titles, while preserving local-only and user-controlled product assumptions.
- Add tests for the new domain model using the required TDD red/green/refactor workflow.
- Do not implement Windows API capture, idle/lock detection, storage, agent runtime loops, categorization UI, installer/startup registration, or `LocalService` infrastructure in this change.
- Treat `scaffold-csharp-project-layout` as a prerequisite because this change assumes the domain project and test project layout exist.

## Capabilities

### New Capabilities
- `attention-domain-contracts`: Defines host-neutral attention-tracking domain contracts for foreground observations, attention spans, and tracking gaps.

### Modified Capabilities

None.

## Impact

- Adds domain types and tests under the scaffolded domain project and matching test project.
- Establishes naming and invariants for future Windows capture, persistence, and categorization changes.
- Does not add runtime tracking behavior, background services, OS-specific API calls, data storage, external network behavior, or UI.
