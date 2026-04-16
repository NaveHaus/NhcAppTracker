## Context

This change follows `add-attention-domain-contracts`, which introduces `TrackingGap` as the domain representation for known untracked time. The next increment needs explicit creation rules for those gaps before Windows foreground capture, session-state capture, or service health monitoring are implemented.

The product remains local-only. This change belongs in host-neutral domain code and tests, with no dependency on Windows APIs, runtime loops, storage, UI, telemetry, sync, or `LocalService` infrastructure.

## Goals / Non-Goals

**Goals:**
- Define the allowed untracked-time reasons for explicit `TrackingGap` creation: stopped, crashed, system sleep, session locked, logged out, and unknown.
- Validate completed gap time ranges with UTC timestamps and end strictly after start.
- Keep gap creation host-neutral so future Windows adapters can translate OS or process signals into neutral domain inputs.
- Distinguish deliberate lifecycle gaps from unknown gaps without encoding Windows-specific signal names in the domain model.
- Add focused unit tests through a TDD red/green/refactor loop.

**Non-Goals:**
- Implement Windows signal capture for lock, logout, sleep, process start, process stop, crash, idle, foreground-window changes, or service health.
- Implement persistence, categorization, UI, startup registration, background service behavior, analytics, sync, telemetry, or network behavior.
- Infer attention spans or merge adjacent gaps beyond the explicit creation rules in this change.

## Decisions

### Decision: Keep `TrackingGap` creation in the domain model

The domain should expose a small creation surface for validated gaps rather than requiring adapters to construct arbitrary objects.

Rationale: future capture hosts will see different lifecycle signals. Keeping creation rules in the domain ensures stopped, crashed, sleep, locked, logged-out, and unknown gaps mean the same thing regardless of where the signal came from.

Alternatives considered:
- Let each adapter construct gaps directly: simpler initially, but it duplicates validation and risks Windows-specific classifications leaking into the domain.
- Infer gaps only from timeline holes: avoids explicit signal mapping, but loses the reason why time was untracked.

### Decision: Model gap reasons as constrained host-neutral reasons

Use a constrained domain reason set for stopped, crashed, system sleep, session locked, logged out, and unknown. Avoid names tied to Windows APIs, event IDs, session messages, or process handles.

Rationale: the domain needs durable semantic reasons, while platform-specific adapters can map their own raw signals into these reasons later.

Alternatives considered:
- Store raw OS signal names as gap reasons: useful for diagnostics, but host-specific and unstable as domain data.
- Use a free-form string reason: flexible, but weakens validation and makes UI or reporting behavior harder to rely on.

### Decision: Treat unknown as a valid explicit gap reason

Unknown untracked time should be representable when the system can identify a time range but cannot classify the cause.

Rationale: the timeline should be honest about missing data without inventing a cause. Unknown gaps also give later recovery, diagnostics, or user-editing workflows a clear placeholder.

Alternatives considered:
- Reject unclassified gaps: stricter, but forces callers either to drop known missing time or misclassify it.
- Collapse unknown into stopped: loses the difference between deliberate stopping and unexplained missing data.

### Decision: Keep signal capture outside this change

This change only defines domain rules. Future Windows work can capture lock, logout, sleep, process crash, and stop events, then translate those inputs into the host-neutral creation surface.

Rationale: separating capture from modeling keeps the domain testable without OS dependencies and keeps the next platform adapter change smaller.

Alternatives considered:
- Implement Windows signal capture together with gap modeling: delivers an end-to-end slice, but mixes platform integration risk with domain rules.

## Risks / Trade-offs

- Reason names may need refinement when Windows capture is implemented: keep names semantic and host-neutral, and update only through a later spec change if needed.
- Domain creation rules may be too narrow for future causes: preserve `unknown` as the fallback for valid untracked time that has no known reason yet.
- Over-modeling could delay implementation: add only the minimal factory or constructor behavior needed to make invalid gaps unrepresentable or rejected.
- Active tracking state may need open-ended ranges later: this change models completed gaps only; runtime state can be added separately.

## Migration Plan

There is no runtime migration. Implement this after `add-attention-domain-contracts` is available in the working tree. If the dependent contracts have not been applied, apply or reconcile that change first so this change can extend the same domain project and tests.

## Open Questions

- The exact public API shape can be chosen during TDD, but it must expose host-neutral creation of completed gaps and enforce the specified invariants.
- Whether to use enum names such as `SystemSleep` and `SessionLocked` or shorter names such as `Sleep` and `Locked` should be decided during implementation based on clarity in tests and domain code.
