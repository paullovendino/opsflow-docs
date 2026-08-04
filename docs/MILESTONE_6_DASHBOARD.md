# Milestone 6 — Dashboard

**Status:** ✅ Milestone 6 complete (Phases 6.1–6.4)  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-04

> Implementation specification for Milestone 6.  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)  
> ADR: [decisions/Dashboard.md](../decisions/Dashboard.md)  
> Prerequisite: Milestone 5 complete (Tasks CRUD, assignment, queries, `TaskPolicy`, status)

---

## 1. Goal

Introduce OpsFlow’s **Dashboard** module on the API as a **read-only operational summary** over existing Projects and Tasks:

- Expose project and task **statistics** (totals + chart-ready breakdowns)
- Expose **statistics cards** data (overdue tasks, assigned-to-me)
- Expose a **recent work items** feed derived from existing entities
- Enforce visibility consistent with Project / Task authorization
- Remain schema-light: **no new persistence tables** in Milestone 6

The backend returns JSON suitable for frontend charts and cards. It does **not** render charts or ship Vue UI.

---

## 2. Domain summary

```text
Authenticated User
    └── Dashboard (read model / aggregate — not a persisted entity)
            ├── Project statistics (scoped)
            ├── Task statistics (scoped)
            └── Recent work items (scoped Projects + Tasks)
```

- Dashboard is a **computed view**, not a domain table
- Aggregates use existing `projects` and `tasks` (soft-deleted rows excluded)
- **Recent activities** in Milestone 6 means **recent work items** (projects/tasks by `updated_at`), **not** the planned Activity Log module
- Remarks, Activity Logs, Reports, and Vue Dashboard UI remain out of scope

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **6.1 Dashboard API Foundation** | `DashboardController` / `DashboardService` / `DashboardResource` / route / `ShowDashboardRequest`; guest `401`; response shape | ✅ Implemented |
| **6.2 Project & Task Statistics** | Totals; `by_status` / `by_priority`; overdue; `assigned_to_me`; chart-ready breakdowns; visibility scoping in service | ✅ Implemented |
| **6.3 Recent Work Items** | Derived recent projects + tasks feed; `recent_limit` query param | ✅ Implemented |
| **6.4 Dashboard Authorization** | `DashboardPolicy` via `Gate::define('viewDashboard')` + Employee matrix Feature tests | ✅ Implemented |

**Phase order is mandatory.** Do not start a later phase until the prior phase is complete and approved.

---

## 4. Approved schema decisions

### Persistence

| Decision | Notes |
|----------|-------|
| New tables | **None** for Milestone 6 |
| Materialized / cache tables | **Deferred** |
| Activity Logs table | **Out of scope** (feeds true audit “recent activity” later) |
| Morph aliases | **No new morph aliases** |

### Soft deletes

- Soft-deleted projects and tasks are **excluded** from all dashboard counts and recent items
- Soft-deleted users are irrelevant to aggregates except as FK targets on historical rows (counts still exclude soft-deleted tasks/projects)

### Overdue task definition (approved)

A task is **overdue** when **all** of the following hold:

1. Not soft-deleted
2. `due_date` is not null
3. `due_date` **&lt;** today’s date (application date / DB `CURRENT_DATE` semantics — document timezone as app default)
4. `status` is **not** `completed` and **not** `cancelled`

### Visibility scoping (approved)

| Role | Aggregate / recent scope |
|------|---------------------------|
| Administrator | All non-soft-deleted projects and tasks |
| Project Manager | All non-soft-deleted projects and tasks |
| Employee | Projects the actor **owns** (`created_by`) **or** **belongs to** as a member; tasks belonging to those projects only |

`assigned_to_me` always counts tasks where `assigned_to` = actor id (within the actor’s visible task set for Employee; org-wide for Admin/PM among non-soft-deleted tasks assigned to them).

This mirrors Milestone 4 / 5 owned-or-member rules. Do **not** invent a separate dashboard ACL.

---

## 5. API contracts

### Route (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/dashboard` | Return dashboard summary payload |

**Named route:** `dashboard.show` (or `dashboard.index` — pick one name at implementation; prefer `dashboard.show`)

No POST/PUT/PATCH/DELETE dashboard routes in Milestone 6.

### Query parameters (`ShowDashboardRequest` / `IndexDashboardRequest`)

| Param | Rules | Default |
|-------|--------|---------|
| `recent_limit` | optional integer; min 1; max **25** (clamp or reject with `422` — prefer **clamp** to max like `per_page`) | **10** |

No search, status filters, date-range analytics, or pagination `meta` on the dashboard endpoint in Milestone 6.

### Success response shape (approved)

Standard envelope; `data` contains:

