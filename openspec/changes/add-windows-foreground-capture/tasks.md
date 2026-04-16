## 1. Dependency and Project Boundary Check

- [ ] 1.1 Confirm `add-attention-domain-contracts` has been applied or that its domain API names are available in this worktree before implementing adapter code.
- [ ] 1.2 Inspect the solution and project layout to identify the Windows platform adapter project and matching test project locations.
- [ ] 1.3 Verify host-neutral projects do not reference Windows-specific projects before adding new Windows capture code.

## 2. Windows Snapshot to Domain Mapping

- [ ] 2.1 RED: using the `tdd` skill, add failing unit tests for mapping a valid Windows foreground snapshot into a `ForegroundObservation` with app identity, window context, observation timestamp, and user/session identity.
- [ ] 2.2 GREEN: implement the minimal Windows-edge snapshot type and mapper needed to create a valid `ForegroundObservation`.
- [ ] 2.3 RED: add failing unit tests showing empty window titles are accepted and non-empty titles with whitespace, punctuation, and mixed casing are preserved exactly.
- [ ] 2.4 GREEN: update the mapper to preserve captured titles exactly while accepting empty titles.
- [ ] 2.5 RED: add failing unit tests showing Win32 handles, process IDs, and Windows API types do not appear in host-neutral domain APIs except as opaque source identifier values.
- [ ] 2.6 GREEN: constrain Windows identifiers to adapter-local types or domain-supported opaque source identifiers.
- [ ] 2.7 REFACTOR: remove duplication in mapper tests and keep all Windows-specific test fixtures under the Windows adapter test boundary.

## 3. Foreground Window Capture

- [ ] 3.1 RED: using the `tdd` skill, add failing tests around an injectable/native capture boundary for foreground-window availability, title capture, owning process ID capture, and no-foreground-window behavior.
- [ ] 3.2 GREEN: implement minimal foreground-window capture using Windows-edge interop for current foreground window handle, window title, owning process ID, and capture timestamp.
- [ ] 3.3 RED: add failing tests for no-foreground-window and unresolved-owning-process cases returning no foreground observation without fabricating placeholder domain values.
- [ ] 3.4 GREEN: implement no-observation behavior for missing foreground window or unresolved app identity cases.
- [ ] 3.5 REFACTOR: keep native interop calls isolated behind narrow Windows adapter abstractions so deterministic tests can cover behavior without live Win32 calls.

## 4. Process Identity and Metadata Capture

- [ ] 4.1 RED: using the `tdd` skill, add failing tests for capturing process ID, process name, executable path, executable file name, and available display/product/file-description metadata.
- [ ] 4.2 GREEN: implement process identity and executable metadata capture using Windows-edge code and best-effort metadata lookup.
- [ ] 4.3 RED: add failing tests for inaccessible or missing optional metadata still producing a valid snapshot when process identity and window context are sufficient.
- [ ] 4.4 GREEN: implement graceful fallback when executable metadata is missing, inaccessible, or the process exits during capture.
- [ ] 4.5 REFACTOR: separate process lookup and file metadata lookup enough to test fallback paths without depending on a specific live process.

## 5. Scope Guardrails

- [ ] 5.1 Inspect the adapter implementation and verify it does not create `AttentionSpan` values, tracking gaps, persistence records, category assignments, telemetry events, external transmissions, tray UI behavior, startup registration, idle/lock detection, or `LocalService` infrastructure.
- [ ] 5.2 Inspect logging and diagnostics to verify captured window titles and process metadata are not logged or transmitted by this change.
- [ ] 5.3 Inspect project references and source files to verify Windows API imports exist only in Windows-specific platform adapter code and tests.
- [ ] 5.4 Run the documented formatting/build/test commands for the solution, including the Windows platform adapter tests.
- [ ] 5.5 Run `openspec status --change "add-windows-foreground-capture"` and verify the change is ready for apply verification.
