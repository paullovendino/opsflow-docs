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
| Policies | Coarse authorization (`UserPolicy`, `ProjectPolicy`; `TaskPolicy` in Milestone 5) |
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

Task Management example (Milestone 5 — designed):

`TaskController` → Form Requests → `TaskService` → `Task` → `TaskResource`  
`index` → `IndexTasksRequest` → `TaskService` → `TaskQuery` → paginated `TaskResource` + `meta` (Phase 5.5)  
Authorize via `TaskPolicy` (Phase 5.6)

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

**Phase 5.6 — Designed.** Coarse role checks for Task Management:

| Role | Task Management |
|------|-----------------|
| Administrator | Full access to all tasks |
| Project Manager | Full access to all tasks |
| Employee | List/view tasks in accessible projects; update status only when assigned to self |

Enforced via `App\Policies\TaskPolicy` (when implemented). See [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md).

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
| Phase 5.2–5.6 — Task Management | ⏳ Pending |
| **Milestone 5** | In progress (5.1 done) |
| Remarks / Activity Logs | Planned |
| Vue Pinia auth | Planned |
| Deployment | Planned |
