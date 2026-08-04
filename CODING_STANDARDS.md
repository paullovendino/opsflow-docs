# OpsFlow Coding Standards

## General

- Follow Laravel 13 best practices.
- Follow PSR-12 coding standards.
- Keep controllers thin.
- Move business logic into Services.
- Use Form Requests for validation.
- Use API Resources for responses.
- Prefer Eloquent relationships over raw queries.
- Use dependency injection.
- Use strict typing where possible (`declare(strict_types=1)`).
- Use PHP Enums instead of magic strings.
- Avoid duplicated code.
- Do not introduce packages unless approved.

---

## Backend Structure

```text
app/
├── Actions/
├── Enums/
├── Exceptions/
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/
│   ├── Middleware/
│   ├── Requests/
│   └── Resources/
├── Models/
├── Policies/
├── Providers/
├── Queries/          # List/search/filter/sort/pagination query objects
├── Repositories/
├── Services/
├── Traits/
```

---

## API

- Prefix all routes with `/api/v1`.
- Return the approved JSON response envelope (see below).
- Never return raw models.
- Always use API Resources.
- Use proper HTTP status codes.
- Version controllers under `App\Http\Controllers\Api\V1`.

### Response Envelope

All API responses must use this structure:

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

Rules:

- Success responses set `success` to `true` and `errors` to `null`.
- Error responses set `success` to `false` and place details in `errors` when applicable.
- Validation failures use HTTP `422` with field errors in `errors`.
- Pagination puts records in `data` and page info in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`).
- Use `App\Traits\ApiResponse` via `BaseApiController`.

---

## Database

- PostgreSQL
- JSONB where appropriate
- Soft Deletes when appropriate
- Foreign keys
- Proper indexes
- Laravel Enums
- Morph relationships where applicable
- Register morph aliases only for existing models via `Relation::enforceMorphMap`
- Add morph aliases incrementally as models are introduced
- Follow `DATABASE_DESIGN.md` and `docs/DOMAIN_MODEL.md` for schema/domain alignment
- Do not invent tables/columns beyond the approved ERD without an ADR update

### Organizational independence

- Roles = permissions
- Departments = grouping
- Job Titles = positions
- Do not conflate these concepts in models, APIs, or seeders
---

## Authentication

- Laravel Sanctum SPA cookie authentication
- Stateful domains and CORS must remain environment-driven
- Do not invent auth flows that conflict with `decisions/Authentication.md`
- Keep auth business logic in `App\Services\Auth\AuthenticationService`
- Validate login with Form Requests; respond with API Resources
- Protect authenticated routes with `auth:sanctum`
- Keep login guest-only and rate-limited (`guest`, `throttle:login`)
- Pass only `email` and `password` into `Auth::attempt()`
- Reject inactive accounts with dedicated `403` (`AccountInactiveException`)

Details: [AUTHENTICATION.md](AUTHENTICATION.md)

---

## User Management (Phase 3.3–3.6)

- Thin `UserController`; business logic in `App\Services\Users\UserService`
- List query concerns in `App\Queries\Users\UserQuery`; validate index params with `IndexUsersRequest`
- Validate mutations with `StoreUserRequest` / `UpdateUserRequest` / `UpdateUserStatusRequest`
- Authorize with `App\Policies\UserPolicy` via `$this->authorize()` (no scattered role checks)
- Respond with `UserResource` and nested `RoleResource` / `DepartmentResource` / `JobTitleResource` via `whenLoaded()`
- Hash passwords with `Hash::make`; never return passwords; omit password on update to leave unchanged
- Soft-delete users only
- Status patch updates `status` only (`UserStatus`)
- Unauthorized → HTTP `403` via `ApiExceptionRenderer` envelope

---

## Lookups (Phase 3.4)

- Thin `LookupController`; business logic in `App\Services\Lookups\LookupService`
- Routes under `/api/v1/lookups` (`roles`, `departments`, `job-titles`) — collections only (no show)
- Respond with `RoleResource` / `DepartmentResource` / `JobTitleResource`
- Exclude soft-deleted departments and job titles; order by `name`
- Protect with `auth:sanctum` (all authenticated users)

---

## Project Management (Milestone 4 — Phases 4.1–4.5 implemented · complete)

> Query and Policy layers are implemented.

- Thin `ProjectController`; business logic in `App\Services\Projects\ProjectService`
- List search/filter/sort/pagination via `ProjectQuery` / `IndexProjectsRequest`
- Validate mutations with `StoreProjectRequest` / `UpdateProjectRequest` / `UpdateProjectStatusRequest` / `StoreProjectMemberRequest`
- Authorize with `App\Policies\ProjectPolicy` via `$this->authorize()`
- Respond with `ProjectResource` / `ProjectMemberResource`
- Soft-delete projects only; status patch updates `status` only (`ProjectStatus`)
- Create always sets status `planning`; `created_by` set server-side; not transferable
- Members via `project_members` — no pivot roles/invitations; owner not auto-added; duplicate → `409`
- Follow [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) and [decisions/Project-Management.md](decisions/Project-Management.md)

---

## Task Management (Milestone 5 — ✅ Complete)

> Follow [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md) and [decisions/Task-Management.md](decisions/Task-Management.md). Do not invent beyond the approved ADR.

- Thin `TaskController`; business logic in `App\Services\Tasks\TaskService`
- Validate with `StoreTaskRequest` / `UpdateTaskRequest` / `UpdateTaskAssignmentRequest` / `UpdateTaskStatusRequest` / `IndexTasksRequest`
- List search/filter/sort/pagination via `TaskQuery` / `IndexTasksRequest`
- Authorize with `App\Policies\TaskPolicy` via `$this->authorize()`
- Status-only updates via `PATCH /api/v1/tasks/{task}/status`
- Respond with `TaskResource` (nested project / assignee / creator when loaded)
- Soft-delete tasks only; assignment patch updates `assigned_to` only; status patch updates `status` only (`TaskStatus`)
- Create always sets status `todo`; default priority `medium`; `created_by` set server-side; single optional assignee

---

## Dashboard (Milestone 6 — ✅ Complete)

> Follow [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) and [decisions/Dashboard.md](decisions/Dashboard.md). Do not invent beyond the approved ADR.

- Thin `DashboardController`; business logic in `App\Services\Dashboard\DashboardService`
- Validate query params with `ShowDashboardRequest` (`recent_limit`)
- Authorize with `Gate::define('viewDashboard', [DashboardPolicy::class, 'view'])` via `$this->authorize('viewDashboard')`
- Respond with `DashboardResource` (never raw models / query rows)
- Read-only aggregates over existing `projects` / `tasks` — **no new tables**
- Recent feed = derived work items (not Activity Logs)
- Visibility: Admin/PM org-wide; Employee owned-or-member (same rules as Project/Task)

---

## Reports (Milestone 7 — ✅ Complete)

> Follow [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) and [decisions/Reports.md](decisions/Reports.md). Do not invent beyond the approved ADR.

- Thin `ReportController`; business logic in `App\Services\Reports\ReportService`
- Validate with Form Requests under `Http/Requests/Api/V1/Reports/`
- Authorize with `ReportPolicy` Gate abilities via `$this->authorize()`
- Respond with `ProjectReportResource` / `EmployeeReportResource`
- Read-only aggregates over existing `projects` / `tasks` / `users` — **no new tables**
- Optional `from_date` / `to_date` filters tasks by `created_at` date
- Visibility: Project reports like Projects; Employee list Admin/PM; Employee detail self-only for Employees

---

## Frontend

- Vue 3 Composition API only.
- Use TypeScript.
- Use Pinia.
- Use Axios (`withCredentials: true`, `withXSRFToken: true` for Sanctum SPA auth across origins).
- Use Vue Router guards.
- Use Tailwind CSS (no UI component framework unless approved via ADR).
- Keep components small and reusable.
- Milestone 8 structure: nested `layouts/` + `views/`, `services/` for HTTP, `stores/` for Pinia — see [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md).
- Login auth errors stay inline on the login form; do not auto-route every API `403` to `/403`.
- Milestone 9 feature UIs live under `modules/` with domain `*Service.ts` and composables — see [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md).
- Phase 9.1 Dashboard UI is implemented (`dashboardService`, `useDashboard`, `modules/dashboard`).
- Do not call Users/Projects/Tasks/Reports APIs for product UI until the relevant Milestone 9 phase is approved.
- No chart libraries or Kanban in Milestone 9 (locked).

---

## Git

One feature = One commit.

Use descriptive commit messages.
