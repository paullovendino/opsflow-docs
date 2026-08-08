# OpsFlow — Project Handoff

**Audience:** New Cursor / AI development session  
**Purpose:** Continue OpsFlow development without losing architectural consistency  
**Last updated:** 2026-08-08  
**Product version:** v1.0.0 (Development)

> **Start here.** Then read [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md), [ROADMAP.md](ROADMAP.md), and the relevant ADR under `decisions/`.  
> **Do not invent architecture.** If unclear, ask. Prefer ADRs in `decisions/`.  
> **Do not implement the next phase until the user explicitly approves scope.**

---

## 1. Project purpose

OpsFlow is a production-quality SaaS **project and operations management** platform for teams.

It lets administrators, project managers, and employees:

- Organize projects and tasks
- Assign work
- Monitor progress
- Improve operational visibility via dashboards / reports (API complete)

**Organizational people model:**

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

**Repositories (sibling folders under `OpsFlow/`):**

| Repo           | Role                                                          |
| -------------- | ------------------------------------------------------------- |
| `opsflow-api`  | Laravel REST API (active backend)                             |
| `opsflow-docs` | Documentation + ADRs (this repo)                              |
| `opsflow-web`  | Vue 3 SPA (M8 + **M9 complete** · ✅ 9.1–9.5) |

**Current status:** ✅ **Milestone 9 — Frontend Modules complete** (Phases 9.1–9.5, including post-ship CRUD/loading/lookup UX). Next: **Phase 10 — Testing & QA** — **wait for explicit implementation approval**. After Phase 10: **Phase 11 — Deployment**.

| Field | Value |
|-------|--------|
| Current milestone | **Milestone 9 — Frontend Modules** ✅ **Complete** |
| Current phase | ✅ 9.1 · ✅ 9.2 · ✅ 9.3 · ✅ 9.4 · ✅ 9.5 · next is **Phase 10 Testing & QA** (awaiting approval) |
| Completed (M8) | ✅ 8.1 · ✅ 8.2 · ✅ 8.3 |
| Completed (M9) | ✅ 9.1 Dashboard · ✅ 9.2 Users · ✅ 9.3 Projects · ✅ 9.4 Tasks · ✅ 9.5 Reports · post-ship UX/performance |
| Planned (M9) | — (none remaining) |

---

## 2. Current architecture

Separated SPA + API:

```text
opsflow-web (Vue)  --cookie + CSRF-->  opsflow-api (Laravel)  -->  PostgreSQL
```

### Backend layering (must preserve)

| Layer                  | Responsibility                      |
| ---------------------- | ----------------------------------- |
| Controllers `Api/V1`   | Thin HTTP only                      |
| Form Requests          | Validation                          |
| API Resources          | Response shaping (never raw models) |
| Services               | Business logic                      |
| Models                 | Persistence / relationships         |
| Enums                  | Domain constants (no magic strings) |
| Policies               | Coarse authorization (`UserPolicy`; `ProjectPolicy`; `TaskPolicy`; `DashboardPolicy` / `viewDashboard`; `ReportPolicy` Gates) |
| Queries                | List search / filter / sort / paginate (`UserQuery`; `ProjectQuery`; `TaskQuery`) |
| `ApiExceptionRenderer` | Consistent API errors (incl. project member `409`) |

### Request paths (implemented)

```text
Auth:     AuthController → LoginRequest → AuthenticationService → UserResource
Users:    UserController → authorize(UserPolicy) → Form Request → UserService → UserResource
          index → IndexUsersRequest → UserService → UserQuery → UserResource + pagination meta
Projects: ProjectController → authorize(ProjectPolicy) → Form Request → ProjectService → ProjectResource
          index → IndexProjectsRequest → ProjectService → ProjectQuery → ProjectResource + pagination meta
          members → StoreProjectMemberRequest → ProjectService → ProjectMemberResource
Tasks:    TaskController → authorize(TaskPolicy) → Form Request → TaskService → TaskResource
          index → IndexTasksRequest → TaskService → TaskQuery (+ Employee visibility) → TaskResource + pagination meta
          assignment → UpdateTaskAssignmentRequest → TaskService::changeAssignment
          status → UpdateTaskStatusRequest → TaskService::changeStatus
Dashboard: DashboardController → authorize(viewDashboard) → ShowDashboardRequest → DashboardService → DashboardResource
Reports:  ReportController → authorize(ReportPolicy Gates) → Form Requests → ReportService → ProjectReportResource / EmployeeReportResource
Lookups:  LookupController → LookupService → RoleResource / DepartmentResource / JobTitleResource
```

Details: [ARCHITECTURE.md](ARCHITECTURE.md), [CODING_STANDARDS.md](CODING_STANDARDS.md), [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)

### Folder structure (`opsflow-api`)

```text
app/
├── Actions/
├── Enums/                  # RoleName, UserStatus, DepartmentCode, JobTitleCode, ProjectStatus, TaskStatus, TaskPriority
├── Exceptions/             # ApiExceptionRenderer, InvalidCredentialsException, AccountInactiveException, DuplicateProjectMemberException
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/         # … DashboardController, ReportController
│   ├── Middleware/
│   ├── Requests/Api/V1/…   # Auth/, Users/, Projects/, Tasks/, Dashboard/, Reports/
│   └── Resources/Api/V1/…  # … Dashboard, ProjectReport, EmployeeReport
├── Models/                 # User, Role, Department, JobTitle, Project, Task
├── Policies/               # UserPolicy, ProjectPolicy, TaskPolicy, DashboardPolicy, ReportPolicy
├── Providers/              # AppServiceProvider (morph map, RateLimiter, Gate::policy, Gate::define)
├── Queries/
│   ├── Users/UserQuery.php
│   ├── Projects/ProjectQuery.php
│   └── Tasks/TaskQuery.php
├── Repositories/
├── Services/
│   ├── Auth/AuthenticationService.php
│   ├── Lookups/LookupService.php
│   ├── Users/UserService.php
│   ├── Projects/ProjectService.php
│   ├── Tasks/TaskService.php
│   ├── Dashboard/DashboardService.php
│   └── Reports/ReportService.php
└── Traits/                 # ApiResponse (incl. paginatedResponse)
```

Routes: `routes/api.php` (prefixed `/api` + `v1` groups)

---

## 3. Tech stack

### Backend (`opsflow-api`)

- **Laravel 13** (do **not** downgrade to Laravel 12)
- **PHP 8.3+**
- **PostgreSQL** only (app DB; not SQLite for runtime)
- **Laravel Sanctum** — SPA cookie authentication
- **REST API** under `/api/v1`

### Frontend (`opsflow-web` — Milestone 8 complete)

