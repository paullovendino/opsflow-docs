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
      "name": "Jane Doe",
      "email": "jane@example.com",
      "email_verified_at": "2026-08-02T00:00:00.000000Z",
      "created_at": "2026-08-02T00:00:00.000000Z",
      "updated_at": "2026-08-02T00:00:00.000000Z"
    }
  },
  "errors": null,
  "meta": null
}
```

> **Milestone 3 note (Planned):** After users ERD alignment, `UserResource` will expose structured profile fields (`first_name`, `last_name`, `role`, `department`, `job_title`, `status`, etc.) instead of the legacy `name` shape. Auth responses must follow the updated resource.

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
    "name": "Jane Doe",
    "email": "jane@example.com",
    "email_verified_at": "2026-08-02T00:00:00.000000Z",
    "created_at": "2026-08-02T00:00:00.000000Z",
    "updated_at": "2026-08-02T00:00:00.000000Z"
  },
  "errors": null,
  "meta": null
}
```

> **Milestone 3 note (Planned):** Same `UserResource` expansion as login (`data` holds the user object directly).

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

> **Planned — Milestone 3 (Organization & User Management)**  
> Implementation has not started. All endpoints in this section are **Planned**.

Domain reference: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)  
Milestone spec: [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)

### Authorization (coarse — Planned)

| Role | Capability |
|------|------------|
| Administrator | Full User Management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

Lookup reads (roles / departments / job titles): **all authenticated users** (`auth:sanctum`).

Advanced RBAC is out of scope for this milestone.

### Planned User resource shape

```json
{
  "id": 1,
  "first_name": "Jane",
  "middle_name": null,
  "last_name": "Doe",
  "email": "jane@example.com",
  "email_verified_at": null,
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
    "name": "engineering",
    "description": "Product and engineering"
  },
  "job_title": {
    "id": 2,
    "name": "project_manager",
    "description": "Delivers projects and coordinates teams"
  },
  "created_at": "2026-08-02T00:00:00.000000Z",
  "updated_at": "2026-08-02T00:00:00.000000Z"
}
```

Password is never returned. `department` / `job_title` may be `null`.

---

## Users

> Planned — Milestone 3

### GET /api/v1/users

**Status:** Planned

List users (directory).

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator, Project Manager (read-only)

**Query parameters (approved filters):**

| Parameter | Description |
|-----------|-------------|
| `search` | Match against name fields and/or email |
| `role_id` | Filter by role ID |
| `department_id` | Filter by department ID |
| `job_title_id` | Filter by job title ID |
| `status` | Filter by user status (`active`, `inactive`) |
| `page` / `per_page` | Pagination |

Do not use name-based FK filters (`role`, `department`, `job_title`). Standardize on IDs.

**Success:** `200` with user collection in `data` and pagination in `meta`.

---

### GET /api/v1/users/{id}

**Status:** Planned

Show a single user.

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator; Project Manager (read); Employee only when `{id}` is self

**Success:** `200` with user object in `data`.

---

### POST /api/v1/users

**Status:** Planned

Create a user.

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator only

**Request body (Planned):**

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
- Password hashed server-side

**Success:** `201` with created user in `data`.

---

### PUT /api/v1/users/{id}

**Status:** Planned

Update a user.

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator only

**Request body (Planned):** same fields as create; password optional on update.

**Success:** `200` with updated user in `data`.

---

### DELETE /api/v1/users/{id}

**Status:** Planned

Soft-delete a user.

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator only

**Success:** `200` (or `204` if team standardizes on no body — prefer envelope `200` for consistency).

---

### PATCH /api/v1/users/{id}/status

**Status:** Planned

Activate or deactivate a user.

**Authentication:** `auth:sanctum`  
**Authorization (Planned):** Administrator only

**Request body (Planned):**

```json
{
  "status": "inactive"
}
```

**Success:** `200` with updated user in `data`.

---

## Roles

> Planned — Milestone 3 (read-only)

### GET /api/v1/roles

**Status:** Planned

List seeded roles.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with roles collection in `data`.

---

### GET /api/v1/roles/{id}

**Status:** Planned

Show a role.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with role object in `data`.

---

## Departments

> Planned — Milestone 3 (read-only; seeded)

### GET /api/v1/departments

**Status:** Planned

List departments.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with departments collection in `data`.

---

### GET /api/v1/departments/{id}

**Status:** Planned

Show a department.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with department object in `data`.

---

## Job Titles

> Planned — Milestone 3 (read-only; seeded)

### GET /api/v1/job-titles

**Status:** Planned

List job titles.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with job titles collection in `data`.

---

### GET /api/v1/job-titles/{id}

**Status:** Planned

Show a job title.

**Authentication:** `auth:sanctum` (all authenticated users)

**Success:** `200` with job title object in `data`.

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
