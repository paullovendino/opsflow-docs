# Architecture

## Overview

OpsFlow uses a separated frontend/backend architecture:

- `opsflow-api` — Laravel 13 REST API + PostgreSQL
- `opsflow-web` — Vue 3 SPA (TypeScript, Pinia, Axios)
- `opsflow-docs` — Product and engineering documentation

The API is the system of record. The SPA consumes `/api/v1` with Sanctum SPA cookie authentication.

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
| Exceptions + `ApiExceptionRenderer` | Consistent API errors |

Authentication example:

`AuthController` → `LoginRequest` → `AuthenticationService` → `User` / session → `UserResource`

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

## Cross-Cutting Concerns

### CORS

- Env-driven `CORS_ALLOWED_ORIGINS`
- Credentials enabled for SPA cookie auth
- Paths include `api/*` and `sanctum/csrf-cookie`

### Exception Handling

API routes (`api/*`) render through `App\Exceptions\ApiExceptionRenderer` using the standard envelope.

### Morph Map

`Relation::enforceMorphMap` registers aliases only for existing models (`user`, `role`).

### Roles Foundation

Roles are seeded for future RBAC. Authorization is not enforced yet.

---

## Implemented vs Planned

| Area | Status |
|------|--------|
| API foundation | Implemented |
| Authentication (API) | Implemented |
| User / Role management | Planned (Phase 3) |
| Projects / Tasks | Planned |
| Vue Pinia auth | Planned |
| Deployment | Planned |
