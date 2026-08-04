# Architecture

## Overview

OpsFlow uses a separated frontend/backend architecture:

- `opsflow-api` — Laravel 13 REST API + PostgreSQL
- `opsflow-web` — Vue 3 SPA (TypeScript, Pinia, Axios)
- `opsflow-docs` — Product and engineering documentation

The API is the system of record. The SPA consumes `/api/v1` with Sanctum SPA cookie authentication.

Domain reference: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)

---

## High-Level Components

```text
┌─────────────────┐     cookie + CSRF      ┌──────────────────┐
│  opsflow-web    │ ─────────────────────► │   opsflow-api    │
│  Vue 3 SPA      │ ◄───────────────────── │   Laravel 13     │
└─────────────────┘                        │   Sanctum SPA    │
                                           │   PostgreSQL     │
                                           └──────────────────┘
```

---

## Organizational Domain Architecture

OpsFlow models a single logical organization:

```text
Organization (logical, single-tenant v1.0)
    ├── Departments     → grouping
    ├── Job Titles      → positions
    ├── Roles           → permissions
    └── Users           → people
```

These concepts are **independent**. A user has one Role (required) and optional Department and Job Title.

See [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) and [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md).

---

## Backend Layering

Follows `CODING_STANDARDS.md`:

| Layer | Role |
|-------|------|
| Controllers (`Http/Controllers/Api/V1`) | HTTP in/out only |
| Form Requests | Validation |
| API Resources | Response shaping |
| Services | Business logic |
| Queries | List search / filter / sort / pagination |
| Models | Persistence / relationships |
| Enums | Domain constants |
| Policies | Coarse authorization (`UserPolicy`, `ProjectPolicy`, `TaskPolicy`, `DashboardPolicy` via `viewDashboard` Gate; `ReportPolicy` via report Gates) |
| Exceptions + `ApiExceptionRenderer` | Consistent API errors |

Authentication example:

`AuthController` → `LoginRequest` → `AuthenticationService` → `User` / session → `UserResource`

User Management example (Phase 3.3 — implemented):

`UserController` → `StoreUserRequest` / `UpdateUserRequest` / `UpdateUserStatusRequest` → `UserService` → `User` + relations → `UserResource` (+ nested `RoleResource` / `DepartmentResource` / `JobTitleResource`)

User list example (Phase 3.5 — implemented):

`UserController::index` → `IndexUsersRequest` → `UserService::list` → `UserQuery` → paginated `UserResource` collection + `meta`

Lookups example (Phase 3.4):

`LookupController` → `LookupService` → `Role` / `Department` / `JobTitle` → `RoleResource` / `DepartmentResource` / `JobTitleResource`

Project Management example (Phases 4.2–4.4 — implemented):

`ProjectController` → Form Requests → `ProjectService` → `Project` / members → `ProjectResource` / `ProjectMemberResource`

Project list example (Phase 4.4 — implemented):

`ProjectController::index` → `IndexProjectsRequest` → `ProjectService::list` → `ProjectQuery` → paginated `ProjectResource` collection + `meta`

Task Management example (Milestone 5 — implemented):

`TaskController` → authorize(`TaskPolicy`) → Form Requests → `TaskService` → `Task` → `TaskResource`  
`index` → `IndexTasksRequest` → `TaskService::list` → `TaskQuery` (+ Employee visibility) → paginated `TaskResource` + `meta`  
Status via `PATCH .../status` (`UpdateTaskStatusRequest` / `TaskService::changeStatus`)

Dashboard example (Milestone 6 — implemented):

`DashboardController` → authorize(`viewDashboard`) → `ShowDashboardRequest` → `DashboardService::summary` → `DashboardResource`  
Read-only aggregates over `projects` / `tasks`; no new tables; recent = derived work items

Reports example (Milestone 7 — implemented):

`ReportController` → authorize(`ReportPolicy` Gates) → Form Requests → `ReportService` → `ProjectReportResource` / `EmployeeReportResource`  
Read-only Project / Employee aggregates; optional date range; no new tables

---

## API Versioning

- Prefix: `/api/v1`
- Controllers under `App\Http\Controllers\Api\V1`
- Shared base: `App\Http\Controllers\Api\BaseApiController`
- Shared envelope helper: `App\Traits\ApiResponse`

---

## Authentication Architecture

- Laravel Sanctum SPA cookie mode
- `web` guard for session auth
- `auth:sanctum` for protected API routes (including `/users`)
- Named `login` RateLimiter
- `guest` middleware on login
- Inactive accounts blocked at login (`403` / `Account is inactive.`)

Details: [AUTHENTICATION.md](AUTHENTICATION.md)

### Frontend consumer (Milestone 8 — ✅ implemented)

SPA auth consumption is specified in [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) and [decisions/Frontend-Foundation.md](decisions/Frontend-Foundation.md): Axios `withCredentials`, CSRF cookie, Pinia auth store, Vue Router guards. **Implemented in `opsflow-web`.**

---

## Authorization Architecture

**Phase 3.6 — Implemented.** Coarse role checks for User Management — not advanced RBAC:

| Role | User Management |
|------|-----------------|
| Administrator | Full user management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

Enforced via `App\Policies\UserPolicy` and `$this->authorize()` in `UserController`. Unauthorized → HTTP `403` API envelope.

Lookup endpoints (Phase 3.4): `GET /api/v1/lookups/{roles,departments,job-titles}` — all authenticated users; collections only.

**Phase 4.5 — Implemented.** Coarse role checks for Project Management:

| Role | Project Management |
|------|--------------------|
| Administrator | Full access to all projects |
| Project Manager | Full access to all projects |
| Employee | List/view owned or member projects only |