- Vue 3 Composition API + TypeScript
- Pinia, Vue Router, Tailwind CSS, Axios (`withCredentials: true`)
- Spec: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) · ADR: [decisions/Frontend-Foundation.md](decisions/Frontend-Foundation.md)
- Auth shell: Guest/Auth layouts, login/logout/`/me` bootstrap
- Feature modules (M9): Dashboard · Users · Projects · Tasks · Reports — [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)
- Post-ship UX: CRUD list+modal aliases, skeleton loading, `AppProgressBar`, `useLookups` SPA-session cache
- Frontend `npm run type-check` and `npm run build` pass; backend APIs were not changed for these UX/performance improvements
- Next: Phase 10 — Testing & QA (awaiting approval) · then Phase 11 — Deployment

### Tooling

- Cursor, Git/GitHub, Postman/Bruno/Insomnia, pgAdmin, PHPUnit

ADR: [decisions/Tech-Stack.md](decisions/Tech-Stack.md)

---

## 4. Completed milestones

| Phase | Milestone | Status |
| ----- | --------- | ------ |
| 1 | Backend Foundation (API) | **Completed** |
| 2 | Authentication (API) | **Completed** |
| **3** | **Organization & User Management** | ✅ **Complete** |
| 3.1 | Organization Foundation | ✅ **Implemented** |
| 3.2 | User Domain Foundation | ✅ **Implemented** |
| 3.3 | User Management APIs | ✅ **Implemented** |
| 3.4 | Lookup APIs | ✅ **Implemented** |
| 3.5 | Search, Filtering & Pagination | ✅ **Implemented** |
| 3.6 | Authorization (RBAC) | ✅ **Implemented** |
| **4** | **Project Management** | ✅ **Complete** |
| 4.1 | Project Domain Foundation | ✅ **Implemented** |
| 4.2 | Project CRUD | ✅ **Implemented** |
| 4.3 | Project Members | ✅ **Implemented** |
| 4.4 | Project Queries | ✅ **Implemented** |
| 4.5 | Project Authorization | ✅ **Implemented** |
| **5** | **Task Management** | ✅ **Complete** |
| 5.1 | Task Domain Foundation | ✅ **Implemented** |
| 5.2 | Task CRUD | ✅ **Implemented** |
| 5.3 | Task Assignment | ✅ **Implemented** |
| 5.4 | Task Queries | ✅ **Implemented** |
| 5.5 | Task Authorization | ✅ **Implemented** |
| 5.6 | Task Status Workflow | ✅ **Implemented** |
| **6** | **Dashboard** | ✅ **Complete** |
| 6.1 | Dashboard API Foundation | ✅ **Implemented** |
| 6.2 | Project & Task Statistics | ✅ **Implemented** |
| 6.3 | Recent Work Items | ✅ **Implemented** |
| 6.4 | Dashboard Authorization | ✅ **Implemented** |
| **7** | **Reports** | ✅ **Complete** |
| 7.1 | Reports API Foundation | ✅ **Implemented** |
| 7.2 | Project Reports | ✅ **Implemented** |
| 7.3 | Employee Reports | ✅ **Implemented** |
| 7.4 | Reports Authorization | ✅ **Implemented** |
| **8** | **Frontend Foundation** | ✅ **Complete** |
| 8.1 | Application Foundation | ✅ **Implemented** |
| 8.2 | Authentication Foundation | ✅ **Implemented** |
| 8.3 | UI Foundation | ✅ **Implemented** |
| **9** | **Frontend Modules** | ✅ **Complete** (9.1–9.5) |
| 9.1 | Dashboard UI | ✅ **Implemented** |
| 9.2 | User Management | ✅ **Implemented** |
| 9.3 | Project Management | ✅ **Implemented** |
| 9.4 | Task Management | ✅ **Implemented** |
| 9.5 | Reports | ✅ **Implemented** |

**Phase 1:** PostgreSQL, Sanctum, `/api/v1`, CORS, envelope, exception renderer, morph map, roles seed, health check, folder structure.

**Phase 2:** login/logout/me, rate limit, guest login, credential allowlist, feature tests, auth docs.

**Phase 3.1:** `departments` / `job_titles` tables (`name` + `code`), models, soft deletes, seeders, morph aliases.

**Phase 3.2:** Users ERD (`role_id`, names, `department_id`, `job_title_id`, `status`, soft deletes, keep `email_verified_at`), relations, `UserStatus`, inactive login `403`, expanded `UserResource`.

**Phase 3.3:** User CRUD + status patch (`UserController` / `UserService` / Form Requests / nested resources).

**Phase 3.4:** Lookup collections under `/api/v1/lookups` (`LookupController` / `LookupService`).

**Phase 3.5:** User list search / filters / sorting / pagination (`UserQuery` / `IndexUsersRequest`).

**Phase 3.6:** Coarse User Management authorization (`UserPolicy` + controller `$this->authorize()`).

**Phase 4.1:** `projects` / `project_members` tables; `Project` model; `ProjectStatus`; relations; morph `project`; `ProjectFactory`; Feature tests.

**Phase 4.2:** Project CRUD + status patch (`ProjectController` / `ProjectService` / Form Requests / `ProjectResource`).

**Phase 4.4:** Project list search / filters / sorting / pagination (`ProjectQuery` / `IndexProjectsRequest`).

**Phase 4.5:** Coarse `ProjectPolicy` authorization (Admin/PM full; Employee owned-or-member list/view).

**Phase 5.1:** `tasks` table; `Task` model; `TaskStatus` / `TaskPriority`; relations; morph `task`; `TaskFactory`; Feature tests.

**Phase 5.2:** Task CRUD (`TaskController` / `TaskService` / Form Requests / `TaskResource`); create defaults (`todo` / `medium`); optional create-time `assigned_to`; soft delete.

**Phase 5.3:** Task Assignment (`PATCH .../assignment`); active + owner/member validation; nullable unassign.

**Phase 5.4:** Task list search / filters / sorting / pagination (`TaskQuery` / `IndexTasksRequest`).

**Phase 5.5:** Coarse `TaskPolicy` authorization; Employee owned-or-member list/view scoping.

**Phase 5.6:** Task status patch (`PATCH .../status`); any `TaskStatus`; no transition graph.

**Phase 6.1–6.4:** Dashboard read-only summary (`GET /api/v1/dashboard`); statistics + recent work items; `DashboardPolicy` / `viewDashboard`; no new tables.

**Phase 7.1–7.4:** Reports (`/api/v1/reports/projects`, `/api/v1/reports/employees`); date range; `ReportPolicy` Gates; no new tables.

