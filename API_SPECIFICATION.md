# API Specification

All endpoints are versioned under `/api/v1`.

## Response Envelope

Every endpoint returns:

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

Pagination (when applicable): records in `data`, page info in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`).

---

## System

### GET /api/v1/health

Health check for the API process.

- Authentication: none
- Success: `200` with service status payload in `data`

---

## Authentication

Sanctum SPA cookie authentication.

Prerequisite for browser/SPA clients:

### GET /sanctum/csrf-cookie

Issues the CSRF cookie used by subsequent state-changing requests.

---

### POST /api/v1/auth/login

Authenticate a user and start a session.

**Description:** Validates credentials, regenerates the session, and returns the authenticated user under `data.user`.

**Authentication:** Guest only (`guest`). Already authenticated callers receive `403`.

**Middleware:** `guest`, `throttle:login`

**Request body:**

```json
{
  "email": "user@example.com",
  "password": "password"
}
```

**Validation rules:**

| Field | Rules |
|-------|-------|
| `email` | required, string, email |
| `password` | required, string |

**Success response (`200`):**

```json
{
  "success": true,
  "message": "Login successful.",
  "data": {
    "user": {
      "id": 1,
      "first_name": "Jane",
      "middle_name": null,
      "last_name": "Doe",
      "full_name": "Jane Doe",
      "email": "jane@example.com",
      "avatar": null,
      "status": "active",
      "last_login_at": "2026-08-02T00:00:00.000000Z",
      "role": {
        "id": 3,
        "name": "employee",
        "description": "Assigned work and updates"
      },
      "department": null,
      "job_title": null
    }
  },
  "errors": null,
  "meta": null
}
```

> **Phase 3.2:** `UserResource` exposes structured profile fields. Auth login/`/me` eager-load `role`, `department`, and `jobTitle`.

**Error responses:**

| HTTP | Condition | Example message |
|------|-----------|-----------------|
| `422` | Validation failure | `The given data was invalid.` |
| `401` | Invalid credentials | `Invalid credentials.` |
| `403` | Already authenticated | `Already authenticated.` |
| `403` | Account inactive (`status = inactive`) | `Account is inactive.` |
| `429` | Rate limit exceeded | `Too Many Attempts.` |

> **Milestone 3 (approved):** Inactive users must not be able to log in. After credentials would otherwise succeed, reject with HTTP `403` and a dedicated inactive-account message (not `401`).

**Example request (HTTP):**

```http
POST /api/v1/auth/login HTTP/1.1
Host: localhost:8000
Origin: http://localhost:5173
Accept: application/json
Content-Type: application/json
X-XSRF-TOKEN: <decoded-xsrf-token>

