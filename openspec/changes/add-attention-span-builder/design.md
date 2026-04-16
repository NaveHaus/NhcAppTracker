## Context

This change follows `add-attention-domain-contracts`. It assumes the host-neutral domain types for `ForegroundObservation`, `AttentionSpan`, app identity, and window context are available before implementation starts.

The span builder belongs in application logic because it coordinates ordered observations into completed work intervals without owning capture, persistence, UI, or runtime loop concerns. Windows foreground-window capture can later provide observations, but this builder must not depend on Win32 handles, polling timers, process APIs, storage, or service infrastructure.

## Goals / Non-Goals

**Goals:**
- Convert ordered `ForegroundObservation` values into completed `AttentionSpan` records deterministically.
- End the current span when the app identity or window context changes.
- Preserve the app identity and window context from the span's starting observation.
- Define explicit behavior for empty inputs, single observations, repeated same-context observations, context changes, and invalid timestamp ordering.
- Keep the implementation host-neutral and covered by focused unit tests created through a TDD red/green/refactor loop.

**Non-Goals:**
- Capture foreground windows, sample on an interval, detect idle/lock/sleep, or manage an active runtime session.
- Persist spans, create tracking gaps, categorize spans, merge across tracking sessions, or infer project assignments.
- Add Windows APIs, hosted services, UI, telemetry, sync, startup registration, or `LocalService` behavior.
- Change the domain contracts created by `add-attention-domain-contracts` unless implementation reveals a blocking contract defect.

## Decisions

### Decision: Build only completed spans

The builder will emit an `AttentionSpan` only when it has both a start observation and a later boundary observation. The final observation in a sequence remains open and does not produce a completed span by itself.

Rationale: completed spans require an end timestamp. The application runtime that owns sampling can later hold active state between calls; this change should not introduce open-ended spans or runtime session state.

Alternatives considered:
- Emit an open-ended span for the final observation: conflicts with the completed `AttentionSpan` contract and adds active-session semantics.
- Require a synthetic end timestamp parameter: useful later for flush behavior, but it mixes runtime lifecycle concerns into the base conversion algorithm.

### Decision: Context equality is based on domain value equality

The builder will treat app identity plus window context as the attention context. Consecutive observations with equal context continue the same pending span; a different app identity or different window context closes the previous span at the boundary observation's timestamp and starts a new pending span.

Rationale: the domain types should define equality semantics for host-neutral context. The span builder should not normalize titles, inspect process details, or add Windows-specific comparison rules.

Alternatives considered:
- Compare only app identity: misses window/document changes inside the same app.
- Compare raw source-specific identifiers: leaks platform adapter concerns into application logic.

### Decision: Require strictly increasing observation timestamps

The builder will require observations to be ordered by `ObservedAtUtc` with each timestamp greater than the previous timestamp. Equal or decreasing timestamps are invalid input and must be rejected before producing misleading zero-length or negative spans.

Rationale: span construction depends on valid temporal boundaries. A strict ordering rule keeps invalid capture sequences visible to the caller instead of silently producing bad output.

Alternatives considered:
- Sort observations internally: hides capture/runtime ordering defects and can reorder observations with ambiguous equal timestamps.
- Skip invalid observations: silently loses data and makes diagnostics harder.

### Decision: Keep the API batch-oriented and stateless

The initial implementation should expose a small stateless component or function that accepts an ordered observation sequence and returns completed spans. It should not retain the last observation across calls.

Rationale: this isolates pure conversion behavior for testing and keeps runtime sampling state out of scope. A future runtime loop can compose this logic with its own active-observation state.

Alternatives considered:
- Stateful builder with `AddObservation`: closer to runtime usage, but it creates lifecycle and flush semantics that are out of scope.
- Background service implementation: premature and violates the host-neutral boundary.

## Risks / Trade-offs

- Domain equality may not capture every future categorization distinction -> keep comparison limited to current domain value equality and adjust through later domain changes if requirements evolve.
- Strict timestamp rejection may surface noisy adapter data -> fail fast now so later capture code must normalize or repair input before calling the builder.
- Batch-only behavior will not directly model active tracking sessions -> compose it later with runtime state rather than embedding runtime concerns in this change.
- Final pending context is not emitted -> document this clearly in tests so callers understand that only completed spans are returned.

## Migration Plan

There is no runtime migration. Implement this after `add-attention-domain-contracts` has been applied and the application/test project scaffold exists.

Rollback consists of removing the span-builder application logic and matching tests; no persisted data or external system state is affected.

## Open Questions

- The final public type and method names should follow the project naming conventions visible after the scaffold and domain contracts are applied.
- If the domain contracts do not provide value equality for app identity or window context, implementation must either add equality there as part of the prerequisite change or define a local comparer without adding platform-specific rules.