**Phase 8.1–8.3:** `opsflow-web` Frontend Foundation — Pinia/Router/Axios/Tailwind; Sanctum CSRF auth; Guest/Auth shell.

**Phase 9.1:** Dashboard UI — `/dashboard` landing; `GET /api/v1/dashboard`; stat cards, Tailwind status bars, recent work; skeleton / empty / error+retry.

**Phase 9.2:** User Management UI — list/search/filters/pagination; Create/Edit/View via **modals** on the list (`AppModal`); `/users/:id` + `/profile` pages for deep links; status/delete confirms; lookups; teleported `AppDropdownMenu`; Clear filter always visible (disabled when idle); shared table/form primitives.

**Phase 9.3:** Project Management UI — list/search/filters/pagination; Create/Edit/View via **modals** on the list (`ProjectFormDialog` / `ProjectDetailDialog`); `/projects/create` and `/projects/:id/edit` keep the list mounted (stable view key); `/projects/:id` Show **page** (workspace: info + members + `ProjectTasksPanel`); status patch `AppModal`; soft delete + member remove via `AppConfirmDialog`; member add inline on Show.

**Phase 9.4:** Task Management UI — table list (not Kanban); Create/Edit/View **modals** (`TaskFormDialog` / `TaskDetailDialog`); `/tasks/:id` Show **page**; assignment via `TaskAssignmentDialog` / detail panel; status patch; soft delete confirm; `ProjectTasksPanel` on Project Show; Admin/PM mutate; Employee list/view + status when assigned to self.

**Phase 9.5:** Reports UI — `/reports` → project reports; `/reports/projects`, `/reports/projects/:id`, `/reports/employees`, `/reports/employees/:id`; `reportService`; `ReportDateFilters`; reuse `DashboardStatCard` / `DashboardStatusBar` (**Tailwind bars only** — no chart library); date filters; role-aware nav (Employee list Admin/PM; Employee self via My report); no exports.

**Post-ship UX / perceived performance (still Milestone 9, no new APIs):**

- Shared skeletons: `AppSkeleton`, `AppTableSkeleton`, `AppCardSkeleton`, `AppDetailSkeleton`, `AppReportSkeleton`; module list skeletons reuse them
- `AppProgressBar`: route navigation + page-level HTTP; **`/api/v1/lookups/*` excluded**; Create/Edit modal alias routes do not start route progress
- Soft refresh: keep prior list/detail data visible; opacity while refreshing; loading ≠ empty state
- `useLookups`: module-level **in-memory SPA-session cache** + in-flight dedupe (not Pinia, not localStorage); Users search/filters/pagination remain server-side
- AuthLayout stable `viewKey` for Users/Projects/Tasks Create/Edit aliases so the underlying list is not remounted

Sidebar (M9): Dashboard, Users (role), Projects, Tasks, Reports, Employee reports (Admin/PM) / My report (Employee), Profile — no “Coming later” for M9 modules.

See [CHANGELOG.md](CHANGELOG.md), [ROADMAP.md](ROADMAP.md).

---

## 5. Remaining roadmap

### Immediate next

**Phase 10 — Testing & QA** (see §12) — wait for explicit implementation approval

Specification: [TESTING.md](TESTING.md) · [ROADMAP.md](ROADMAP.md)

Completed: Milestones 1–9 (including Phases 9.1–9.5).

### Still pending

- Phase 10 — Testing & QA
- Phase 11 — Deployment
- GitHub repos finalized (as applicable)

### Later phases (do not invent early)

Kanban and other v1.2+ items after core module UIs.

### Future versions

Notifications, remarks, kanban, time tracking, mobile, multi-org, etc. ([ROADMAP.md](ROADMAP.md))

### Explicitly out of scope for Milestone 9

- Chart libraries / UI frameworks / Kanban
- PDF/CSV exports, Activity Logs / Remarks UI
- Automated test suite (Phase 10) · Deployment (Phase 11)
- Backend schema / new API endpoints

---

## 6. Authentication contract

**Mode:** Sanctum SPA cookie auth (`web` guard + session), not bearer-token login for the SPA.

| Method | Path                   | Middleware                | Notes              |
| ------ | ---------------------- | ------------------------- | ------------------ |
| GET    | `/sanctum/csrf-cookie` | —                         | CSRF cookie        |
| POST   | `/api/v1/auth/login`   | `guest`, `throttle:login` | Session login      |
| POST   | `/api/v1/auth/logout`  | `auth:sanctum`            | Invalidate session |
| GET    | `/api/v1/auth/me`      | `auth:sanctum`            | Current user       |

### Behaviors

- Login: attempt **only** `email` + `password`; regenerate session; set `last_login_at`
- Eager-load `role`, `department`, `jobTitle` for auth responses
- Logout: logout + invalidate session + regenerate CSRF token
- Invalid credentials → `401` (`InvalidCredentialsException`)
- Inactive account → `403` (`AccountInactiveException`, message `Account is inactive.`)
- Validation failure → `422`
- Already authenticated login → `403` (`Already authenticated.`)
- Rate limit: named limiter `login` — **5/min** per `email|ip` → `429`
- Out of scope: register, reset password, email verify, social login, advanced RBAC

### Key classes

- `App\Http\Controllers\Api\V1\AuthController`
- `App\Services\Auth\AuthenticationService`
- `App\Http\Requests\Api\V1\Auth\LoginRequest`
- `App\Http\Resources\Api\V1\UserResource`
- `App\Exceptions\AccountInactiveException`

### SPA client requirements

- `Origin`/`Referer` on stateful domains (local: `http://localhost:5173`)
- Axios `withCredentials: true` + `withXSRFToken: true`
- CSRF header from `XSRF-TOKEN` cookie
- Frontend consumer design: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) · [decisions/Frontend-Foundation.md](decisions/Frontend-Foundation.md) (**implemented**)

Docs: [AUTHENTICATION.md](AUTHENTICATION.md), [decisions/Authentication.md](decisions/Authentication.md), [API_SPECIFICATION.md](API_SPECIFICATION.md)

---

## 7. User Management implementation summary (Phase 3.3)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/users` | List with search/filters/sorting/pagination (Phase 3.5) |
| POST | `/api/v1/users` | Create (`StoreUserRequest`) |
| GET | `/api/v1/users/{user}` | Show |
| PUT | `/api/v1/users/{user}` | Update (`UpdateUserRequest`; password optional) |
| DELETE | `/api/v1/users/{user}` | Soft delete |
| PATCH | `/api/v1/users/{user}/status` | Status only (`UpdateUserStatusRequest`; `active` / `inactive`) |

