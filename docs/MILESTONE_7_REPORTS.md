# Milestone 7 — Reports

**Status:** ✅ Milestone 7 complete (Phases 7.1–7.4)  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-04

> Implementation specification for Milestone 7.  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)  
> ADR: [decisions/Reports.md](../decisions/Reports.md)  
> Prerequisite: Milestone 6 complete (Dashboard read model, Project/Task visibility rules)

---

## 1. Goal

Introduce OpsFlow’s **Reports** module on the API as **read-only, filterable analytical summaries** over existing Projects, Tasks, and Users:

- **Project Reports** — per-project and list-level task aggregates
- **Employee Reports** — per-assignee productivity aggregates (and Admin/PM directory list)
- Optional **date-range** filtering (deferred from Dashboard Milestone 6)
- Enforce visibility consistent with Project / Task / User authorization
- Remain schema-light: **no new persistence tables** in Milestone 7

Reports differ from the Dashboard:

| Concern | Dashboard (M6) | Reports (M7) |
|---------|----------------|--------------|
| Purpose | Snapshot cards + recent feed | Filterable Project / Employee analysis |
| Endpoints | Single `GET /dashboard` | Project + Employee report routes |
| Date range | Not supported | Optional `from_date` / `to_date` |
| Subject focus | Org (scoped) totals | Project-centric and User-centric |

The backend returns JSON suitable for future report UI. It does **not** render charts, generate files, or ship Vue screens.

---

## 2. Domain summary

```text
Authenticated User
    └── Reports (read models — not persisted entities)
            ├── Project Report(s)  → Project + Task aggregates
            └── Employee Report(s) → User (assignee) + Task aggregates
```

- Reports are **computed views**, not domain tables
- Aggregates use existing `projects`, `tasks`, and `users` (soft-deleted rows excluded)
- No Activity Logs, Remarks, or export artifacts in Milestone 7
- Dashboard remains the org snapshot; Reports do not replace `GET /api/v1/dashboard`

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **7.1 Reports API Foundation** | `ReportController` / `ReportService` / shared date-range helpers / `ReportPolicy` Gate wiring / route group; guest `401` | ✅ Implemented |
| **7.2 Project Reports** | `GET /reports/projects`, `GET /reports/projects/{project}`; list query + detail aggregates; resources; Feature tests | ✅ Implemented |
| **7.3 Employee Reports** | `GET /reports/employees`, `GET /reports/employees/{user}`; list + detail; Feature tests | ✅ Implemented |
| **7.4 Reports Authorization** | Full Admin / PM / Employee matrix Feature tests (list + detail isolation) | ✅ Implemented |

**Phase order is mandatory.** Do not start a later phase until the prior phase is complete and approved.

---

## 4. Approved schema decisions

### Persistence

| Decision | Notes |
|----------|-------|
| New tables | **None** for Milestone 7 |
| Materialized / cache / export tables | **Deferred** |
| Activity Logs | **Out of scope** |
| Morph aliases | **No new morph aliases** |

### Soft deletes

- Soft-deleted **projects**, **tasks**, and **users** are excluded from report lists and aggregates
- Soft-deleted projects are not addressable via `{project}` route model binding (default Laravel soft-delete behavior)
- Soft-deleted users are not addressable via `{user}` for employee reports

### Overdue task definition (reuse Milestone 6)

A task is **overdue** when **all** hold:

1. Not soft-deleted  
2. `due_date` is not null  
3. `due_date` **&lt;** today’s date (app default timezone / `CURRENT_DATE` semantics)  
4. `status` is **not** `completed` and **not** `cancelled`

### Date-range filter (approved)

| Param | Rules |
|-------|--------|
| `from_date` | optional; `Y-m-d`; inclusive |
| `to_date` | optional; `Y-m-d`; inclusive; must be **≥** `from_date` when both present |

**Semantics:** When either bound is present, task aggregates include only tasks whose **`created_at` date** falls within the inclusive range (missing bound = open-ended on that side). Project identity / employee identity are not filtered by date — only which **tasks** contribute to counts.

When neither bound is present, all non-soft-deleted tasks in scope are included (same as Dashboard counts).

### Visibility scoping (approved)

#### Project reports

| Role | List / detail scope |
|------|---------------------|
| Administrator | All non-soft-deleted projects |
| Project Manager | All non-soft-deleted projects |
| Employee | Owned (`created_by`) **or** member projects only; `{project}` outside scope → `403` |

#### Employee reports

| Role | List | Detail `{user}` |
|------|------|-----------------|
| Administrator | All non-soft-deleted users (paginated) | Any non-soft-deleted user |
| Project Manager | All non-soft-deleted users (paginated) | Any non-soft-deleted user |
| Employee | **Forbidden** (`403` on list) | **Own profile only**; other users → `403` |

Do **not** invent a separate report ACL beyond these rules.

