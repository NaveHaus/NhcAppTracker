## 1. Test Startup Registration Contracts

- [ ] 1.1 Add failing unit tests for host-neutral startup registration status results: absent, installed, mismatched, and failed read.
- [ ] 1.2 Add failing unit tests for deterministic launch-command rendering, including executable paths with spaces, runtime arguments, and exclusion of startup-management commands.
- [ ] 1.3 Add failing unit tests for install behavior against a fake current-user registration store: create missing registration, replace stale registration, and keep canonical registration idempotent.
- [ ] 1.4 Add failing unit tests for uninstall behavior against a fake current-user registration store: remove existing registration and treat already-absent registration as a successful absent state.
- [ ] 1.5 Add failing unit tests for failure handling so install, uninstall, and status failures report local diagnostics or failed command results without fabricating success.

## 2. Implement Host-Neutral Registration Behavior

- [ ] 2.1 Add host-neutral startup registration abstractions for reading, writing, deleting, and comparing current-user agent startup registration.
- [ ] 2.2 Implement canonical launch-command rendering from the resolved agent runtime executable path and normal runtime arguments.
- [ ] 2.3 Implement install orchestration that creates missing registration, replaces stale registration, and treats canonical registration as idempotent success.
- [ ] 2.4 Implement uninstall orchestration that removes existing registration and treats absent registration as idempotent success.
- [ ] 2.5 Implement status orchestration that reports absent, installed, mismatched, or failed states without mutating registration.
- [ ] 2.6 Refactor registration behavior after tests pass to keep Windows-specific concerns out of host-neutral contracts.

## 3. Add Windows Current-User Registration Edge

- [ ] 3.1 Add failing tests for the Windows edge using a fake or isolated registry adapter boundary to verify current-user Run-key value names, reads, writes, deletes, and error propagation.
- [ ] 3.2 Implement the Windows current-user sign-in registration adapter behind the host-neutral registration boundary.
- [ ] 3.3 Ensure the Windows implementation writes only current-user registration and does not add service-control-manager, `LocalService`, machine-wide, UI, network, or telemetry dependencies.
- [ ] 3.4 Refactor edge composition so Windows-specific APIs remain limited to the registration edge.

## 4. Wire Backend Startup Management Commands

- [ ] 4.1 Add failing command-level tests for `install-startup`, `uninstall-startup`, and `startup-status` or the equivalent backend entry points.
- [ ] 4.2 Wire startup-management commands to the host-neutral registration orchestration and local diagnostics or structured command results.
- [ ] 4.3 Ensure normal sign-in registration launches the agent runtime mode from `add-user-agent-runtime`, not startup-management commands.
- [ ] 4.4 Add or update backend documentation or command help for reversible install, uninstall, and status usage.

## 5. Validate Scope And Integration

- [ ] 5.1 Add tests or assertions that host-neutral registration projects do not reference Windows registry APIs, Windows service APIs, UI frameworks, network clients, telemetry clients, or persistence implementation details.
- [ ] 5.2 Add tests or inspections that this change introduces no Windows service, `LocalService`, service installer, service recovery configuration, or service-control-manager integration.
- [ ] 5.3 Run the required build and test commands for the affected projects from the assigned worktree.
- [ ] 5.4 Run OpenSpec verification for `add-agent-startup-registration` and resolve any artifact or implementation mismatches.