**Auth:** `auth:sanctum`  
**Authorization:** `UserPolicy` (Phase 3.6) — Administrator full; Project Manager read-only; Employee own profile only

**Behaviors:** `UserService` owns create/update/delete/status/list; list delegates to `UserQuery`; passwords via `Hash::make`; nested resources via `whenLoaded()`.

### Key classes

- `App\Http\Controllers\Api\V1\UserController`
- `App\Services\Users\UserService`
- `App\Queries\Users\UserQuery`
- `App\Http\Requests\Api\V1\Users\IndexUsersRequest`
- `App\Http\Requests\Api\V1\Users\StoreUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserStatusRequest`
- `App\Enums\UserStatus`
- `App\Policies\UserPolicy`
- Resources: `UserResource`, `RoleResource`, `DepartmentResource`, `JobTitleResource`

---

## 8. Lookup API implementation summary (Phase 3.4)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/lookups/roles` | `RoleResource` collection; ordered by `name` |
| GET | `/api/v1/lookups/departments` | Soft-deleted excluded; ordered by `name` |
| GET | `/api/v1/lookups/job-titles` | Soft-deleted excluded; ordered by `name` |

**Auth:** `auth:sanctum` (all authenticated users)  
**Classes:** `LookupController`, `LookupService`  
**Scope:** Collections only — no show/`{id}`, no CRUD, no filters/pagination on lookups.

---

## 8b. Search, Filtering & Pagination summary (Phase 3.5)

| Concern | Behavior |
|---------|----------|
| Search | `search` on `first_name`, `middle_name`, `last_name`, `email` (case-insensitive) |
| Filtering | `role_id`, `department_id`, `job_title_id`, `status` (composable) |
| Sorting | `sort` + `direction`; allowed `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100 clamped); standard `meta` |

**Classes:** `IndexUsersRequest`, `UserQuery`, `UserService::list()`, `paginatedResponse` + `UserResource::collection`

---

## 8c. Authorization summary (Phase 3.6)

| Role | List | View | Create | Update | Delete | Status |
|------|------|------|--------|--------|--------|--------|
| Administrator | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Employee | ❌ | own only | ❌ | ❌ | ❌ | ❌ |

**Classes:** `UserPolicy`; `Gate::policy` in `AppServiceProvider`; `$this->authorize()` in `UserController`

---

## 8d. Project CRUD summary (Phase 4.2)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/projects` | List with search/filters/sorting/pagination (Phase 4.4) |
| POST | `/api/v1/projects` | Create; `created_by` = auth user; status always `planning` |
| GET | `/api/v1/projects/{project}` | Show with nested `owner` |
| PUT | `/api/v1/projects/{project}` | Update name/description/dates only (not status/owner) |
| DELETE | `/api/v1/projects/{project}` | Soft delete |
| PATCH | `/api/v1/projects/{project}/status` | Status only (`ProjectStatus`) |

**Auth:** `auth:sanctum`  
**Authorization:** `ProjectPolicy` (Phase 4.5) — Administrator & Project Manager full; Employee owned/member list+view only

**Classes:** `ProjectController`, `ProjectService`, `ProjectQuery`, `ProjectPolicy`, `IndexProjectsRequest`, `StoreProjectRequest`, `UpdateProjectRequest`, `UpdateProjectStatusRequest`, `ProjectResource`

---

## 8e. Project Members summary (Phase 4.3)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/projects/{project}/members` | List members (`ProjectMemberResource`) |
| POST | `/api/v1/projects/{project}/members` | Add member (`user_id`) |
| DELETE | `/api/v1/projects/{project}/members/{user}` | Remove member (pivot hard delete) |

**Auth:** `auth:sanctum`  
**Authorization:** `ProjectPolicy` — manage members: Admin/PM; list members: same as project `view`

**Approved business rules:**

- Owner (`created_by`) is independent from membership — **not** auto-added to `project_members`
- `joined_at` is server-generated only; client-supplied `joined_at` is ignored
- Duplicate membership → HTTP `409` (`DuplicateProjectMemberException` / `User is already a member of this project.`)
- Only **active**, non-soft-deleted users may be added (`422` otherwise)
- Removing a member does not change `created_by`
- Unknown membership on delete → `404`

**Classes:** `ProjectController` member actions; `ProjectService::listMembers` / `addMember` / `removeMember`; `StoreProjectMemberRequest`; `ProjectMemberResource`

---

## 8f. Project Queries summary (Phase 4.4)

| Concern | Behavior |
|---------|----------|
| Search | `search` on `name`, `description` (case-insensitive) |
| Filtering | `status`, `created_by` (composable) |
| Sorting | `sort` + `direction`; allowed `name`, `status`, `start_date`, `due_date`, `created_at`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100 clamped); standard `meta` |

**Classes:** `IndexProjectsRequest`, `ProjectQuery`, `ProjectService::list()`, `paginatedResponse` + `ProjectResource::collection`

---

## 8g. Project Authorization summary (Phase 4.5)

| Role | List | View | Create | Update | Delete | Status | Manage members |
|------|------|------|--------|--------|--------|--------|----------------|
| Administrator | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Employee | owned **or** member | owned **or** member | ❌ | ❌ | ❌ | ❌ | ❌ |

**Classes:** `ProjectPolicy`; `Gate::policy` in `AppServiceProvider`; `$this->authorize()` in `ProjectController`; Employee list scoped in `ProjectQuery::applyVisibility`

---

## 8h. Task Domain Foundation summary (Phase 5.1)

| Concern | Notes |
|---------|-------|
| Table | `tasks` — soft deletes; FKs `project_id`, `assigned_to` (nullable), `created_by` (**RESTRICT**) |
| Enums | `TaskStatus` (`todo` default), `TaskPriority` (`medium` default) |
| Relations | `Task` ↔ `Project` / assignee / creator; `User::createdTasks` / `assignedTasks`; `Project::tasks` |
| Morph | `task` → `App\Models\Task` |
| Factory | `TaskFactory` |
| Tests | `tests/Feature/Task/TaskDomainFoundationTest.php` |

**Classes:** `App\Models\Task`, `App\Enums\TaskStatus`, `App\Enums\TaskPriority`, `Database\Factories\TaskFactory`

---

## 8i. Task CRUD summary (Phase 5.2)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/tasks` | List with search/filters/sorting/pagination (Phase 5.4) |
| POST | `/api/v1/tasks` | Create; `created_by` = auth user; status always `todo`; optional `assigned_to` |
| GET | `/api/v1/tasks/{task}` | Show with nested `project`, `assignee`, `creator` |
| PUT | `/api/v1/tasks/{task}` | Update title/description/priority/due_date only |
| DELETE | `/api/v1/tasks/{task}` | Soft delete |