{
  "email": "jane@example.com",
  "password": "password"
}
```

---

### POST /api/v1/auth/logout

End the authenticated session.

**Description:** Logs out the `web` guard user, invalidates the session, and regenerates the CSRF token.

**Authentication:** Required (`auth:sanctum`)

**Request body:** none

**Success response (`200`):**

```json
{
  "success": true,
  "message": "Logout successful.",
  "data": null,
  "errors": null,
  "meta": null
}
```

**Error responses:**

| HTTP | Condition | Example message |
|------|-----------|-----------------|
| `401` | Missing/invalid session | `Unauthenticated.` |

**Example request (HTTP):**

```http
POST /api/v1/auth/logout HTTP/1.1
Host: localhost:8000
Origin: http://localhost:5173
Accept: application/json
X-XSRF-TOKEN: <decoded-xsrf-token>
Cookie: <session-cookie>
```

---

### GET /api/v1/auth/me

Return the currently authenticated user.

**Description:** Resolves the session user via Sanctum and returns a `UserResource`.

**Authentication:** Required (`auth:sanctum`)

**Request body:** none

**Success response (`200`):**

```json
{
  "success": true,
  "message": "Authenticated user retrieved successfully.",
  "data": {
    "id": 1,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com",
    "avatar": null,
    "status": "active",
    "last_login_at": "2026-08-02T00:00:00.000000Z",
    "role": {
      "id": 3,
      "name": "employee",
      "description": "Assigned work and updates"
    },
    "department": null,
    "job_title": null
  },
  "errors": null,
  "meta": null
}
```

> **Phase 3.2:** Same expanded `UserResource` as login (`data` holds the user object directly).

**Error responses:**

| HTTP | Condition | Example message |
|------|-----------|-----------------|
| `401` | Missing/invalid session | `Unauthenticated.` |

**Example request (HTTP):**

```http
GET /api/v1/auth/me HTTP/1.1
Host: localhost:8000
Origin: http://localhost:5173
Accept: application/json
Cookie: <session-cookie>
```

---

## Organization & User Management

> **Phases 3.1–3.6 implemented (Milestone 3 complete).**

Domain reference: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)  
Milestone spec: [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)

### Authorization (coarse — Phase 3.6 Implemented)

| Role | Capability |
|------|------------|
| Administrator | Full User Management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

User Management routes require `auth:sanctum` and `UserPolicy` checks. Unauthorized → HTTP `403` with the standard API envelope.

Lookup reads (roles / departments / job titles): **all authenticated users** — Phase 3.4 (not role-gated).

Advanced RBAC / permission matrices remain out of scope.

### User resource shape (Phase 3.3)

```json
{
  "id": 1,
  "first_name": "Jane",
  "middle_name": null,
  "last_name": "Doe",
  "full_name": "Jane Doe",
  "email": "jane@example.com",
  "avatar": null,
  "status": "active",
  "last_login_at": null,
  "role": {
    "id": 1,
    "name": "administrator",
    "description": "Full system access"
  },
  "department": {
    "id": 1,
    "name": "Engineering",
    "code": "ENG",
    "description": "Product and engineering"
  },
  "job_title": {
    "id": 2,
    "name": "Project Manager",
    "code": "PM",
    "description": "Delivers projects and coordinates teams"
  }
}
```

Password is never returned. `department` / `job_title` may be `null`. Nested objects use `RoleResource` / `DepartmentResource` / `JobTitleResource` when relations are loaded.

---

## Users

> **Phase 3.3 — Implemented (CRUD APIs)** · **Phase 3.5 — Implemented (list query)** · **Phase 3.6 — Implemented (authorization)**  
> Lookup APIs are Phase 3.4 (`/api/v1/lookups/*`).

### GET /api/v1/users

**Status:** Implemented (Phases 3.3 + 3.5 + 3.6)

List users (directory) with search, filtering, sorting, and pagination.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator, Project Manager

**Query parameters:**

| Param | Rules / notes |
|-------|----------------|
| `search` | Optional string; matches `first_name`, `middle_name`, `last_name`, `email` (case-insensitive) |
| `role_id` | Optional integer; must exist in `roles` |
| `department_id` | Optional integer; must exist in `departments` |
| `job_title_id` | Optional integer; must exist in `job_titles` |
| `status` | Optional; `active` \| `inactive` |
| `sort` | Optional; one of `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status` (default `created_at`) |
| `direction` | Optional; `asc` \| `desc` (default `desc`) |
| `page` | Optional integer ≥ 1 (default `1`) |
| `per_page` | Optional integer 1–100 (default `15`); values above 100 are clamped to 100 |

Filters are composable (e.g. `status` + `department_id` + `search` together).

**Success:** `200` with user collection in `data` and pagination in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`). Invalid query params → `422`. Unauthorized → `403`.

---

### GET /api/v1/users/{id}

**Status:** Implemented (Phase 3.3 + 3.6)

Show a single user.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator, Project Manager; Employee may view **own** profile only

**Success:** `200` with user object in `data`. Unauthorized → `403`.

---

### POST /api/v1/users

**Status:** Implemented (Phase 3.3 + 3.6)

Create a user.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator only

**Request body:**

```json
{
  "first_name": "Jane",
  "middle_name": null,
  "last_name": "Doe",
  "email": "jane@example.com",
  "password": "secret-password",
  "role_id": 3,
  "department_id": 1,
  "job_title_id": 2,
  "avatar": null,
  "status": "active"
}
```

**Notes:**

- `role_id` required
- `department_id` / `job_title_id` nullable
- Password hashed server-side via `Hash::make`
- `status` uses `UserStatus` values: `active`, `inactive`

**Success:** `201` with created user in `data`.

---

### PUT /api/v1/users/{id}

**Status:** Implemented (Phase 3.3 + 3.6)

Update a user.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator only

**Request body:** same fields as create; password optional on update (unchanged when omitted).

**Success:** `200` with updated user in `data`. Unauthorized → `403`.

---

### DELETE /api/v1/users/{id}

**Status:** Implemented (Phase 3.3 + 3.6)

Soft-delete a user.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator only

**Success:** `200` with standard envelope. Unauthorized → `403`.

---

### PATCH /api/v1/users/{id}/status

**Status:** Implemented (Phase 3.3 + 3.6)

Activate or deactivate a user. Updates `status` only.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator only

**Request body:**

```json
{
  "status": "inactive"
}
```

Accepted `status` values (`UserStatus`): `active`, `inactive`.

**Success:** `200` with updated user in `data`.

---

## Lookups

> **Phase 3.4 — Implemented** (read-only reference collections)

Shared prefix: `/api/v1/lookups`. Collection endpoints only (no show/`{id}` routes). Soft-deleted departments and job titles are excluded. Results are sorted alphabetically by `name`.

### GET /api/v1/lookups/roles

**Status:** Implemented (Phase 3.4)

List seeded roles.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with roles collection in `data` (`RoleResource`).

---

### GET /api/v1/lookups/departments

**Status:** Implemented (Phase 3.4)

List departments (non-soft-deleted).

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with departments collection in `data` (`DepartmentResource`).

---

### GET /api/v1/lookups/job-titles

**Status:** Implemented (Phase 3.4)

List job titles (non-soft-deleted).

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with job titles collection in `data` (`JobTitleResource`).

---

## Projects

> Planned

GET /api/v1/projects

GET /api/v1/projects/{id}

POST /api/v1/projects

PUT /api/v1/projects/{id}

DELETE /api/v1/projects/{id}

---

## Tasks

> Planned

GET /api/v1/tasks

GET /api/v1/tasks/{id}

POST /api/v1/tasks

PUT /api/v1/tasks/{id}

DELETE /api/v1/tasks/{id}

---

## Remarks

> Planned — future

Remark endpoints will be defined when the Remarks milestone begins.

---

## Dashboard

> Planned

GET /api/v1/dashboard

---

## Activity Logs

> Planned

GET /api/v1/activity-logs
