## Why

The backend MVP has several completed local capabilities, but it still needs an end-to-end Windows hardening pass before a separate front-end UI round can safely depend on its data and diagnostics. This change verifies real per-user behavior across foreground changes, title changes, activity transitions, restart recovery, persistence integrity, and the local-only privacy boundary.

## What Changes

- Add a backend-only hardening verification plan for the completed per-user agent MVP capabilities.
- Validate foreground app switches and window-title changes through persisted observations, completed spans, and diagnostics.
- Validate idle, session lock/unlock, sleep/resume, restart, crash recovery, startup registration, and diagnostic inspection behavior on Windows.
- Validate local database integrity after normal operation, interruption, restart, recovery, and repeated diagnostics inspection.
- Validate that backend capture, recovery, startup, diagnostics, and hardening paths do not transmit captured usage data or diagnostics over a network.
- Produce a readiness decision for the later front-end UI round based on repeatable evidence and documented residual risks.
- Keep the change backend-only and verification-focused, with no tray UI, timeline UI, project/category assignment, remote sync, telemetry upload, external analytics, `LocalService`, or product front-end behavior.

## Capabilities

### New Capabilities
- `backend-mvp-hardening`: End-to-end Windows backend hardening verification for the per-user agent MVP, covering capture, activity transitions, restart recovery, startup registration, diagnostics, local database integrity, network absence, and front-end readiness evidence.

### Modified Capabilities

## Impact

- Adds OpenSpec requirements and implementation tasks for end-to-end backend verification rather than new product runtime behavior.
- Depends on completed local dependency artifacts for `add-agent-restart-recovery`, `add-agent-startup-registration`, and `add-backend-diagnostics`.
- May add or update test harnesses, scripts, fixtures, manual verification checklists, local diagnostic command coverage, or verification documentation needed to make the hardening pass repeatable.
- Requires TDD for any code added or changed to support verification, including tests for harness code, local-only guards, diagnostics assertions, or database integrity checks.
- Preserves the backend/front-end boundary by confirming backend readiness without adding user-facing UI, categorization, export/sync, telemetry, or service infrastructure.
