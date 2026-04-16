## 1. Dependency and Scope Check

- [ ] 1.1 Confirm `add-attention-domain-contracts` has been applied or reconcile its pending domain contracts into this worktree before changing code.
- [ ] 1.2 Inspect the scaffolded domain and test projects to locate the existing `TrackingGap`, time-range, and attention-domain test structure.
- [ ] 1.3 Verify the planned implementation remains limited to host-neutral domain code and unit tests.

## 2. TDD Red Tests

- [ ] 2.1 Add failing unit tests for valid `TrackingGap` creation with a completed UTC range.
- [ ] 2.2 Add failing unit tests for equal and reversed gap ranges being rejected.
- [ ] 2.3 Add failing unit tests proving the supported reasons are stopped, crashed, system sleep, session locked, logged out, and unknown.
- [ ] 2.4 Add failing unit tests proving unsupported reasons or invalid reason values are rejected where the public API permits invalid input.
- [ ] 2.5 Add failing unit or project-structure tests proving the domain implementation does not reference Windows-specific capture, persistence, UI, telemetry, sync, network, or `LocalService` infrastructure.

## 3. Domain Implementation

- [ ] 3.1 Implement the minimal host-neutral gap reason model needed to represent stopped, crashed, system sleep, session locked, logged out, and unknown.
- [ ] 3.2 Implement or refine `TrackingGap` creation so it accepts only completed UTC ranges with end after start.
- [ ] 3.3 Implement or refine validation so invalid ranges and invalid reasons are rejected consistently with the existing domain contract style.
- [ ] 3.4 Keep Windows signal names, handles, event identifiers, session messages, foreground-window APIs, and adapter concerns out of the domain model.

## 4. Green and Refactor

- [ ] 4.1 Run the focused domain unit tests and make the new tests pass.
- [ ] 4.2 Refactor names, constructors, factories, or shared validation only where it simplifies the tracking-gap model and stays consistent with existing domain patterns.
- [ ] 4.3 Run the full repository test suite with `dotnet test` from the assigned worktree.
- [ ] 4.4 Run the repository build command if it is separate from `dotnet test`.

## 5. OpenSpec Verification

- [ ] 5.1 Confirm every scenario in `specs/tracking-gap-modeling/spec.md` is covered by tests or explicit project-structure verification.
- [ ] 5.2 Run `openspec status --change add-tracking-gap-modeling` and confirm the change remains apply-ready.
- [ ] 5.3 Run the available OpenSpec verification command for `add-tracking-gap-modeling` after implementation.
