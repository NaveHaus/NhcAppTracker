# Back-End And Front-End Boundary

## Context

The user-agent MVP is a strictly local self-tracking backend. Its job is to collect and persist enough foreground app usage data for later review and categorization. The front-end UI is a second implementation round after the agent backend has been thoroughly tested.

The backend MVP runs in the interactive user's session as a per-user agent. It is not a corporate monitoring tool and it is not a `LocalService` replacement for foreground capture.

## Backend MVP Responsibilities

The backend implementation round owns:

- Per-user headless agent process lifecycle.
- Windows foreground app and window-title capture.
- Windows activity signals such as idle, lock/unlock, sleep/resume, logout, and shutdown.
- Host-neutral conversion from observations to attention spans.
- Explicit tracking gap creation for known untracked time.
- Strictly local persistence for observations, spans, and gaps.
- Startup registration for real-world backend testing.
- Backend diagnostics for test and verification.
- No network transmission of captured usage data.

The backend may include test-only or developer-facing inspection commands when needed to verify behavior without a UI. Those commands are not the product front end.

## Front-End Round Responsibilities

The front-end implementation round owns:

- Timeline review and editing.
- Project/category assignment.
- Bulk categorization and rule creation.
- User-facing pause/resume/status controls.
- Data purge/export user experience.
- User-facing privacy controls and explanations.
- Visual correction of spans and gaps.

The front end should consume backend data and backend control contracts instead of reimplementing capture, span-building, or persistence behavior.

## Boundary Contract

The backend should expose stable local data and control boundaries for the later UI:

- Captured foreground observations.
- Completed attention spans.
- Tracking gaps.
- Agent health/status data.
- Local-only storage access patterns.
- Future control points for pause, resume, stop, purge, and export.

The backend should not make product decisions about project/category assignment during the backend MVP. It should preserve enough context, especially window titles and app identity, for the user-facing categorization round.

## Rationale

Separating the backend and front-end rounds reduces risk. The highest-risk parts of the product are Windows foreground capture, session lifecycle behavior, idle/lock/sleep handling, restart recovery, and data integrity. These can be tested without a UI.

Building the UI too early would make the UI double as a debugging tool for capture correctness. That couples user experience work to platform behavior that is not proven yet. A backend-first sequence allows deterministic tests and diagnostics to harden the agent before timeline editing and categorization workflows are designed.

The boundary also protects the self-tracking product stance. The backend records local facts and gaps; the front end later gives the user control over interpretation, categorization, correction, and deletion.

## Deferred From Backend MVP

The backend MVP should not include:

- Tray or desktop UI.
- Project/category assignment UI.
- Timeline visualization.
- Bulk edit workflows.
- Rule-management UI.
- `LocalService` infrastructure.
- Network sync or analytics.