**Auth:** `auth:sanctum`  
**Authorization:** `TaskPolicy` (Phase 5.5) — Administrator & Project Manager full; Employee owned/member list+view only

**Approved business rules:**

- Create status always `todo` (client-supplied `status` ignored)
- Default priority `medium` when omitted
- `created_by` always the authenticated user (client-supplied ignored)
- Optional `assigned_to` on create only — must be active + project owner or member (`422` otherwise)
- Update does **not** accept `status`, `assigned_to`, `project_id`, or `created_by`
- Soft delete only; soft-deleted tasks excluded from default list

**Classes:** `TaskController`, `TaskService`, `StoreTaskRequest`, `UpdateTaskRequest`, `TaskResource`

---

## 8j. Task Assignment summary (Phase 5.3)

| Method | Path | Notes |
|--------|------|-------|
| PATCH | `/api/v1/tasks/{task}/assignment` | Set or clear `assigned_to` |

**Auth:** `auth:sanctum`  
**Authorization:** `TaskPolicy` — manage assignment: Admin/PM

**Approved business rules:**

- `assigned_to` key required (`present`); value may be `null` to unassign
- When non-null: active, not soft-deleted, and project owner **or** member (`422` otherwise)
- Replaces previous assignee (single assignee only)
- Does not change `status`, `created_by`, or `project_id`

**Classes:** `TaskController::updateAssignment`, `TaskService::changeAssignment`, `UpdateTaskAssignmentRequest`

---

## 8k. Task Queries summary (Phase 5.4)

| Concern | Behavior |
|---------|----------|
| Search | `search` on `title`, `description` (case-insensitive) |
| Filtering | `status`, `priority`, `project_id`, `assigned_to`, `created_by` (composable) |
| Sorting | `sort` + `direction`; allowed `title`, `status`, `priority`, `due_date`, `created_at`; default `created_at` / `desc` + `id` tie-break |
| Pagination | `page` / `per_page` (default 15, max 100 clamped); standard `meta` |

**Classes:** `IndexTasksRequest`, `TaskQuery`, `TaskService::list()`, `paginatedResponse` + `TaskResource::collection`

---

## 8l. Task Authorization summary (Phase 5.5)

| Role | List | View | Create | Update | Delete | Status | Assignment |
|------|------|------|--------|--------|--------|--------|------------|
| Administrator | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Employee | owned **or** member projects | owned **or** member projects | ❌ | ❌ | ❌ | assigned to self only | ❌ |

**Classes:** `TaskPolicy`; `Gate::policy` in `AppServiceProvider`; `$this->authorize()` in `TaskController`; Employee list scoped in `TaskQuery::applyVisibility`

**Note:** `updateStatus` ability is defined in `TaskPolicy` and enforced on `PATCH .../status` (Phase 5.6).

---

## 8m. Task Status Workflow summary (Phase 5.6)

| Method | Path | Notes |
|--------|------|-------|
| PATCH | `/api/v1/tasks/{task}/status` | Status only; any `TaskStatus` value |

**Auth:** `auth:sanctum`  
**Authorization:** `TaskPolicy::updateStatus` — Admin/PM full; Employee assigned to self + accessible project

**Approved business rules:**

- Create/update bodies remain status-free (always `todo` on create)
- No transition graph
- Does not change title, priority, assignee, dates, or project

**Classes:** `TaskController::updateStatus`, `TaskService::changeStatus`, `UpdateTaskStatusRequest`

---

## 8n. Dashboard summary (Milestone 6)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/dashboard` | Read-only summary; optional `recent_limit` (default 10, max 25, clamp) |

**Auth:** `auth:sanctum`  
**Authorization:** `Gate::define('viewDashboard')` → `DashboardPolicy::view` — all authenticated roles; data scoped Admin/PM org-wide, Employee owned-or-member

**Approved business rules:**

- No new tables; aggregates from `projects` + `tasks`
- Statistics: project/task totals + by_status / by_priority; overdue; assigned_to_me
- Recent = derived work items (not Activity Logs)
- Soft-deleted rows excluded; enum buckets zero-filled

**Classes:** `DashboardController`, `DashboardService`, `ShowDashboardRequest`, `DashboardResource`, `DashboardPolicy`

---

## 8o. Reports summary (Milestone 7)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/reports/projects` | Paginated project report summaries |
| GET | `/api/v1/reports/projects/{project}` | Single project report |
| GET | `/api/v1/reports/employees` | Paginated employee summaries (Admin/PM) |
| GET | `/api/v1/reports/employees/{user}` | Single employee report |

**Auth:** `auth:sanctum`  
**Authorization:** `ReportPolicy` Gates — Project reports scoped like Projects; Employee list Admin/PM; Employee detail self-only for Employees

**Approved business rules:**

- No new tables; aggregates from `projects` + `tasks` + `users`
- Optional `from_date` / `to_date` filters tasks by `created_at` date
- Overdue reuses Milestone 6 definition; zero-filled enum buckets
- Distinct from Dashboard snapshot API

**Classes:** `ReportController`, `ReportService`, Form Requests, `ProjectReportResource`, `EmployeeReportResource`, `ReportPolicy`

---

## 9. Current database status

### Implemented tables

- **`roles`**: seeded `administrator`, `project_manager`, `employee` (`RoleName` enum); no soft deletes in Milestone 3
- **`departments`**: human-readable `name` + unique `code` + soft deletes; seeded (Administration/`ADMIN`, …)
- **`job_titles`**: same shape; seeded (Project Manager/`PM`, …)
- **`users`** (ERD aligned):
  - `role_id` (required, RESTRICT)
  - `department_id` / `job_title_id` (nullable, RESTRICT)
  - `first_name`, `middle_name` (nullable), `last_name` (nullable for legacy name-split compatibility)
  - `email`, `email_verified_at` (kept), `password`, `avatar` (nullable)
  - `status` (`UserStatus`: `active` / `inactive`)
  - `last_login_at` (nullable)
  - timestamps + soft deletes
- **`projects`** (Phase 4.1):
  - `name`, `description` (nullable), `status` (`ProjectStatus`), `start_date` / `due_date` (nullable)
  - `created_by` → `users.id` (**RESTRICT**)
  - timestamps + soft deletes
- **`project_members`** (Phase 4.1):
  - `project_id`, `user_id` (**RESTRICT**), `joined_at`, timestamps
  - unique (`project_id`, `user_id`); no soft deletes; no member roles
- **`tasks`** (Phase 5.1):
  - `project_id` → `projects.id` (**RESTRICT**)
  - `title`, `description` (nullable), `status` (`TaskStatus`), `priority` (`TaskPriority`)
  - `due_date` (nullable), `assigned_to` → `users.id` (nullable, **RESTRICT**), `created_by` → `users.id` (**RESTRICT**)
  - timestamps + soft deletes
