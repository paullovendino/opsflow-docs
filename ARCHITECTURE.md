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
| Models | Persistence / relationships |
| Enums | Domain constants |
| Policies | Coarse authorization (Phase 3.6) |
| Exceptions + `ApiExceptionRenderer` | Consistent API errors |

Authentication example:

`AuthController` → `LoginRequest` → `AuthenticationService` → `User` / session → `UserResource`

User Management example (Phase 3.3 — implemented):

`UserController` → `StoreUserRequest` / `UpdateUserRequest` / `UpdateUserStatusRequest` → `UserService` → `User` + relations → `UserResource` (+ nested `RoleResource` / `DepartmentResource` / `JobTitleResource`)

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

**Phase 3.6 — Pending.** Coarse role checks only — not advanced RBAC:

| Role | User Management |
|------|-----------------|
| Administrator | Full user management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

Until Phase 3.6, User Management APIs require authentication only (`auth:sanctum`).

Lookup endpoints (Phase 3.4): all authenticated users.

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
| Registered | `user`, `role`, `department`, `job_title` |
| Later | `project`, `task`, `remark`, `activity_log` |

### Reference Data

- Roles: seeded; lookup APIs Phase 3.4
- Departments: seeded; soft deletes; lookup APIs Phase 3.4
- Job Titles: seeded; soft deletes; lookup APIs Phase 3.4

---

## Implemented vs Planned

| Area | Status |
|------|--------|
| API foundation | ✅ Implemented |
| Authentication (API) | ✅ Implemented |
| Organization Foundation (3.1) | ✅ Implemented |
| User Domain Foundation (3.2) | ✅ Implemented |
| User Management APIs (3.3) | ✅ Implemented |
| Lookup APIs (3.4) | Pending |
| Search / filters / pagination (3.5) | Pending |
| Coarse authorization (3.6) | Pending |
| Projects / Tasks / Remarks / Activity Logs | Planned |
| Vue Pinia auth | Planned |
| Deployment | Planned |
