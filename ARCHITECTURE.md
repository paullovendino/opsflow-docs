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
| Policies | Coarse authorization (Milestone 3+) |
| Exceptions + `ApiExceptionRenderer` | Consistent API errors |

Authentication example:

`AuthController` → `LoginRequest` → `AuthenticationService` → `User` / session → `UserResource`

User Management example (Planned — Milestone 3):

`UserController` → Form Request → User service → `User` + relations → `UserResource`

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
- `auth:sanctum` for protected API routes
- Named `login` RateLimiter
- `guest` middleware on login

Details: [AUTHENTICATION.md](AUTHENTICATION.md)

---

## Authorization Architecture (Milestone 3 — Planned)

Coarse role checks only — not advanced RBAC:

| Role | User Management |
|------|-----------------|
| Administrator | Full user management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

Lookup endpoints (roles / departments / job titles): **all authenticated users**.

Inactive accounts cannot log in (`403` dedicated message).

Permission management UIs, ability matrices, and multi-role users are deferred.

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
| Registered today | `user`, `role` |
| Milestone 3 (when models exist) | `department`, `job_title` |
| Later | `project`, `task`, `remark`, `activity_log` |

### Reference Data

- Roles: seeded; read-only in Milestone 3
- Departments: seeded; read-only in Milestone 3
- Job Titles: seeded; read-only in Milestone 3

---

## Implemented vs Planned

| Area | Status |
|------|--------|
| API foundation | Implemented |
| Authentication (API) | Implemented |
| Organization & User Management (API) | Planned (Milestone 3 — design approved) |
| Projects / Tasks / Remarks / Activity Logs | Planned |
| Vue Pinia auth | Planned |
| Deployment | Planned |