```json
{
  "projects": {
    "total": 12,
    "by_status": {
      "planning": 2,
      "active": 5,
      "on_hold": 1,
      "completed": 3,
      "archived": 1
    }
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
    "assigned_to_me": 7
  },
  "recent": [
    {
      "type": "task",
      "id": 15,
      "title": "Prepare kickoff deck",
      "status": "in_progress",
      "project_id": 3,
      "updated_at": "2026-08-03T10:00:00+00:00"
    },
    {
      "type": "project",
      "id": 3,
      "name": "OpsFlow Launch",
      "status": "active",
      "updated_at": "2026-08-02T18:00:00+00:00"
    }
  ]
}
```

**Breakdown rules:**

- `by_status` / `by_priority` keys **must include every enum value**, even when count is `0`
- Counts are non-negative integers
- `recent` is an array ordered by `updated_at` **descending** (newest first)
- Mixed types are interleaved by `updated_at` (merge projects + tasks, sort, take `recent_limit`)
- Task recent items use `title`; project recent items use `name`
- `type` is exactly `task` or `project` (string literals; may use a small PHP Enum later if desired)

**Message (suggested):** `Dashboard retrieved successfully.`

### Errors

| Case | HTTP |
|------|------|
| Guest | `401` |
| Authenticated but policy denies (none expected if all roles may view) | `403` |
| Invalid `recent_limit` (if not clamping non-integers) | `422` |

---

## 6. Layering & classes

Mirror Lookup / read-service patterns (no new Eloquent model required):

| Layer | Class | Responsibility |
|-------|--------|----------------|
| Controller | `App\Http\Controllers\Api\V1\DashboardController` | Thin HTTP; authorize; delegate; envelope |
| Request | `App\Http\Requests\Api\V1\Dashboard\ShowDashboardRequest` | Validate / normalize `recent_limit` |
| Service | `App\Services\Dashboard\DashboardService` | Visibility scoping + aggregates + recent merge |
| Policy | `App\Policies\DashboardPolicy` | Coarse `view` ability for all authenticated roles |
| Resource | `App\Http\Resources\Api\V1\DashboardResource` | Shape `data` (never raw query rows as models) |

**Optional helper (not required):** private methods or a `DashboardQuery` / aggregator class under `App\Queries\Dashboard\` if the service grows too large — prefer keeping Milestone 6 logic inside `DashboardService` unless duplication appears.

**Do not** put aggregate SQL in the controller.

### Request path

```text
GET /api/v1/dashboard
  → auth:sanctum
  → DashboardController::show
  → authorize(DashboardPolicy::view)
  → ShowDashboardRequest
  → DashboardService::summary(User $actor, int $recentLimit)
  → DashboardResource
