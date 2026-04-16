## 1. Prerequisite And Boundary Checks

- [ ] 1.1 Confirm `add-attention-domain-contracts` has been applied or that its host-neutral attention/gap contracts are available in the working tree.
- [ ] 1.2 Confirm the scaffolded solution includes an appropriate Windows-specific adapter project and matching test project, or add only the minimal project wiring consistent with the established scaffold.
- [ ] 1.3 Confirm this change will not modify UI, persistence, foreground-window capture, span-building runtime loops, startup registration, network, telemetry, or `LocalService` infrastructure.

## 2. TDD Host-Neutral Activity Event Contract

- [ ] 2.1 RED: using the `tdd` skill, add failing unit tests for creating host-neutral activity events with constrained event kinds and UTC observed timestamps.
- [ ] 2.2 GREEN: implement the minimal activity event type and event-kind values for idle, active, locked, unlocked, suspending, resumed, logging out, shutting down, and unknown.
- [ ] 2.3 RED: add failing unit tests showing activity events reject or normalize non-UTC timestamps according to the chosen implementation rule.
- [ ] 2.4 GREEN: implement the minimal timestamp validation or normalization behavior.
- [ ] 2.5 REFACTOR: keep event names host-neutral and free of Win32 handles, Windows message IDs, UI framework event args, persistence types, or `LocalService` concepts.

## 3. TDD Windows Signal Translation

- [ ] 3.1 RED: add failing unit tests that translate fake raw Windows idle and active signals into host-neutral activity events.
- [ ] 3.2 GREEN: implement the minimal idle/active signal translator using deterministic timestamps from an injectable clock or timestamp source.
- [ ] 3.3 RED: add failing unit tests that translate fake raw Windows lock, unlock, suspend, resume, logout, and shutdown signals.
- [ ] 3.4 GREEN: implement the minimal session, power, logout, and shutdown signal translation.
- [ ] 3.5 RED: add failing unit tests showing repeated idle or repeated active samples do not emit duplicate transition events.
- [ ] 3.6 GREEN: implement the minimal state tracking needed to suppress duplicate idle and active transitions while preserving distinct lifecycle transitions in source order.
- [ ] 3.7 REFACTOR: separate raw Windows signal representations from host-neutral event output and keep translation tests independent of real OS transitions.

## 4. TDD Windows Adapter Sources

- [ ] 4.1 RED: add failing unit tests around the idle source boundary using fake last-input samples and a configurable idle threshold.
- [ ] 4.2 GREEN: implement the minimal Windows idle source abstraction and threshold calculation needed by the translator.
- [ ] 4.3 RED: add failing unit tests around session, power, logout, and shutdown source boundaries using fake notification inputs.
- [ ] 4.4 GREEN: implement the minimal Windows source abstractions needed to surface lock, unlock, suspend, resume, logout, and shutdown raw signals.
- [ ] 4.5 REFACTOR: quarantine Windows API calls and any platform-specific package references inside the Windows-specific adapter project.

## 5. Validation

- [ ] 5.1 Run the Windows activity signal unit tests from the repository root.
- [ ] 5.2 Run the documented solution build/test commands from the repository root.
- [ ] 5.3 Inspect host-neutral project references and verify they contain no Windows-only API references, Windows adapter references, UI framework references, persistence packages, telemetry, or `LocalService` infrastructure.
- [ ] 5.4 Verify the change contains no direct UI behavior, storage writes, foreground-window capture, attention span building, runtime orchestration, startup registration, network transmission, telemetry, or `LocalService` behavior.
