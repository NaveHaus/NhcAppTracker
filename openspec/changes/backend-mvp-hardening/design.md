## Context

The backend MVP is a local per-user Windows agent that captures foreground app usage, converts observations into attention spans and tracking gaps, persists usage locally, registers for current-user startup, recovers from interruptions, and exposes backend diagnostics for inspection before a UI exists. The hardening change is the final backend-only verification gate before front-end work starts.

This change depends on the completed local artifacts for `add-agent-restart-recovery`, `add-agent-startup-registration`, and `add-backend-diagnostics`. It should not introduce product behavior unless verification exposes a defect that requires a focused fix. Any code added for harnesses, diagnostics assertions, integrity checks, or local-only guards must be developed with a TDD red/green/refactor workflow.

## Goals / Non-Goals

**Goals:**

- Define repeatable end-to-end Windows verification for foreground app switches, window title changes, idle, lock/unlock, sleep/resume, restart, crash recovery, startup registration, diagnostics inspection, local database integrity, and absence of network transmission.
- Capture evidence that persisted observations, spans, gaps, recovery records, and diagnostics match expected behavior.
- Make verification failures actionable by tying each hardening scenario to specific diagnostics, persistence records, or local command output.
- Produce an explicit readiness decision for the separate front-end UI round.
- Keep the backend local-only and preserve the backend/front-end boundary.

**Non-Goals:**

- No tray UI, desktop UI, timeline UI, web UI, categorization, project assignment, visual editing, export/sync workflow, telemetry upload, remote analytics, or network service.
- No `LocalService`, service-control-manager integration, elevated installer, anti-tamper behavior, or corporate monitoring behavior.
- No broad refactor of completed backend capabilities unless a hardening failure demonstrates a specific defect.
- No permanent production dependency on test-only network monitors, manual tools, or harness-only fixtures.

## Decisions

### Decision: Treat hardening as an evidence-producing verification capability

Hardening will produce a local evidence bundle or equivalent documented output containing command invocations, runtime diagnostics, persisted record summaries, integrity check results, network absence checks, environment notes, and pass/fail status per scenario.

Alternative considered: rely only on existing automated unit and integration tests. That is insufficient because the risk is end-to-end Windows behavior across real session lifecycle events and persisted local state.

### Decision: Use existing backend diagnostics as the primary observation surface

Verification will inspect recent observations, spans, gaps, health, current runtime state, and structured diagnostics through the backend diagnostics capability rather than reading implementation details first.

Alternative considered: inspect storage tables directly for every scenario. Direct database inspection is still required for integrity checks, but using diagnostics first validates the intended local inspection boundary that the future UI round will depend on.

### Decision: Separate automated harness checks from operator-driven Windows lifecycle steps

Deterministic checks such as command rendering, local-only dependency inspection, database integrity, diagnostics parsing, startup status, and crash-recovery idempotence should be automated where practical. Physical Windows lifecycle events such as lock/unlock and sleep/resume may use a documented operator checklist with required timestamps and evidence capture when they cannot be performed reliably in unattended tests.

Alternative considered: require all hardening scenarios to be fully automated. That would overfit the current backend round and may require brittle OS control code unrelated to product behavior.

### Decision: Validate absence of network transmission at both static and runtime levels

Hardening will verify that backend projects and command paths do not include network clients or telemetry/sync dependencies for captured usage data, and will also run a local runtime observation or equivalent OS-level check to confirm no outbound transmission occurs during the scenarios.

Alternative considered: only inspect source dependencies. Static inspection can miss runtime composition mistakes, while runtime observation alone can miss dormant code paths.

### Decision: Gate front-end readiness on explicit backend criteria

The front-end round is ready only when capture, activity transitions, recovery, startup registration, diagnostics, database integrity, and local-only checks either pass or have documented residual risks accepted as non-blocking.

Alternative considered: start the UI after the agent appears to run manually. That would let UI work depend on unproven persistence and lifecycle behavior.

## Risks / Trade-offs

- Windows lifecycle events are environment-sensitive -> Record OS version, power settings, user session state, timestamps, and commands with each run; classify environmental failures separately from backend defects.
- Manual checks can drift -> Keep operator steps scripted where possible and require concrete local evidence for every manual step.
- Network absence checks may produce false positives from unrelated processes -> Scope runtime observation to the backend process identity where possible and pair it with static dependency inspection.
- Direct database checks can couple to schema details -> Limit direct reads to integrity invariants and use backend diagnostics for behavior-level verification.
- Hardening may uncover defects outside this change's verification scope -> Record the failed scenario, add or update focused tests before code fixes, and avoid broad unrelated refactoring.

## Migration Plan

1. Add hardening verification tests, scripts, checklists, or documentation required by the tasks.
2. Run automated tests and the documented end-to-end Windows verification sequence in the assigned worktree.
3. Fix only defects required for the backend hardening criteria, using TDD for code changes.
4. Record evidence and the final readiness decision in the hardening output.
5. Roll back test-only harness or script changes by removing the affected files if the approach proves unsuitable; production runtime behavior should not need migration because this change is verification-focused.

## Open Questions

- Which local evidence format should become the long-term convention for backend readiness runs: Markdown report, structured JSON, command transcript, or a combination?
- Which Windows event steps can be automated reliably in the project test environment without adding product-irrelevant OS control dependencies?