- Sanctum **`personal_access_tokens`**, sessions / cache / jobs as Laravel defaults require

### Morph map (`Relation::enforceMorphMap`)

- `user` → `App\Models\User`
- `role` → `App\Models\Role`
- `department` → `App\Models\Department`
- `job_title` → `App\Models\JobTitle`
- `project` → `App\Models\Project`
- `task` → `App\Models\Task`

### Not yet

- `organizations` table / multi-tenant org settings
- Activity Logs / Remarks tables

ADRs: [DATABASE_DESIGN.md](DATABASE_DESIGN.md), [decisions/Database.md](decisions/Database.md), [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md), [decisions/Project-Management.md](decisions/Project-Management.md), [decisions/Task-Management.md](decisions/Task-Management.md), [decisions/Dashboard.md](decisions/Dashboard.md), [decisions/Reports.md](decisions/Reports.md)

---

## 10. Engineering conventions

### Response envelope

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

- Success: `success: true`, `errors: null`
- Errors: `success: false`; validation details in `errors` (HTTP `422`)
- Pagination (Phase 3.5 — implemented): items in `data`, page info in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`)
- Use `App\Traits\ApiResponse` via `BaseApiController` (`paginatedResponse` for list endpoints)
- Never return raw Eloquent models
- Auth note: login puts the user under **`data.user`**; `/me` returns the user object in **`data`**

### Must preserve

1. **Ask before assuming** unclear requirements
2. **No unapproved packages**
3. **`/api/v1` only** for versioned API routes
4. **Standard response envelope** on all API success/error paths
5. **Services for business logic**; thin controllers; **query objects** for list search/filter/sort/pagination
6. **Form Requests + API Resources** (Form Requests not required for parameter-less lookup GETs)
7. **PHP Enums** instead of magic strings
8. **Sanctum SPA cookies** for first-party auth — do not replace with ad-hoc JWT without an ADR
9. **Guest + `throttle:login`** on login; **`auth:sanctum`** on logout/me/users/lookups/projects/tasks/dashboard/reports
10. **Credential allowlist** into `Auth::attempt()`
11. **Morph aliases only for existing models**
12. **Do not expand schema beyond the approved ERD** without updating `decisions/Database.md`
13. **Do not implement later modules early** (Activity Logs / Kanban) unless approved; do not implement Phase 10/11 until explicitly approved
14. **Authorize via Policies** — do not scatter manual role checks outside policies (`UserPolicy`, `ProjectPolicy`, `TaskPolicy`, `DashboardPolicy` / `viewDashboard`, `ReportPolicy` Gates)
15. **CORS / Sanctum domains stay environment-driven**
16. **Update docs/ADRs when behavior changes**
17. **One feature = one commit** when asked to commit
18. **Roles ≠ Departments ≠ Job Titles** — keep concepts independent
19. **Read domain model** (`docs/DOMAIN_MODEL.md`) and milestone specs before inventing entities
20. **Project owner ≠ automatic member** — `created_by` and `project_members` are independent; duplicate members → `409`
21. **Task status / assignment patches are separate endpoints** — create/update bodies do not change assignment after create or status
22. **Dashboard is schema-neutral in Milestone 6** — aggregates only; no Activity Logs table for “recent”
23. **Reports are schema-neutral in Milestone 7** — Project/Employee read models; no export tables; do not replace Dashboard

### Coding standards snapshot

- Laravel 13 + PSR-12 + `declare(strict_types=1)`
- DI; no magic strings; no duplicated logic
- Match patterns: `UserController` / `UserService` / `UserQuery` / `UserPolicy`; `LookupController` / `LookupService`; `ProjectController` / `ProjectService` / `ProjectQuery` / `ProjectPolicy`; `TaskController` / `TaskService` / `TaskQuery` / `TaskPolicy`; `DashboardController` / `DashboardService` / `DashboardPolicy` / `DashboardResource`; `ReportController` / `ReportService` / `ReportPolicy` / report Resources

Details: [CODING_STANDARDS.md](CODING_STANDARDS.md), [CURSOR_RULES.md](CURSOR_RULES.md)

---

## 11. Known technical debt

1. **`phpunit.xml` embeds DB credentials** — move to env/local override later
2. **Laravel 13 HTTP tests** — guard caching requires `forgetGuards()` in some multi-request auth tests
3. **Draw.io diagrams empty** — future docs milestone
4. **No Postman/Bruno collection checked in** — future
5. **Feature Vue module UIs** — Milestone 9 complete (9.1–9.5 + post-ship UX/performance); frontend automated tests still Phase 10
6. **Historical “Laravel 12” wording** in old git commits vs actual Laravel 13
7. **Resource wrapping inconsistency** — login uses `data.user` + `resolve()`; `/me` uses Resource in `data` (intentional for now)
8. **Environment-config hardening** deferred (secrets, production cookie domain strategy)
9. **`last_name` nullable** — supports legacy name split; CRUD validation currently requires `last_name` on create/update
10. **Combined commit naming** — `opsflow-api` commit `ef14ade` (`feat(lookup)`) includes both Phase 3.3 User Management APIs and Phase 3.4 Lookup APIs

---

## 12. Immediate next milestone

### Phase 10 — Testing & QA

**Status:** 📋 Pending — **wait for explicit Phase 10 approval before coding.**

**Completed:** Milestones 1–9 (Frontend Modules 9.1–9.5 + post-ship CRUD/loading/lookup UX). Backend APIs were not changed for those frontend UX/performance improvements. `opsflow-web` type-check and production build pass.

**Prior milestone:** [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md) · ADR: [decisions/Frontend-Modules.md](decisions/Frontend-Modules.md)

**Milestone 9 (complete):**

| Phase | Scope | Status |
|-------|--------|--------|
| 9.1 | Dashboard UI (`GET /dashboard`; landing redirect) | ✅ Implemented |
| 9.2 | User Management | ✅ Implemented |
| 9.3 | Project Management (list+modal Create/Edit; Show workspace page) | ✅ Implemented |
| 9.4 | Task Management (table; Create/Edit/View **modals**) | ✅ Implemented |
| 9.5 | Reports (Tailwind bars; no exports) | ✅ Implemented |

**Phase 10 focus (when approved):** API test gaps · frontend automated tests · bug fixes — see [TESTING.md](TESTING.md) · [ROADMAP.md](ROADMAP.md)

**Still out of scope until approved:** Deployment (Phase 11) · Kanban · chart libraries · exports


---

## Current API routes (implemented)

| Method | Path | Auth | Authorization |
|--------|------|------|---------------|
| GET | `/api/v1/health` | none | — |
| GET | `/sanctum/csrf-cookie` | none | — |
| POST | `/api/v1/auth/login` | guest + throttle | — |
| POST | `/api/v1/auth/logout` | `auth:sanctum` | — |
| GET | `/api/v1/auth/me` | `auth:sanctum` | — |
| GET | `/api/v1/users` | `auth:sanctum` | Admin, Project Manager |
| POST | `/api/v1/users` | `auth:sanctum` | Administrator |
| GET | `/api/v1/users/{user}` | `auth:sanctum` | Admin, PM; Employee own only |
| PUT | `/api/v1/users/{user}` | `auth:sanctum` | Administrator |
| DELETE | `/api/v1/users/{user}` | `auth:sanctum` | Administrator |
| PATCH | `/api/v1/users/{user}/status` | `auth:sanctum` | Administrator |
| GET | `/api/v1/lookups/roles` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/lookups/departments` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/lookups/job-titles` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/projects` | `auth:sanctum` | Admin, PM (all); Employee owned/member |
| POST | `/api/v1/projects` | `auth:sanctum` | Administrator, Project Manager |
| GET | `/api/v1/projects/{project}` | `auth:sanctum` | Admin, PM (all); Employee owned/member |
| PUT | `/api/v1/projects/{project}` | `auth:sanctum` | Administrator, Project Manager |
| DELETE | `/api/v1/projects/{project}` | `auth:sanctum` | Administrator, Project Manager |
| PATCH | `/api/v1/projects/{project}/status` | `auth:sanctum` | Administrator, Project Manager |
| GET | `/api/v1/projects/{project}/members` | `auth:sanctum` | Same as project `view` |
| POST | `/api/v1/projects/{project}/members` | `auth:sanctum` | Administrator, Project Manager |
| DELETE | `/api/v1/projects/{project}/members/{user}` | `auth:sanctum` | Administrator, Project Manager |
| GET | `/api/v1/tasks` | `auth:sanctum` | Admin, PM (all); Employee accessible projects |
| POST | `/api/v1/tasks` | `auth:sanctum` | Administrator, Project Manager |
| GET | `/api/v1/tasks/{task}` | `auth:sanctum` | Admin, PM (all); Employee accessible projects |
| PUT | `/api/v1/tasks/{task}` | `auth:sanctum` | Administrator, Project Manager |
| DELETE | `/api/v1/tasks/{task}` | `auth:sanctum` | Administrator, Project Manager |
| PATCH | `/api/v1/tasks/{task}/assignment` | `auth:sanctum` | Administrator, Project Manager |
| PATCH | `/api/v1/tasks/{task}/status` | `auth:sanctum` | Admin, PM; Employee assigned to self only |
| GET | `/api/v1/dashboard` | `auth:sanctum` | All authenticated; data scoped by role |
| GET | `/api/v1/reports/projects` | `auth:sanctum` | Admin, PM (all); Employee owned/member |
| GET | `/api/v1/reports/projects/{project}` | `auth:sanctum` | Admin, PM (all); Employee owned/member |
| GET | `/api/v1/reports/employees` | `auth:sanctum` | Administrator, Project Manager |
| GET | `/api/v1/reports/employees/{user}` | `auth:sanctum` | Admin, PM; Employee self only |

