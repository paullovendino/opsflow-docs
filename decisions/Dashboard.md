# Decision: Dashboard

## Status

Accepted — Milestone 6 complete (Phases 6.1–6.4)

## Context

OpsFlow completed Milestone 5 (Task Management). Product requirements and the roadmap call for a **Dashboard** that surfaces project/task summaries, statistics cards, chart-oriented breakdowns, and recent activity for operational visibility.

Activity Logs and Remarks are still future modules. Implementing a full audit trail solely to power “recent activities” would invent architecture ahead of those milestones. Companion docs previously stubbed only `GET /api/v1/dashboard` without contracts.

This ADR records approved Milestone 6 architecture so implementation can follow documentation without ad-hoc invention.

Companion specification: [docs/MILESTONE_6_DASHBOARD.md](../docs/MILESTONE_6_DASHBOARD.md)

## Decision

### Dashboard is a read model

- The Dashboard is a **computed aggregate**, not a persisted Eloquent entity
- Milestone 6 adds **no new database tables**, cache tables, or morph aliases
- Aggregates are derived from existing **`projects`** and **`tasks`** (exclude soft-deleted rows)

### Single read endpoint

- `GET /api/v1/dashboard` under `auth:sanctum`
- Standard API response envelope; payload shaped by `DashboardResource`
- Optional query param `recent_limit` (default **10**, max **25**, **clamp** preferred)
- No write routes; no nested project dashboard routes in Milestone 6

### Statistics (chart-ready JSON)

Return:

- Projects: `total`, `by_status` (all `ProjectStatus` keys, zero-filled)
- Tasks: `total`, `by_status` (all `TaskStatus` keys), `by_priority` (all `TaskPriority` keys), `overdue`, `assigned_to_me`

Backend does **not** render charts; the SPA (later) may consume these counts.

### Overdue definition

A non-soft-deleted task is overdue when `due_date` is set, `due_date` &lt; today, and `status` is neither `completed` nor `cancelled`.

### Recent work items (not Activity Logs)

- “Recent activities” in Milestone 6 means a merged feed of recent **projects** and **tasks** ordered by `updated_at` descending
- Item discriminator: `type` = `project` | `task`
- True Activity Log / audit feeds remain deferred until an Activity Logs milestone

### Visibility

| Role | Scope |
|------|--------|
| Administrator | Organization-wide projects/tasks |
| Project Manager | Organization-wide projects/tasks |
| Employee | Owned **or** member projects; tasks in those projects only |

`assigned_to_me` counts tasks assigned to the authenticated user within their visible set (org-wide assigned-to-self for Admin/PM).

### Authorization

- All authenticated roles may **view** the dashboard
- Enforce via `DashboardPolicy` (or equivalent Gate ability) + `$this->authorize()` in `DashboardController`
- Data scoping lives in `DashboardService` (same owned-or-member rules as Project/Task policies)

### Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| 6.1 | API foundation (controller, service, resource, request, route, guest `401`) | ✅ Implemented |
| 6.2 | Project & task statistics (incl. overdue / assigned_to_me + scoping) | ✅ Implemented |
| 6.3 | Recent work items feed | ✅ Implemented |
| 6.4 | Dashboard authorization Feature matrix | ✅ Implemented |

### Layering

- Thin `DashboardController`
- `ShowDashboardRequest` for query validation
- `DashboardService` for aggregates
- `DashboardPolicy` for coarse view ability
- `DashboardResource` for response shaping
- Prefer Enums / existing status enums — no magic strings for status keys

### Explicitly out of scope for Milestone 6

Vue Dashboard UI, Activity Logs, Remarks, Reports (Phase 7), caching/materialized views, date-range analytics, user-directory stats, nested project dashboards, live updates, write APIs.

## Consequences

- `API_SPECIFICATION.md` must document the approved payload (replacing the bare stub)
- `DOMAIN_MODEL.md` should describe Dashboard as a read model and clarify Recent Work Items vs Activity Logs
- `DATABASE_DESIGN.md` / Database ADR: confirm **no schema change** for Milestone 6
- Employee isolation tests are mandatory before Milestone 6 is marked complete
- Implemented via `Gate::define('viewDashboard', [DashboardPolicy::class, 'view'])` + `$this->authorize('viewDashboard')`
- Do not invent beyond this ADR / Milestone 6 specification during follow-on work
- Milestone 7 (Reports) has a separate ADR — do not start Reports coding until explicitly approved

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_6_DASHBOARD.md](../docs/MILESTONE_6_DASHBOARD.md)
- [docs/MILESTONE_5_TASK_MANAGEMENT.md](../docs/MILESTONE_5_TASK_MANAGEMENT.md)
- [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](../docs/MILESTONE_4_PROJECT_MANAGEMENT.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [decisions/Task-Management.md](Task-Management.md)
- [decisions/Project-Management.md](Project-Management.md)
- [decisions/Database.md](Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [REQUIREMENTS.md](../REQUIREMENTS.md)
- [UI_PAGES.md](../UI_PAGES.md)
