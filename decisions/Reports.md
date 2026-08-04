# Decision: Reports

## Status

Accepted — Milestone 7 complete (Phases 7.1–7.4)

## Context

OpsFlow completed Milestone 6 (Dashboard). The roadmap calls for **Phase 7 — Reports** with **Project Reports** and **Employee Reports**.

The Dashboard already provides an organization snapshot without date filters. Reports must add **subject-focused**, **filterable** analytics without inventing Activity Logs, export pipelines, or new persistence tables ahead of later milestones.

Companion specification: [docs/MILESTONE_7_REPORTS.md](../docs/MILESTONE_7_REPORTS.md)

## Decision

### Reports are read models

- Reports are **computed aggregates**, not persisted Eloquent entities
- Milestone 7 adds **no new database tables**, cache tables, export tables, or morph aliases
- Aggregates derive from existing **`projects`**, **`tasks`**, and **`users`** (exclude soft-deleted rows)

### Endpoints

Under `auth:sanctum`:

| Method | Path |
|--------|------|
| GET | `/api/v1/reports/projects` |
| GET | `/api/v1/reports/projects/{project}` |
| GET | `/api/v1/reports/employees` |
| GET | `/api/v1/reports/employees/{user}` |

No write routes. No file-download / export routes in Milestone 7.

### Relationship to Dashboard

- Dashboard (`GET /api/v1/dashboard`) remains the snapshot API
- Reports do **not** replace or modify Dashboard contracts
- Reports add date-range filtering and Project / Employee subject focus

### Date range

- Optional `from_date` / `to_date` (`Y-m-d`, inclusive; `to_date` ≥ `from_date`)
- Filters which **tasks** count toward aggregates by **`created_at` date**
- Overdue definition **reuses Milestone 6** rules

### Project reports

- List: paginated summaries (search/status/sort + task aggregates + `members_count`)
- Detail: same payload shape for one project
- Visibility: Admin/PM all; Employee owned-or-member only

### Employee reports

- List: Admin/PM only; paginated user summaries with assignment aggregates
- Detail: Admin/PM any user; Employee **self only**
- Detail includes `by_project` breakdown; list omits `by_project`
- Only tasks with `assigned_to` = subject; exclude tasks whose project is soft-deleted

### Authorization

Enforced via `ReportPolicy` (+ Gate wiring as needed) and `$this->authorize()`:

| Ability | Admin | PM | Employee |
|---------|-------|----|----------|
| View project report list/detail | ✅ | ✅ | ✅ scoped |
| View employee report list | ✅ | ✅ | ❌ |
| View employee report detail | ✅ | ✅ | self only |

### Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| 7.1 | API foundation (controller, service, policy, routes, guest `401`) | ✅ Implemented |
| 7.2 | Project reports | ✅ Implemented |
| 7.3 | Employee reports | ✅ Implemented |
| 7.4 | Authorization Feature matrix | ✅ Implemented |

### Layering

- Thin `ReportController`
- Form Requests per action
- `ReportService` for aggregates
- `ReportPolicy` for coarse abilities
- `ProjectReportResource` / `EmployeeReportResource`
- Reuse existing enums — no magic strings for status/priority keys
- Zero-fill enum buckets (same as Dashboard)

### Explicitly out of scope for Milestone 7

PDF/CSV/Excel export, scheduled/email reports, Activity Logs, Remarks analytics, custom builders, caching/queues, Vue Reports UI, time tracking, replacing Dashboard.

## Consequences

- `API_SPECIFICATION.md` must document the four endpoints and payloads
- `DOMAIN_MODEL.md` should describe Reports as read models distinct from Dashboard and Activity Logs
- `DATABASE_DESIGN.md` / Database ADR: confirm **no schema change** for Milestone 7
- Employee isolation tests are mandatory before Milestone 7 is marked complete
- Implemented via `Gate::define` for `viewAnyProjectReports`, `viewProjectReport`, `viewAnyEmployeeReports`, `viewEmployeeReport`
- Do not invent beyond this ADR / Milestone 7 specification during follow-on work
- Milestone 8 is **Frontend Foundation** ([docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md)) — do not start implementation until explicitly approved; Vue Reports UI remains out of Milestone 8

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_7_REPORTS.md](../docs/MILESTONE_7_REPORTS.md)
- [docs/MILESTONE_6_DASHBOARD.md](../docs/MILESTONE_6_DASHBOARD.md)
- [decisions/Dashboard.md](Dashboard.md)
- [decisions/Task-Management.md](Task-Management.md)
- [decisions/Project-Management.md](Project-Management.md)
- [decisions/Database.md](Database.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [REQUIREMENTS.md](../REQUIREMENTS.md)
- [UI_PAGES.md](../UI_PAGES.md)
