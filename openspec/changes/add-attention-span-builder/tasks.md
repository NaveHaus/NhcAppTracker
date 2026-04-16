## 1. Prerequisite Check

- [ ] 1.1 Confirm `add-attention-domain-contracts` has been applied or that the host-neutral attention domain contracts exist in the working tree.
- [ ] 1.2 Confirm the application project and matching test project exist before adding implementation code.
- [ ] 1.3 Confirm this change will modify only host-neutral application logic and matching tests.

## 2. TDD Completed Span Construction

- [ ] 2.1 RED: using the `tdd` skill, add failing unit tests showing empty observation input returns no completed spans.
- [ ] 2.2 RED: add failing unit tests showing a single foreground observation returns no completed spans.
- [ ] 2.3 GREEN: implement the minimal stateless span-builder API needed for empty and single-observation inputs to pass.
- [ ] 2.4 RED: add failing unit tests showing repeated observations with the same app identity and window context return no completed spans.
- [ ] 2.5 GREEN: implement minimal same-context pending-span behavior.
- [ ] 2.6 REFACTOR: keep test setup readable and reuse valid foreground-observation builders without hiding expected timestamps or contexts.

## 3. TDD Context Boundary Behavior

- [ ] 3.1 RED: add failing unit tests showing an app identity change completes the previous span at the later observation timestamp.
- [ ] 3.2 GREEN: implement app-identity boundary detection using host-neutral domain value equality.
- [ ] 3.3 RED: add failing unit tests showing a window context change completes the previous span at the later observation timestamp.
- [ ] 3.4 GREEN: implement window-context boundary detection using host-neutral domain value equality.
- [ ] 3.5 RED: add failing unit tests showing completed spans preserve the app identity and window context from the starting observation.
- [ ] 3.6 GREEN: implement starting-context preservation for completed spans.
- [ ] 3.7 REFACTOR: remove duplication between app-change and window-change cases while keeping boundary expectations explicit.

## 4. TDD Observation Ordering

- [ ] 4.1 RED: add failing unit tests showing equal consecutive observation timestamps are rejected.
- [ ] 4.2 GREEN: implement strict timestamp ordering validation for equal timestamps.
- [ ] 4.3 RED: add failing unit tests showing decreasing observation timestamps are rejected.
- [ ] 4.4 GREEN: implement strict timestamp ordering validation for decreasing timestamps.
- [ ] 4.5 RED: add failing unit tests showing multiple strictly increasing observations can produce all completed spans implied by app/window context changes.
- [ ] 4.6 GREEN: implement multi-boundary span construction across the full ordered input sequence.
- [ ] 4.7 REFACTOR: ensure invalid input produces no partial completed spans and the public API documents the strict-ordering expectation.

## 5. Boundary And Dependency Review

- [ ] 5.1 Inspect the implementation and project references to verify there are no Windows APIs, persistence packages, UI dependencies, hosted services, telemetry, sync, startup registration, or `LocalService` infrastructure.
- [ ] 5.2 Verify the builder is stateless and does not retain observations across calls.
- [ ] 5.3 Verify the final observation remains pending and is not emitted as a completed span without a later context boundary.

## 6. Validation

- [ ] 6.1 Run the focused application test command for the span-builder tests from the repository root.
- [ ] 6.2 Run the documented solution build command from the repository root.
- [ ] 6.3 Run the documented solution test command from the repository root.
- [ ] 6.4 Run `openspec status --change "add-attention-span-builder"` and confirm all implementation-required artifacts remain complete.
