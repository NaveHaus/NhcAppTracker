# LocalService Migration Path

## Context

The MVP is a strictly local self-tracking and project-management tool. It tracks user attention on foreground apps, including window titles, so a future UI can help the user categorize time by project.

This product is not intended for corporate control or anti-tamper monitoring. The design should prioritize transparency, user control, recoverability, and accurate gap reporting.

## Core Constraint

A Windows `LocalService` service cannot replace the per-user foreground tracking agent.

Foreground app and window-title capture must happen in the interactive user's session. A Windows service runs in session 0 and does not have direct access to the user's foreground desktop context.

```text
Required long-term shape:

Per-User Agent
  - captures foreground window/app/title
  - interprets idle, lock, and session signals
  - exposes user-facing pause/quit/status controls

Optional LocalService Service
  - provides local infrastructure
  - owns shared storage or maintenance tasks
  - monitors health and records gaps
```

## MVP Shape

Start with a single per-user background agent.

```text
Agent.exe
  - foreground window capture
  - idle/lock handling
  - attention span building
  - local storage writes
```

The agent should record untracked gaps instead of hiding them. Examples include paused tracking, stopped agent, crash/restart, sleep/resume, lock, and logout.

## Preserve This Boundary

The important boundary is not "agent versus service." It is:

```text
Capture and build observations
        |
        v
Persist and manage observations
```

Keep capture/span-building logic separate from persistence through a small application-level boundary, such as a usage sink:

```text
UsageSink.RecordObservation(...)
UsageSink.RecordSpan(...)
UsageSink.RecordGap(...)
```

This does not need a large abstraction framework. The goal is only to avoid coupling OS capture logic directly to a specific host process or storage implementation.

## Migration Path

1. MVP agent writes directly to local storage.
2. Keep the domain model host-neutral: `ForegroundObservation`, `AttentionSpan`, `TrackingGap`, and related value types should not depend on the agent process.
3. Put storage behind a small usage-sink boundary.
4. Later extract storage and domain code into a shared library if needed.
5. Add a named-pipe usage-sink implementation in the agent.
6. Add a `LocalService` host that receives records over the named pipe and writes to the same store.
7. Keep direct storage as a fallback until the service path is stable.

## Responsibilities That Stay In The Agent

| Responsibility | Reason |
| --- | --- |
| Foreground window lookup | Requires the interactive user session. |
| Window title capture | Requires the interactive user session. |
| App/process identity capture | Closest to the foreground window source. |
| Idle/lock interpretation | Closest to user session signals. |
| Tray/status/pause UX | User-facing control belongs in the user session. |

## Responsibilities That Can Move To LocalService Later

| Responsibility | Reason |
| --- | --- |
| Database ownership | Centralizes writes and access control. |
| Migrations | Host-independent maintenance. |
| Compaction and cleanup | Does not require desktop access. |
| Heartbeat monitoring | Supports crash detection and gap accounting. |
| Multi-user coordination | A service can coordinate across interactive sessions. |
| Local IPC endpoint | Provides a stable communication point for agent and future UI. |

## Design Rule

Choosing a per-user agent for the MVP does not lock out a future `LocalService` service. The later service should be a deployment split around persistence, maintenance, and coordination, not a rewrite of foreground tracking.
