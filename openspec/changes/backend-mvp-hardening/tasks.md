## 1. Verification Setup

- [ ] 1.1 Review dependency artifacts for `add-agent-restart-recovery`, `add-agent-startup-registration`, and `add-backend-diagnostics` and record the backend commands, storage paths, diagnostics commands, and startup registration commands to use during hardening.
- [ ] 1.2 Define the hardening evidence format with required environment, command, timestamp, expected result, actual result, diagnostics, persistence summary, network check, and pass/fail fields.
- [ ] 1.3 If code is needed for evidence rendering or parsing, write failing unit tests first under the relevant `tests/<category>` path, implement the minimal code, and refactor after the tests pass.
- [ ] 1.4 Add or update a backend hardening runbook or script that starts from a clean local test storage location and records the assigned worktree path and backend-only scope.

## 2. Foreground Capture Verification

- [ ] 2.1 Run the backend agent locally and verify foreground app switches between at least two applications through backend diagnostics and local persistence evidence.
- [ ] 2.2 Verify window-title changes for a stable foreground application through observations, spans, or diagnostics according to the capture and span-building model.
- [ ] 2.3 If capture verification requires harness code, write failing tests first for command parsing, diagnostics assertions, or persistence summary extraction, then implement and refactor.
- [ ] 2.4 Record capture evidence showing app identity, window context, timestamps, completed spans where expected, and local-only inspection paths.

## 3. Activity Transition Verification

- [ ] 3.1 Verify idle handling produces idle gap or pending-gap evidence and does not count idle time as attended foreground time.
- [ ] 3.2 Verify Windows lock and unlock produce session-locked gap or constrained state evidence and that foreground tracking resumes after unlock.
- [ ] 3.3 Verify sleep and resume produce the expected system-sleep, reboot-compatible, or unknown unavailable-tracking gap evidence without attended-time fabrication.
- [ ] 3.4 Verify controlled agent shutdown flushes accepted records and leaves clean shutdown or stopped-tracking evidence for the next startup.
- [ ] 3.5 Record manual Windows lifecycle steps with timestamps, operator actions, diagnostics, persisted records, and environment notes.

## 4. Restart, Crash Recovery, and Startup Verification

- [ ] 4.1 Verify a normal agent restart runs recovery before sampling resumes and does not duplicate recovery-created records.
- [ ] 4.2 Verify unclean process termination followed by restart records constrained or unknown unavailable tracking gaps and does not create attended spans across downtime.
- [ ] 4.3 Verify repeated recovery over equivalent durable state is idempotent and emits duplicate-skip diagnostics.
- [ ] 4.4 Verify current-user startup registration installs, reports status, launches the normal runtime command, and does not register management commands.
- [ ] 4.5 If recovery or startup verification requires support code, write failing tests first for the support code, implement the minimal code, and refactor.

## 5. Local Persistence and Diagnostics Integrity

- [ ] 5.1 Run the project local database or persistence integrity check after capture, activity, restart, crash recovery, and diagnostics scenarios.
- [ ] 5.2 Verify persisted observations, spans, gaps, recovery records, and checkpoints have valid timestamp ordering and no fabricated open-ended usage records.
- [ ] 5.3 Verify repeated backend diagnostics inspection is read-only by comparing persistence summaries before and after diagnostics queries.
- [ ] 5.4 Verify valid records remain readable through local diagnostics or integrity inspection after restart.
- [ ] 5.5 If integrity comparison code is added, write failing unit or integration tests first, implement the comparison, and refactor.

## 6. Local-Only and Network Absence Verification

- [ ] 6.1 Perform static inspection of backend projects and command paths for HTTP clients, sockets, telemetry clients, sync clients, analytics clients, remote endpoints, or network transports that could transmit captured usage data or diagnostics.
- [ ] 6.2 Run process-scoped runtime network observation or an equivalent local network check during capture, activity, recovery, startup, diagnostics, and integrity scenarios.
- [ ] 6.3 Mark network absence failed or inconclusive when process attribution is ambiguous or a required static or runtime check cannot be completed.
- [ ] 6.4 Record local-only evidence proving observations, titles, app identity, session identity, runtime state, health, diagnostics, checkpoints, and recovery metadata stay in local sinks.

## 7. Readiness Decision and Verification

- [ ] 7.1 Produce the backend readiness evidence report listing each required hardening scenario, status, evidence path or command output, residual risk, and blocking/non-blocking classification.
- [ ] 7.2 Identify the local backend boundaries available to the future front-end UI round, including observations, spans, gaps, health, current runtime state, and recent diagnostic events.
- [ ] 7.3 Mark the backend ready only if required scenarios pass or documented non-blocking exceptions are explicitly accepted.
- [ ] 7.4 Run the repository build and test commands required by the project after any code or test changes, and record the results in the final hardening evidence.
- [ ] 7.5 Run `openspec status --change "backend-mvp-hardening"` and verify the OpenSpec change is apply-ready before implementation begins.