---

## 5. API contracts

### Routes (`auth:sanctum`)

| Method | Path | Name | Behavior |
|--------|------|------|----------|
| GET | `/api/v1/reports/projects` | `reports.projects.index` | Paginated project report summaries |
| GET | `/api/v1/reports/projects/{project}` | `reports.projects.show` | Single project report |
| GET | `/api/v1/reports/employees` | `reports.employees.index` | Paginated employee report summaries (Admin/PM) |
| GET | `/api/v1/reports/employees/{user}` | `reports.employees.show` | Single employee report |

No POST/PUT/PATCH/DELETE report routes in Milestone 7. No file-download routes.

### Query parameters

#### Shared (where applicable)

| Param | Rules | Default |
|-------|--------|---------|
| `from_date` | optional date `Y-m-d` | — |
| `to_date` | optional date `Y-m-d`; ≥ `from_date` | — |
| `page` | integer ≥ 1 | `1` |
| `per_page` | integer; clamp max **100** | `15` |

#### `GET /reports/projects`

| Param | Rules |
|-------|--------|
| `search` | optional string; case-insensitive match on project `name` / `description` |
| `status` | optional `ProjectStatus` enum |
| `sort` | `name`, `status`, `created_at` |
| `direction` | `asc` / `desc` (default `desc`) |
| `from_date` / `to_date` | task aggregate window |
| `page` / `per_page` | pagination |

Default sort: `created_at` / `desc` (+ `id` tie-break).

#### `GET /reports/projects/{project}`

| Param | Rules |
|-------|--------|
| `from_date` / `to_date` | task aggregate window only |

#### `GET /reports/employees`

| Param | Rules |
|-------|--------|
| `search` | optional; `first_name`, `middle_name`, `last_name`, `email` (same spirit as User list) |
| `role_id` | optional exists |
| `department_id` | optional exists |
| `status` | optional `UserStatus` |
| `sort` | `first_name`, `last_name`, `email`, `created_at` |
| `direction` | `asc` / `desc` (default `desc`) |
| `from_date` / `to_date` | task aggregate window (tasks assigned to each user) |
| `page` / `per_page` | pagination |

Default sort: `created_at` / `desc` (+ `id` tie-break).

#### `GET /reports/employees/{user}`

| Param | Rules |
|-------|--------|
| `from_date` / `to_date` | task aggregate window for that assignee |

### Success payload shapes (approved)

Standard envelope. List endpoints put items in `data` and pagination in `meta` (same keys as User/Project/Task lists).

#### Project report summary item / detail `data`

```json
{
  "project": {
    "id": 3,
    "name": "OpsFlow Launch",
    "status": "active",
    "start_date": null,
    "due_date": "2026-09-01",
    "created_at": "2026-07-01T10:00:00+00:00"
  },
  "tasks": {
    "total": 40,
    "by_status": {
      "todo": 10,
      "in_progress": 12,
      "in_review": 4,
      "blocked": 2,
      "completed": 10,
      "cancelled": 2
    },
    "by_priority": {
      "low": 5,
      "medium": 20,
      "high": 10,
      "urgent": 5
    },
    "overdue": 3,
    "unassigned": 4
  },
  "members_count": 5
}
```

**Notes:**

- `by_status` / `by_priority` **zero-fill** all enum keys (same as Dashboard)
- `unassigned` = tasks with `assigned_to` null (within date window + project)
- `members_count` = count of `project_members` rows (not filtered by date)
- Detail uses the same shape as a list item (no separate schema)
- Nested `project` is a slim summary (not full `ProjectResource` with owner) unless implementation reuses a shared summary array — either is fine if fields match

#### Employee report summary item / detail `data`

```json
{
  "user": {
    "id": 12,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com",
    "status": "active"
  },
  "tasks": {
    "total": 18,
    "by_status": { "...": 0 },
    "by_priority": { "...": 0 },
    "overdue": 2,
    "by_project": [
      {
        "project_id": 3,
        "name": "OpsFlow Launch",
        "total": 10
      }
    ]
  }
}
```

**Notes:**

- Task set = non-soft-deleted tasks where `assigned_to` = subject user id (+ date window)
- Soft-deleted projects’ tasks: if task isn’t soft-deleted but project is, **exclude** those tasks (join / whereHas non-deleted project)
- `by_project` ordered by `total` desc, then `name` asc; include only projects with `total > 0`
- List items **may omit** `by_project` for payload size **or** include it — **approved preference: include `by_project` on detail only; list summaries omit `by_project`**

**Messages (suggested):**

- List projects: `Project reports retrieved successfully.`
- Show project: `Project report retrieved successfully.`
- List employees: `Employee reports retrieved successfully.`
- Show employee: `Employee report retrieved successfully.`

### Errors