---

## Milestone 3 decisions (approved)

1. Department seeds: Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`
2. Job Title seeds: Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`
3. User list (Phase 3.5): `search`, filters (`role_id`, `department_id`, `job_title_id`, `status`), **sorting**, pagination `meta` — **implemented**
4. Inactive users cannot log in → HTTP `403` (`Account is inactive.`) — **implemented**
5. Keep `email_verified_at` for future compatibility — **implemented**
6. FK **RESTRICT** for Department, Job Title, and Role — **implemented**
7. Lookup collections under `/api/v1/lookups` — **implemented** (Phase 3.4)
8. Coarse authorization matrix — **implemented** (Phase 3.6)

Specs: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) · [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) · [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md)

---

## Testing status

- PHPUnit via `php artisan test`
- DB: PostgreSQL **`opsflow_testing`** (`phpunit.xml`; no SQLite driver in current PHP)
- `RefreshDatabase`; session driver `cookie` in tests
- Stateful SPA simulated with `Origin: http://localhost:5173`
- After logout/guest follow-ups, tests may call `auth()->forgetGuards()` (Laravel 13 test-client quirk)
- **Last known full suite:** 212 tests passed (Phases 1–7.4 / Milestone 7 complete)

| Suite | Path | Coverage |
|-------|------|----------|
| Auth | `tests/Feature/Auth/AuthenticationTest.php` | login/logout/me, guest, validation |
| Organization | `tests/Feature/Organization/OrganizationFoundationTest.php` | dept/job title migrate/seed/unique/soft delete |
| User domain | `tests/Feature/User/UserDomainFoundationTest.php` | users ERD, relations, `full_name`, inactive login |
| User APIs | `tests/Feature/User/UserManagementApiTest.php` | CRUD, status, validation, hashing, resources |
| User list query | `tests/Feature/User/UserListQueryTest.php` | search, filters, sort, pagination, clamp, validation |
| User authz | `tests/Feature/User/UserAuthorizationTest.php` | Admin / PM / Employee policy matrix |
| Lookups | `tests/Feature/Lookup/LookupApiTest.php` | `/lookups/*` auth, soft-delete exclusion, sort by name |
| Project domain | `tests/Feature/Project/ProjectDomainFoundationTest.php` | projects/members schema, relations, soft delete, enum, FK RESTRICT, factory |
| Project APIs | `tests/Feature/Project/ProjectManagementApiTest.php` | CRUD, status, validation, owner assignment, guest `401`, resource shape |
| Project members | `tests/Feature/Project/ProjectMembersApiTest.php` | list/add/remove, duplicate `409`, active-only, soft-deleted rejected |
| Project list query | `tests/Feature/Project/ProjectListQueryTest.php` | search, filters, sort, pagination, clamp, validation |
| Project authz | `tests/Feature/Project/ProjectAuthorizationTest.php` | Admin / PM / Employee owned-or-member matrix |
| Task domain | `tests/Feature/Task/TaskDomainFoundationTest.php` | tasks schema, relations, enums, soft delete, FK RESTRICT, factory, morph |
| Task APIs | `tests/Feature/Task/TaskManagementApiTest.php` | CRUD, validation, defaults, `created_by`, create-time assignment, guest `401`, resource shape |
| Task assignment | `tests/Feature/Task/TaskAssignmentApiTest.php` | assign/clear; owner/member; inactive/soft-deleted/non-member `422`; guest `401` |
| Task list query | `tests/Feature/Task/TaskListQueryTest.php` | search, filters, sort, pagination, clamp, validation |
| Task authz | `tests/Feature/Task/TaskAuthorizationTest.php` | Admin / PM / Employee owned-or-member matrix; status ability |
| Task status | `tests/Feature/Task/TaskStatusApiTest.php` | status patch; all enums; create/update status-free; Employee assigned-to-self; guest `401` |
| Dashboard APIs | `tests/Feature/Dashboard/DashboardApiTest.php` | envelope; statistics; overdue; assigned_to_me; recent; clamp; validation; guest `401` |
| Dashboard authz | `tests/Feature/Dashboard/DashboardAuthorizationTest.php` | Admin / PM / Employee visibility matrix |
| Project reports | `tests/Feature/Report/ProjectReportApiTest.php` | list/detail; filters; date range; overdue/unassigned; pagination; guest `401` |
| Employee reports | `tests/Feature/Report/EmployeeReportApiTest.php` | list/detail; `by_project`; date range; soft-deleted project exclusion |
| Report authz | `tests/Feature/Report/ReportAuthorizationTest.php` | Admin / PM / Employee matrix |

