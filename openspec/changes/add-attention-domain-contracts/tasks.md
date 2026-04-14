## 1. Prerequisite Check

- [ ] 1.1 Confirm `scaffold-csharp-project-layout` has been applied or that the scaffolded `src/Nhc.AppTracker.Domain` and `tests/Nhc.AppTracker.Domain.Tests` projects exist in the working tree.
- [ ] 1.2 Confirm this change will modify only host-neutral domain code and matching domain tests.

## 2. TDD Domain Contracts

- [ ] 2.1 RED: using the `tdd` skill, add failing unit tests for creating a valid `ForegroundObservation` from UTC observation time, valid app identity, valid window context, and opaque user/session identity.
- [ ] 2.2 GREEN: implement the minimal `ForegroundObservation`, `AppIdentity`, `WindowContext`, and `UserSessionIdentity` contracts needed for the valid observation test to pass.
- [ ] 2.3 RED: add failing unit tests for rejecting invalid foreground observations, including missing app identity and missing window context.
- [ ] 2.4 GREEN: implement minimal validation for invalid foreground observations.
- [ ] 2.5 REFACTOR: remove duplication in the observation tests and keep type names neutral rather than Windows-specific.

## 3. TDD Identity And Window Context Invariants

- [ ] 3.1 RED: add failing unit tests showing app identity accepts at least one non-empty identifying signal and rejects an identity with no non-empty signals.
- [ ] 3.2 GREEN: implement minimal app identity validation for captured process name, executable path, display name, product name, or opaque source identifiers.
- [ ] 3.3 RED: add failing unit tests showing window context preserves a non-empty title exactly and accepts an empty title as valid captured context.
- [ ] 3.4 GREEN: implement minimal window context behavior for exact title preservation.
- [ ] 3.5 REFACTOR: keep platform identifiers stored as opaque values and avoid introducing Win32 handle, process, or window API types.

## 4. TDD Attention Spans And Tracking Gaps

- [ ] 4.1 RED: add failing unit tests for creating a valid `AttentionSpan` with UTC start/end timestamps, end after start, app identity, and window context.
- [ ] 4.2 GREEN: implement the minimal `AttentionSpan` and shared time-range validation needed for the valid span test to pass.
- [ ] 4.3 RED: add failing unit tests showing `AttentionSpan` rejects an end timestamp equal to or before the start timestamp.
- [ ] 4.4 GREEN: implement the minimal invalid-span validation.
- [ ] 4.5 RED: add failing unit tests for creating valid `TrackingGap` values and rejecting invalid gap time ranges.
- [ ] 4.6 GREEN: implement the minimal `TrackingGap` contract and constrained gap reason values.
- [ ] 4.7 REFACTOR: share time-range validation between attention spans and tracking gaps without introducing persistence or runtime agent dependencies.

## 5. Validation

- [ ] 5.1 Run the documented domain test command from the repository root.
- [ ] 5.2 Run the documented solution build/test commands from the repository root.
- [ ] 5.3 Inspect the domain project references and verify they contain no references to Windows-specific projects, Windows-only APIs, persistence packages, UI, or `LocalService` infrastructure.
- [ ] 5.4 Verify this change contains no foreground-window capture, idle detection, span-building runtime loop, storage, categorization UI, startup registration, or `LocalService` behavior.