| Case | HTTP |
|------|------|
| Guest | `401` |
| Authenticated but forbidden (Employee list employees; Employee viewing other user; Employee inaccessible project) | `403` |
| Unknown / soft-deleted `{project}` or `{user}` | `404` |
| Invalid query params / bad date range | `422` |

---

## 6. Layering & classes

Mirror Dashboard / list patterns:

| Layer | Class | Responsibility |
|-------|--------|----------------|
| Controller | `App\Http\Controllers\Api\V1\ReportController` | Thin HTTP; authorize; delegate; envelope |
| Requests | `App\Http\Requests\Api\V1\Reports\IndexProjectReportsRequest` | List project report validation |
| | `App\Http\Requests\Api\V1\Reports\ShowProjectReportRequest` | Detail date-range validation |
| | `App\Http\Requests\Api\V1\Reports\IndexEmployeeReportsRequest` | List employee report validation |
| | `App\Http\Requests\Api\V1\Reports\ShowEmployeeReportRequest` | Detail date-range validation |
| Service | `App\Services\Reports\ReportService` | Visibility + aggregates + pagination orchestration |
| Policy | `App\Policies\ReportPolicy` | Coarse report abilities |
| Resources | `App\Http\Resources\Api\V1\ProjectReportResource` | Project report shaping |
| | `App\Http\Resources\Api\V1\EmployeeReportResource` | Employee report shaping |

**Optional:** `App\Queries\Reports\ProjectReportQuery` / `EmployeeReportQuery` if list filtering grows — prefer keeping Milestone 7 logic in `ReportService` (+ small private helpers) unless duplication appears.

**Do not** put aggregate SQL in the controller.

### Request paths

```text
GET /api/v1/reports/projects
  → auth:sanctum
  → ReportController::projects
  → authorize(ReportPolicy::viewAnyProjectReports)
  → IndexProjectReportsRequest
  → ReportService::projectReports(...)
  → paginated ProjectReportResource + meta

GET /api/v1/reports/projects/{project}
  → authorize(ReportPolicy::viewProjectReport, $project)
  → ShowProjectReportRequest
  → ReportService::projectReport(...)
  → ProjectReportResource

GET /api/v1/reports/employees
  → authorize(ReportPolicy::viewAnyEmployeeReports)
  → IndexEmployeeReportsRequest
  → ReportService::employeeReports(...)
  → paginated EmployeeReportResource + meta

GET /api/v1/reports/employees/{user}
  → authorize(ReportPolicy::viewEmployeeReport, $user)
  → ShowEmployeeReportRequest
  → ReportService::employeeReport(...)
  → EmployeeReportResource
```

### Gate registration

Prefer model-less abilities **or** policy methods invoked with `Project` / `User` instances:

```text
Gate::policy / $this->authorize('viewProjectReport', $project)
ReportPolicy::viewAnyProjectReports(User $actor): bool
ReportPolicy::viewProjectReport(User $actor, Project $project): bool
ReportPolicy::viewAnyEmployeeReports(User $actor): bool
ReportPolicy::viewEmployeeReport(User $actor, User $subject): bool
```

Register `ReportPolicy` via dedicated Gate defines **or** map abilities explicitly in `AppServiceProvider` (document the chosen wiring at implementation — same flexibility as Dashboard `viewDashboard`).

---

## 7. Authorization rules

| Ability | Administrator | Project Manager | Employee |
|---------|---------------|-----------------|----------|
| `viewAnyProjectReports` | ✅ | ✅ | ✅ |
| `viewProjectReport` | ✅ all | ✅ all | ✅ accessible projects only |
| `viewAnyEmployeeReports` | ✅ | ✅ | ❌ |
| `viewEmployeeReport` | ✅ any user | ✅ any user | ✅ self only |

Unauthorized → `403` API envelope. Guests → `401`.

---

## 8. Validation rules

| Field | Rules |
|-------|--------|
| `from_date` | sometimes, nullable, date_format:Y-m-d |
| `to_date` | sometimes, nullable, date_format:Y-m-d; after_or_equal:from_date when both set |
| `search` | sometimes, nullable, string, max:255 |
| `status` (projects) | sometimes, nullable, `Rule::enum(ProjectStatus)` |
| `status` (employees) | sometimes, nullable, `Rule::enum(UserStatus)` |
| `role_id` / `department_id` | sometimes, nullable, integer, exists |
| `sort` / `direction` | allow-lists per endpoint |
| `page` | sometimes, integer, min:1 |
| `per_page` | sometimes, integer, min:1, max:100 (**clamp** max like existing list requests) |

---

## 9. Query / aggregate behavior

### Project list

1. Start from `projects` (non-soft-deleted)  
2. Apply Employee visibility (owned-or-member)  
3. Apply `search` / `status` / sort / pagination  
4. For each page of projects, compute task aggregates (date-windowed) + `members_count`  
5. Prefer efficient aggregate queries (avoid N+1 row loads); batching by project ids is acceptable

