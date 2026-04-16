## Why

The completed per-user agent runtime can run only when launched manually. Backend testing needs a reversible way to register that runtime for the interactive user at sign-in without introducing Windows service infrastructure before it is justified.

## What Changes

- Add per-user startup registration for the agent runtime so it starts when the user signs in.
- Provide install and uninstall paths that backend testers can run repeatedly to register or remove startup behavior.
- Ensure registration targets the current interactive user and does not require machine-wide service installation.
- Keep the approach local-only and reversible, with diagnostics or command output that make registration state observable during backend testing.
- Avoid adding a `LocalService`, Windows service host, elevated installer, UI, network sync, or telemetry behavior.
- Add tests first for registration, unregistration, idempotency, command behavior, and scope boundaries.

## Capabilities

### New Capabilities
- `agent-startup-registration`: Per-user sign-in startup registration and reversible install/uninstall behavior for the backend agent runtime.

### Modified Capabilities

## Impact

- Adds a backend startup registration boundary and Windows implementation for current-user sign-in launch.
- Adds install/uninstall command behavior or equivalent backend entry point for testable registration lifecycle.
- Depends on the per-user agent runtime produced by `add-user-agent-runtime`.
- Adds tests under the startup registration library or tool category using fakes for registration storage and command execution boundaries where possible.
- Preserves runtime architecture by avoiding `LocalService`, elevated service installation, UI, network transmission, and telemetry upload.