```

---

## 7. Authorization rules

### Ability

| Ability | Who |
|---------|-----|
| `view` dashboard | All authenticated roles: Administrator, Project Manager, Employee |

Unauthorized guests → `401`. Soft-deleted / inactive users follow existing Sanctum / login rules (inactive users cannot obtain a session).

### Data scoping (enforced in service, tested in 6.4)

Employee responses must **not** include counts or recent items from projects outside owned-or-member scope (same rule as `ProjectQuery` / `TaskQuery` visibility).

Administrator and Project Manager see organization-wide aggregates.

**Out of scope:** per-project dashboard endpoints, custom widget permissions, field-level ACLs.

---

## 8. Validation rules

| Field | Rules |
|-------|--------|
| `recent_limit` | optional; integer; if present, clamp to `[1, 25]` (preferred, matching list `per_page` clamp) **or** fail `422` — **approved preference: clamp** after integer validation |

No body payload on GET.

---

## 9. Query / aggregate behavior

### Project statistics

- Source: `projects` where `deleted_at` is null
- Apply visibility scope for Employee
- `total` = count
- `by_status` = `GROUP BY status` for all `ProjectStatus` values (fill zeros)

### Task statistics

- Source: `tasks` where `deleted_at` is null
- Apply visibility scope for Employee (tasks whose `project_id` is in the actor’s accessible projects)
- `total`, `by_status` (`TaskStatus`), `by_priority` (`TaskPriority`) with zero-fill
- `overdue` per §4 definition
- `assigned_to_me` = count where `assigned_to` = actor id (scoped as above)

### Recent work items

- Candidate sets: accessible non-soft-deleted projects and tasks
- Order by `updated_at` desc, then stable tie-break (`type` + `id` if needed)
- Take first `recent_limit` after merge
- Do **not** require Activity Logs

### Performance (v1 guidance)

- Prefer a small number of aggregate queries (`COUNT` / `GROUP BY`) over loading all rows
- No Redis/cache layer in Milestone 6
- No N+1 from recent items: select only needed columns

---

## 10. Phase details

### Phase 6.1 — Dashboard API Foundation

**Deliverables:**

- [x] Route `GET /api/v1/dashboard` under `auth:sanctum`
- [x] `DashboardController::show`
- [x] `DashboardService::summary` returning the approved shape with **zeros / empty `recent`** (or wired enough for contract tests)
- [x] `ShowDashboardRequest` (`recent_limit` default/clamp)
- [x] `DashboardResource`
- [x] Feature tests: authenticated success envelope shape; guest `401`

**Out of scope for 6.1:** Full statistics correctness, recent merge, Employee scoping matrix (may stub Admin path only).

---

### Phase 6.2 — Project & Task Statistics

**Deliverables:**

- [x] Project `total` + `by_status`
- [x] Task `total` + `by_status` + `by_priority` + `overdue` + `assigned_to_me`
- [x] Visibility scoping applied to all counts
- [x] Feature tests: `tests/Feature/Dashboard/DashboardStatisticsTest.php` (or combined suite name — see §12)

**Out of scope for 6.2:** Recent feed (may remain `[]`).

---

### Phase 6.3 — Recent Work Items

**Deliverables:**

- [x] Merged recent projects + tasks
- [x] `recent_limit` respected
- [x] Feature tests for ordering, limit, type discriminators, visibility

**Out of scope for 6.3:** Activity Log entities, remark previews, actor attribution beyond what’s on project/task rows.

---

### Phase 6.4 — Dashboard Authorization

**Deliverables:**

- [x] `DashboardPolicy::view` (all authenticated roles allow)
- [x] Register policy / Gate in `AppServiceProvider` (`Gate::define('viewDashboard', [DashboardPolicy::class, 'view'])`)
- [x] `$this->authorize('viewDashboard')` in `DashboardController`
- [x] Feature tests: Employee cannot see other projects’ aggregates; Admin/PM see all; guest `401`

**Out of scope for 6.4:** Advanced RBAC, widget-level permissions.

---

## 11. Out of Scope (Future Work)

Explicitly deferred (do **not** implement in Milestone 6):

- Vue Dashboard UI / charts rendering in the browser app
- Activity Logs table / audit trail / true “who did what” feed
- Remarks / comments on the dashboard
- Reports module (Phase 7 — exports, date ranges, employee productivity reports)
- Caching, queues, materialized views, scheduled rollups
- Date-range filters, comparison periods, trends over time
- Nested `GET /api/v1/projects/{project}/dashboard`
- User directory statistics (active users, by role/department)
- Notifications, kanban widgets, calendar widgets
- WebSockets / live updates
- Changing Project or Task write APIs

---

## 12. Testing expectations

| Phase | Path (approved) | Coverage |
|-------|-----------------|----------|
| 6.1–6.3 | `tests/Feature/Dashboard/DashboardApiTest.php` | ✅ Envelope; statistics; overdue; assigned_to_me; recent merge/limit; guest `401` |
| 6.4 | `tests/Feature/Dashboard/DashboardAuthorizationTest.php` | ✅ Admin / PM / Employee visibility matrix |

Alternatively a single combined suite is acceptable if phases stay reviewable; prefer the split above for parity with Milestones 4–5.

**Assertions must cover:**

- Zero-fill for missing status/priority buckets
- Soft-deleted projects/tasks excluded
- Overdue edge cases (`completed` / `cancelled` / null `due_date` not overdue)
- Employee isolation
- `recent_limit` clamp / default

---

## 13. Acceptance Criteria

### Design package

- [x] `docs/MILESTONE_6_DASHBOARD.md` exists
- [x] `decisions/Dashboard.md` ADR exists
- [x] Companion docs synchronized (HANDOFF, ROADMAP, API_SPEC, DOMAIN_MODEL, etc.)

### Implementation

- [x] `GET /api/v1/dashboard` returns approved payload under `auth:sanctum`
- [x] No new dashboard/activity tables introduced
- [x] Statistics + recent items respect visibility scoping
- [x] Overdue definition matches this spec
- [x] Feature tests green for 6.1–6.4
- [x] Docs updated to ✅ Implemented as phases complete

---

## 14. Related documents

| Document | Use |
|----------|-----|
| [decisions/Dashboard.md](../decisions/Dashboard.md) | ADR |
| [DOMAIN_MODEL.md](DOMAIN_MODEL.md) | Domain concepts |
| [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md) | Task visibility / enums |
| [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) | Project visibility / enums |
| [API_SPECIFICATION.md](../API_SPECIFICATION.md) | Endpoint contract |
| [ROADMAP.md](../ROADMAP.md) | Phase roadmap |
| [HANDOFF.md](../HANDOFF.md) | Session handoff |