### Project detail

1. Authorize project visibility  
2. Return single report shape for that project  

### Employee list

1. Authorize Admin/PM only  
2. Start from `users` (non-soft-deleted)  
3. Apply search / role / department / status / sort / pagination  
4. For each user on the page, aggregate **assigned** tasks (date-windowed; project not soft-deleted)  

### Employee detail

1. Authorize subject visibility  
2. Return detail shape including `by_project`  

### Performance (v1 guidance)

- No Redis/cache layer  
- No queued report generation  
- No CSV/PDF streaming  

---

## 10. Phase details

### Phase 7.1 — Reports API Foundation

**Deliverables:**

- [x] Route group under `/api/v1/reports` (`auth:sanctum`)
- [x] `ReportController` stub methods wired
- [x] `ReportService` skeleton
- [x] `ReportPolicy` + Gate / authorize wiring
- [x] Shared date-range validation approach
- [x] Feature tests: guest `401` on all four routes

**Out of scope for 7.1:** Full aggregate correctness (may return empty/zero scaffolds).

---

### Phase 7.2 — Project Reports

**Deliverables:**

- [x] `IndexProjectReportsRequest` / `ShowProjectReportRequest`
- [x] `ProjectReportResource`
- [x] List + detail implementations with zero-filled buckets, overdue, unassigned, members_count
- [x] Date-range semantics
- [x] Feature tests: `tests/Feature/Report/ProjectReportApiTest.php`

---

### Phase 7.3 — Employee Reports

**Deliverables:**

- [x] `IndexEmployeeReportsRequest` / `ShowEmployeeReportRequest`
- [x] `EmployeeReportResource`
- [x] List + detail; detail includes `by_project`
- [x] Feature tests: `tests/Feature/Report/EmployeeReportApiTest.php`

---

### Phase 7.4 — Reports Authorization

**Deliverables:**

- [x] Complete matrix Feature tests: `tests/Feature/Report/ReportAuthorizationTest.php`
- [x] Employee isolation for projects; self-only employee detail; employee forbidden on employee list

---

## 11. Out of Scope (Future Work)

Explicitly deferred (do **not** implement in Milestone 7):

- PDF / CSV / Excel export endpoints or packages
- Email / scheduled / saved reports
- Activity Log–based reports
- Remarks / comment analytics
- Custom report builder / ad-hoc SQL
- Caching, queues, materialized views
- Vue Reports UI / chart rendering
- Time tracking / billable hours
- Department-level or Role-level dedicated report endpoints (beyond filters on employee list)
- Replacing or changing Dashboard (`GET /api/v1/dashboard`)
- Write APIs for reports

---

## 12. Testing expectations

| Phase | Path (approved) | Coverage |
|-------|-----------------|----------|
| 7.1–7.2 | `tests/Feature/Report/ProjectReportApiTest.php` | ✅ list/detail; filters; date range; overdue/unassigned; pagination; guest `401` |
| 7.3 | `tests/Feature/Report/EmployeeReportApiTest.php` | ✅ list/detail; filters; `by_project`; pagination; validation |
| 7.4 | `tests/Feature/Report/ReportAuthorizationTest.php` | ✅ Admin / PM / Employee matrix |

**Assertions must cover:**

- Zero-fill for status/priority buckets  
- Soft-deleted projects/tasks/users excluded  
- Date range inclusive bounds + `to_date` ≥ `from_date` validation  
- Employee cannot list employee reports; cannot view other users’ reports  
- Employee cannot view inaccessible project reports  
- `per_page` clamp  

---

## 13. Acceptance Criteria

### Design package

- [x] `docs/MILESTONE_7_REPORTS.md` exists  
- [x] `decisions/Reports.md` ADR exists  
- [x] Companion docs synchronized  

### Implementation

- [x] Four report routes return approved payloads under `auth:sanctum`  
- [x] No new report/activity tables introduced  
- [x] Date-range + visibility rules match this spec  
- [x] Feature tests green for 7.1–7.4  
- [x] Docs updated to ✅ Implemented as phases complete  

---

## 14. Related documents

| Document | Use |
|----------|-----|
| [decisions/Reports.md](../decisions/Reports.md) | ADR |
| [DOMAIN_MODEL.md](DOMAIN_MODEL.md) | Domain concepts |
| [MILESTONE_6_DASHBOARD.md](MILESTONE_6_DASHBOARD.md) | Overdue definition / aggregate patterns |
| [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md) | Task visibility |
| [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) | Project visibility |
| [API_SPECIFICATION.md](../API_SPECIFICATION.md) | Endpoint contracts |
| [ROADMAP.md](../ROADMAP.md) | Phase roadmap |
| [HANDOFF.md](../HANDOFF.md) | Session handoff |
