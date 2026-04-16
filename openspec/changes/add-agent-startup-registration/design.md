## Context

The `add-user-agent-runtime` dependency defines a per-user backend runtime that must run in the interactive user's session. That runtime intentionally excludes startup registration, installer behavior, Windows service hosting, and `LocalService` infrastructure.

This change adds only the startup-registration layer needed for backend testing: a current-user install path, a current-user uninstall path, and observable status for whether the agent runtime is registered to launch at sign-in. The implementation must preserve the runtime decision to avoid service infrastructure because foreground capture belongs in the interactive user session.

## Goals / Non-Goals

**Goals:**

- Register the per-user agent runtime to launch when the current user signs in.
- Provide reversible install, uninstall, and status behavior that backend testers can run repeatedly.
- Keep registration implementation testable behind a host-neutral boundary.
- Make registration idempotent: repeated install converges on the requested launch command, and repeated uninstall leaves registration absent.
- Emit local diagnostics or command results suitable for backend verification.
- Validate behavior with TDD using fake registration stores before adding the Windows implementation.

**Non-Goals:**

- No Windows service, `LocalService`, scheduled-task service host, elevated installer, machine-wide registration, or service recovery behavior.
- No UI, tray controls, timeline behavior, project/category assignment, network sync, telemetry upload, or remote diagnostics.
- No runtime orchestration changes beyond accepting the launch arguments needed to run normally or manage startup registration.
- No crash or restart recovery; that remains outside this change.

## Decisions

### Use Current-User Sign-In Registration

The Windows implementation should register the agent runtime for the current user only, using a per-user sign-in mechanism such as the `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` value. The registered command should point to the per-user agent executable and include only the arguments needed to start the runtime in normal backend-agent mode.

Rationale: current-user registration runs in the interactive session, avoids elevation, and is reversible for backend testing. It also keeps startup behavior aligned with the runtime requirement that foreground capture happens as the signed-in user.

Alternative considered: install a `LocalService` or machine-wide service. That does not satisfy foreground capture constraints and adds service lifecycle complexity before any later change has justified it.

Alternative considered: use the Startup folder. That is also per-user, but a registry-backed boundary is easier to inspect, replace idempotently, and test as a key/value registration store.

### Hide Windows Registration Behind a Port

The application layer should define a small startup-registration boundary for install, uninstall, and status. Tests should exercise command behavior against fake stores. The Windows edge should translate that boundary to the concrete current-user registration mechanism.

Rationale: this keeps command behavior deterministic and prevents tests from mutating the real user's startup configuration.

Alternative considered: put registry calls directly in command handling. That is simpler but makes idempotency, quoting, failure handling, and current-user scope harder to test without touching real machine state.

### Command Behavior Owns Reversibility And Observability

The backend agent executable or a closely related backend tool should expose explicit startup-management commands, for example `install-startup`, `uninstall-startup`, and `startup-status`. Each command should return a structured result or clear exit code and local diagnostic output.

Rationale: backend testers need a repeatable path to install and remove sign-in behavior without an external installer. Status output makes manual and automated verification possible.

Alternative considered: perform startup registration implicitly on first runtime launch. That hides side effects, makes uninstall less obvious, and complicates test setup and cleanup.

### Store A Fully Quoted Launch Command

The registered launch command should be built from the resolved agent executable path plus normal runtime arguments, with quoting rules that preserve paths containing spaces. Install should replace stale or differently quoted values with the canonical command.

Rationale: sign-in launch must survive common Windows paths, and idempotent replacement prevents stale backend test registrations from launching the wrong executable.

Alternative considered: store only an executable path. That limits future launch options and makes it harder to distinguish runtime mode from startup-management commands.

## Risks / Trade-offs

- Registry writes can fail because of policy or access restrictions -> return a failed command result, emit a local diagnostic with the failing operation, and leave existing registration untouched where possible.
- Quoting mistakes can break sign-in launch -> centralize command rendering and cover paths with spaces and arguments in unit tests.
- Backend testers can leave startup registration behind -> make uninstall idempotent and make status report the current registered command or absence.
- Per-user registration does not start the agent before sign-in -> accepted because foreground capture requires an interactive user session.
- The registered executable path can move after installation -> status should identify stale or mismatched commands, and install should replace stale values with the current resolved command.
- Current-user registration is Windows-specific -> keep the host-neutral boundary independent of Windows APIs and place registry access only in the Windows edge.