```bash
php artisan test
```

Deferred: dedicated `429` test, CSRF failure cases, frontend automated tests (Phase 10), Activity Logs suites.

Details: [TESTING.md](TESTING.md)

---

## Current Git status

_(Re-check with `git status` before committing.)_

### `opsflow-api`

- Branch: `feature/backend-foundation` (tracks `origin/feature/backend-foundation`)
- Working tree may include **uncommitted** Phase 3.5–3.6 implementation — re-check `git status` before committing
- Recent commits (historical):
  - `ef14ade` `feat(lookup): implement lookup APIs` — includes Phase **3.3** User Management + Phase **3.4** Lookups
  - `2db5137` `feat(user): implement user domain foundation`
  - `d43af15` `feat(organization): add organization foundation`
  - `cfacf33` `feat(auth): implement authentication module`
  - `1ed3bb7` `feat: setup backend foundation`

### `opsflow-docs`

- Branch: `main` (tracks `origin/main`)
- Working tree typically has **uncommitted** Phase 4.3 / Milestone 4 documentation sync — commit when asked

### Monorepo folder `OpsFlow/`

- Not a meaningful single git root; treat **api** and **docs** as separate repos
- `opsflow-web/` Milestone 8 + Milestone 9 complete (Dashboard + Users + Projects + Tasks + Reports)

**Convention:** one feature = one commit when asked to commit.

---

## Documentation index

| Document                 | Use                                   |
| ------------------------ | ------------------------------------- |
| `README.md`              | Docs index + status                   |
| `HANDOFF.md`             | **This handoff**                      |
| `docs/DOMAIN_MODEL.md`   | **Primary business domain reference** |
| `docs/MILESTONE_3_…`     | Milestone 3 implementation spec       |
| `docs/MILESTONE_4_…`     | Milestone 4 implementation spec       |
| `docs/MILESTONE_5_…`     | Milestone 5 implementation spec       |
| `docs/MILESTONE_6_…`     | Milestone 6 implementation spec       |
| `docs/MILESTONE_7_…`     | Milestone 7 implementation spec       |
| `docs/MILESTONE_8_…`     | Milestone 8 Frontend Foundation spec  |
| `docs/MILESTONE_9_…`     | Milestone 9 Frontend Modules spec     |
| `PROJECT_OVERVIEW.md`    | Product overview                      |
| `REQUIREMENTS.md`        | Functional requirements               |
| `ARCHITECTURE.md`        | System architecture                   |
| `AUTHENTICATION.md`      | Auth module guide                     |
| `API_SPECIFICATION.md`   | Endpoint contracts                    |
| `DATABASE_DESIGN.md`     | ERD                                   |
| `CODING_STANDARDS.md`    | Conventions                           |
| `CURSOR_RULES.md`        | Agent rules                           |
| `SETUP.md`               | Local setup                           |
| `TESTING.md`             | Test guide                            |
| `ROADMAP.md`             | Canonical roadmap                     |
| `DEVELOPMENT_ROADMAP.md` | Pointer to `ROADMAP.md`               |
| `CHANGELOG.md`           | Milestone changelog                   |
| `UI_PAGES.md`            | UI inventory                          |
| `decisions/`             | ADRs (incl. `Frontend-Foundation.md`, `Frontend-Modules.md`) |
| `diagrams/`              | Draw.io placeholders (empty — future) |

---

## Quick start for the next session

1. Read this handoff + [TESTING.md](TESTING.md) · [ROADMAP.md](ROADMAP.md)
2. Confirm git branch/status in `opsflow-api` / `opsflow-docs` / `opsflow-web`
3. ✅ Milestone 9 (Phases 9.1–9.5 + UX/performance polish) is complete — next is **Phase 10 — Testing & QA** — **wait for explicit approval**
4. Do **not** invent backend APIs; do **not** implement Phase 10/11 until approved
5. Do **not** modify code when the user asks for docs-only tasks
6. SPA auth shell + all M9 module UIs are live (Dashboard, Users, Projects, Tasks, Reports)

### Essential commands

```bash
# API
cd opsflow-api
composer install
cp .env.example .env   # if needed
php artisan migrate
php artisan db:seed
php artisan serve
php artisan test

# Web
cd opsflow-web
npm install
cp .env.example .env
npm run dev

# Docs
cd opsflow-docs
# edit markdown only unless asked otherwise
```

---

## Document control

| Field      | Value                                          |
| ---------- | ---------------------------------------------- |
| Location   | `opsflow-docs/HANDOFF.md`                      |
| Supersedes | Ad-hoc chat memory                             |
| Maintain   | Update when milestones complete or ADRs change |
| Ready for  | Next session → **Phase 10 — Testing & QA** (after approval) |

## Project Principles

OpsFlow follows these engineering principles:

- Architecture first, implementation second.
- Documentation is updated before a milestone is considered complete.
- One milestone at a time; do not implement future modules.
- Controllers must remain thin.
- Business logic belongs in Services.
- Validation belongs in Form Requests.
- API responses must always use the approved response envelope.
- API Resources must be used instead of returning raw models.
- Prefer PHP Enums over magic strings.
- Use Laravel best practices and PSR-12.
- Database changes require approval before implementation.
- No additional packages without approval.
- Keep the code simple, maintainable, and production-ready.
- Every completed milestone should include:
  - Passing tests
  - Updated documentation
  - Git commit
  - Git push