Enforced via `App\Policies\ProjectPolicy` and `$this->authorize()` in `ProjectController`. Employee list scoping in `ProjectQuery`. Unauthorized → HTTP `403` API envelope.

**Phase 5.5 — Implemented.** Coarse role checks for Task Management:

| Role | Task Management |
|------|-----------------|
| Administrator | Full access to all tasks |
| Project Manager | Full access to all tasks |
| Employee | List/view tasks in accessible projects; update status only when assigned to self |

Enforced via `App\Policies\TaskPolicy`. See [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md).

**Phase 6.4 — Implemented.** Coarse dashboard view + data scoping:

| Role | Dashboard |
|------|-----------|
| Administrator | View; org-wide aggregates |
| Project Manager | View; org-wide aggregates |
| Employee | View; owned-or-member scoped aggregates |

Enforced via `Gate::define('viewDashboard', [DashboardPolicy::class, 'view'])` + scoping in `DashboardService`. See [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md).

**Phase 7.4 — Implemented.** Project / Employee report abilities:

| Role | Project reports | Employee report list | Employee report detail |
|------|-----------------|----------------------|------------------------|
| Administrator | all | ✅ | any user |
| Project Manager | all | ✅ | any user |
| Employee | owned-or-member | ❌ | self only |

Enforced via `ReportPolicy` Gate abilities + scoping in `ReportService`. See [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md).

---

## Frontend Architecture (`opsflow-web`)

> Milestone 8 — ✅ Complete · [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md)  
> Milestone 9 — 🔄 In progress (✅ 9.1) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

| Layer | Role |
|-------|------|
| Views | Thin pages under `modules/*/views` (+ auth/error views) |
| Layouts | `GuestLayout`, `AuthLayout` (nested routes) |
| Modules | `dashboard` (✅), `users` / `projects` / `tasks` / `reports` (planned) |
| Router | History mode; `requiresAuth` / `guest`; `/` → `/dashboard` |
| Pinia | `auth`, `ui` (module list state via composables by default) |
| Services | Axios `http` + domain `*Service.ts` (`dashboardService` ✅) |
| Components | Shared UI + layout; Tailwind only (no chart lib / UI kit) |
| Types | Envelope + domain shapes |

Feature module pages ship phase-by-phase under Milestone 9; implement only after explicit phase approval.

---

## Cross-Cutting Concerns

### CORS

- Env-driven `CORS_ALLOWED_ORIGINS`
- Credentials enabled for SPA cookie auth
- Paths include `api/*` and `sanctum/csrf-cookie`

### Exception Handling

API routes (`api/*`) render through `App\Exceptions\ApiExceptionRenderer` using the standard envelope.

### Morph Map

`Relation::enforceMorphMap` registers aliases only for existing models.

| Status | Aliases |
|--------|---------|
| Registered | `user`, `role`, `department`, `job_title`, `project`, `task` |
| Later | `remark`, `activity_log` |

### Reference Data

- Roles: seeded; lookup via `/api/v1/lookups/roles`
- Departments: seeded; soft deletes; lookup via `/api/v1/lookups/departments`
- Job Titles: seeded; soft deletes; lookup via `/api/v1/lookups/job-titles`

---

## Implemented vs Planned

| Area | Status |
|------|--------|
| API foundation | ✅ Implemented |
| Authentication (API) | ✅ Implemented |
| Organization Foundation (3.1) | ✅ Implemented |
| User Domain Foundation (3.2) | ✅ Implemented |
| User Management APIs (3.3) | ✅ Implemented |
| Lookup APIs (3.4) | ✅ Implemented |
| Search / filters / sorting / pagination (3.5) | ✅ Implemented |
| Coarse authorization (3.6) | ✅ Implemented |
| **Milestone 3** | ✅ **Complete** |
| Phase 4.1 — Project Domain Foundation | ✅ Implemented |
| Phase 4.2 — Project CRUD | ✅ Implemented |
| Phase 4.3 — Project Members | ✅ Implemented |
| Phase 4.4 — Project Queries | ✅ Implemented |
| Phase 4.5 — Project Authorization | ✅ Implemented |
| **Milestone 4** | ✅ **Complete** (4.1–4.5) |
| Phase 5.1 — Task Domain Foundation | ✅ Implemented |
| Phase 5.2 — Task CRUD | ✅ Implemented |
| Phase 5.3 — Task Assignment | ✅ Implemented |
| Phase 5.4 — Task Queries | ✅ Implemented |
| Phase 5.5 — Task Authorization | ✅ Implemented |
| Phase 5.6 — Task Status Workflow | ✅ Implemented |
| **Milestone 5** | ✅ **Complete** (5.1–5.6) |
| Phase 6.1 — Dashboard API Foundation | ✅ Implemented |
| Phase 6.2 — Project & Task Statistics | ✅ Implemented |
| Phase 6.3 — Recent Work Items | ✅ Implemented |
| Phase 6.4 — Dashboard Authorization | ✅ Implemented |
| **Milestone 6 — Dashboard** | ✅ **Complete** (6.1–6.4) |
| Phase 7.1 — Reports API Foundation | ✅ Implemented |
| Phase 7.2 — Project Reports | ✅ Implemented |
| Phase 7.3 — Employee Reports | ✅ Implemented |
| Phase 7.4 — Reports Authorization | ✅ Implemented |
| **Milestone 7 — Reports** | ✅ **Complete** (7.1–7.4) |
| Phase 8 — Frontend Foundation | ✅ **Complete** (8.1–8.3) |
| Phase 9 — Frontend Modules | 🔄 In progress (✅ 9.1 Dashboard UI) |
| Phase 10 — Testing | ⏳ Pending |
| Remarks / Activity Logs | Planned |
| Phase 11 — Deployment | Planned |
