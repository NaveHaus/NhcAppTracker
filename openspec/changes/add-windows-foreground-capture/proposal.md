## Why

The Windows user-agent MVP needs a platform adapter that can observe the foreground window in the interactive user session and translate that native context into the host-neutral attention-domain contracts. Keeping this as its own change lets Windows API capture stay at the platform edge without pushing span-building, storage, or runtime orchestration into the adapter.

## What Changes

- Add a Windows foreground capture adapter that can read the current foreground window title and owning process identity.
- Capture process identity details including process ID, process name, executable path/name, and available app metadata such as display/product information.
- Translate captured Windows data into `ForegroundObservation` values from the attention-domain contracts.
- Preserve Windows-specific identifiers only as adapter-local or opaque source values when mapping into domain contracts.
- Do not implement attention-span construction, persistence, categorization, tray UI, idle/lock handling, startup registration, external transmission, or `LocalService` behavior.

## Capabilities

### New Capabilities
- `windows-foreground-capture`: Defines Windows foreground-window observation behavior and translation into host-neutral foreground observations.

### Modified Capabilities

## Impact

- Adds implementation and tests under the Windows platform adapter boundary.
- Depends on the project layout from `scaffold-csharp-project-layout` and the domain contracts from `add-attention-domain-contracts`.
- Introduces Windows API interop at the Windows edge only; host-neutral projects must not reference Windows-specific code or APIs.
- Establishes test coverage for Windows data translation while avoiding persistence and span-building responsibilities.
