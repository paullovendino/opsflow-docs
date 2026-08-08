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

Frontend consumption (Milestone 8 — ✅; Milestone 9 — ✅ Phases 9.1–9.5): [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md).

SPA must:

- Call `GET /sanctum/csrf-cookie` immediately before login
- Send Axios requests with `withCredentials: true`, `withXSRFToken: true`, and `X-XSRF-TOKEN`
- Treat login success user as `data.user`; `/me` user as `data` (normalize in the SPA)
- Show login `401` / inactive `403` / `422` inline on the login form (not via a global `/403` redirect)

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
> Vue UI: Phase 9.2 ✅ — list + modal Create/Edit/View; show/profile pages; `useLookups` SPA-session cache — [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

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

> **Milestone 4 — ✅ Phases 4.1–4.5 implemented (complete)**  
> Spec: [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md)  
> ADR: [decisions/Project-Management.md](decisions/Project-Management.md)  
> Vue UI: Phase 9.3 ✅ — list + modal Create/Edit/View; Show workspace page (members + `ProjectTasksPanel`) — [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Authorization (coarse — Phase 4.5)

| Role | Capability |
|------|------------|
| Administrator | Full Project Management (all projects) |
| Project Manager | Full Project Management (all projects) |
| Employee | List/view projects they own or are members of |

Project routes require `auth:sanctum` and `ProjectPolicy` checks via `$this->authorize()`. Unauthorized → `403`.

### Project resource shape

```json
{
  "id": 1,
  "name": "OpsFlow Launch",
  "description": "Initial product launch workstream",
  "status": "planning",
  "start_date": "2026-08-01",
  "due_date": "2026-12-31",
  "owner": {
    "id": 1,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com"
  },
  "created_at": "2026-08-02T00:00:00.000000Z",
  "updated_at": "2026-08-02T00:00:00.000000Z"
}
```

`status` values (`ProjectStatus`): `planning`, `active`, `on_hold`, `completed`, `archived`.  
`owner` is nested when the relation is loaded (`whenLoaded()`). Members are exposed via dedicated member endpoints (Phase 4.3).

---

### GET /api/v1/projects

**Status:** Implemented (Phase 4.4 + 4.5)

List projects with search, filtering, sorting, and pagination.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator, Project Manager (all projects); Employee (owned or member only)

**Query parameters:**

| Param | Rules / notes |
|-------|----------------|
| `search` | Optional string; matches `name`, `description` (case-insensitive) |
| `status` | Optional; one of `planning`, `active`, `on_hold`, `completed`, `archived` |
| `created_by` | Optional integer; must exist in `users` |
| `sort` | Optional; one of `name`, `status`, `start_date`, `due_date`, `created_at` (default `created_at`) |
| `direction` | Optional; `asc` \| `desc` (default `desc`) |
| `page` | Optional integer ≥ 1 (default `1`) |
| `per_page` | Optional integer 1–100 (default `15`); values above 100 are clamped to 100 |

Filters are composable (e.g. `status` + `created_by` + `search` together).

**Success:** `200` with paginated `ProjectResource` collection in `data` and pagination in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`). Invalid query params → `422` (standard API envelope). Unauthorized → `403`.

---

### GET /api/v1/projects/{id}

**Status:** Implemented (Phase 4.2 + 4.5)

Show a single project.

**Authentication:** `auth:sanctum`

**Success:** `200` with project object in `data`.

---

### POST /api/v1/projects

**Status:** Implemented (Phase 4.2 + 4.5)

Create a project.

**Authentication:** `auth:sanctum`

**Request body:**

```json
{
  "name": "OpsFlow Launch",
  "description": "Initial product launch workstream",
  "start_date": "2026-08-01",
  "due_date": "2026-12-31"
}
```

**Notes:**

- `name` required
- `description`, `start_date`, `due_date` optional
- `status` is **not** accepted on create — always `planning`
- `created_by` set server-side from the authenticated user (client-supplied values ignored)

**Success:** `201` with created project in `data`.

---

### PUT /api/v1/projects/{id}

**Status:** Implemented (Phase 4.2 + 4.5)

Update a project.

**Authentication:** `auth:sanctum`

**Request body:** `name`, `description`, `start_date`, `due_date` only. Ownership (`created_by`) and `status` are not updatable here.

**Success:** `200` with updated project in `data`.

---

### DELETE /api/v1/projects/{id}

**Status:** Implemented (Phase 4.2 + 4.5)

Soft-delete a project.

**Authentication:** `auth:sanctum`

**Success:** `200` with standard envelope.

---

### PATCH /api/v1/projects/{id}/status

**Status:** Implemented (Phase 4.2 + 4.5)

Update project status only.

**Authentication:** `auth:sanctum`

**Request body:**

```json
{
  "status": "active"
}
```

Accepted `status` values (`ProjectStatus`): `planning`, `active`, `on_hold`, `completed`, `archived`.

**Success:** `200` with updated project in `data`.

---

### GET /api/v1/projects/{id}/members

**Status:** Implemented (Phase 4.3 + 4.5)

List members of a project.

**Authentication:** `auth:sanctum`

**Success:** `200` with member collection in `data` (`ProjectMemberResource`: user summary + `joined_at`).

---

### POST /api/v1/projects/{id}/members

**Status:** Implemented (Phase 4.3 + 4.5)

Add a member to a project.

**Authentication:** `auth:sanctum`

**Request body:**

```json
{
  "user_id": 3
}
```

**Notes:**

- `user_id` required; must exist, be `active`, and not soft-deleted (`422` otherwise)
- Unique (`project_id`, `user_id`) — duplicate → HTTP `409`
- `joined_at` set server-side to now (client value ignored)
- No invitation workflow

**Success:** `201` with member payload in `data`.

---

### DELETE /api/v1/projects/{id}/members/{user}

**Status:** Implemented (Phase 4.3 + 4.5)

Remove a member from a project (hard-delete pivot row).

**Authentication:** `auth:sanctum`

**Success:** `200` with standard envelope. Does not change `created_by`. Unknown membership → `404`.

---

## Tasks

> **Milestone 5 — ✅ Complete** (Phases 5.1–5.6)  
> Spec: [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md)  
> ADR: [decisions/Task-Management.md](decisions/Task-Management.md)  
> Vue UI: Phase 9.4 ✅ — table list + modal Create/Edit/View; `/tasks/:id` deep link; assignment/status; not Kanban — [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Authorization (coarse — Phase 5.5)

| Role | Capability |
|------|------------|
| Administrator | Full Task Management (all tasks) |
| Project Manager | Full Task Management (all tasks) |
| Employee | List/view tasks in owned or member projects; update status only when assigned to self |

Task routes require `auth:sanctum`. `TaskPolicy` checks are enforced.

### Task resource shape

```json
{
  "id": 1,
  "title": "Draft API contract",
  "description": "Document task endpoints",
  "status": "todo",
  "priority": "medium",
  "due_date": "2026-08-15",
  "project": {
    "id": 1,
    "name": "OpsFlow Launch"
  },
  "assignee": {
    "id": 3,
    "first_name": "Sam",
    "middle_name": null,
    "last_name": "Lee",
    "full_name": "Sam Lee",
    "email": "sam@example.com"
  },
  "creator": {
    "id": 1,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com"
  },
  "created_at": "2026-08-03T00:00:00.000000Z",
  "updated_at": "2026-08-03T00:00:00.000000Z"
}
```

`status` values (`TaskStatus`): `todo`, `in_progress`, `in_review`, `blocked`, `completed`, `cancelled`.  
`priority` values (`TaskPriority`): `low`, `medium`, `high`, `urgent`.  
`assignee` may be `null`. Nested `project` / `assignee` / `creator` appear when relations are loaded (`whenLoaded()`).

---

### GET /api/v1/tasks

**Status:** ✅ Phase 5.4 implemented · authz Phase 5.5

List tasks with search, filtering, sorting, and pagination.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator, Project Manager (all tasks); Employee (tasks in accessible projects)

**Query parameters:**

| Param | Rules / notes |
|-------|----------------|
| `search` | Optional string; matches `title`, `description` (case-insensitive) |
| `status` | Optional; one of `todo`, `in_progress`, `in_review`, `blocked`, `completed`, `cancelled` |
| `priority` | Optional; one of `low`, `medium`, `high`, `urgent` |
| `project_id` | Optional integer; must exist in `projects` |
| `assigned_to` | Optional integer; must exist in `users` |
| `created_by` | Optional integer; must exist in `users` |
| `sort` | Optional; one of `title`, `status`, `priority`, `due_date`, `created_at` (default `created_at`) |
| `direction` | Optional; `asc` \| `desc` (default `desc`) |
| `page` | Optional integer ≥ 1 (default `1`) |
| `per_page` | Optional integer 1–100 (default `15`); values above 100 are clamped to 100 |

Filters are composable (e.g. `status` + `project_id` + `search` together).

**Success:** `200` with paginated `TaskResource` collection in `data` and pagination in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`). Invalid query params → `422`. Unauthorized → `403`.

---

### GET /api/v1/tasks/{id}

**Status:** ✅ Phase 5.2 implemented · authz Phase 5.5

Show a single task.

**Authentication:** `auth:sanctum`

**Success:** `200` with task object in `data`. Unauthorized → `403`.

---

### POST /api/v1/tasks

**Status:** ✅ Phase 5.2 implemented · authz Phase 5.5

Create a task.

**Authentication:** `auth:sanctum`

**Request body:**

```json
{
  "project_id": 1,
  "title": "Draft API contract",
  "description": "Document task endpoints",
  "priority": "medium",
  "due_date": "2026-08-15",
  "assigned_to": 3
}
```

**Notes:**

- `project_id`, `title` required
- `description`, `priority`, `due_date`, `assigned_to` optional
- Default `priority` when omitted: `medium`
- `status` is **not** accepted on create — always `todo`
- `created_by` set server-side from the authenticated user (client-supplied values ignored)
- When `assigned_to` is present: must be active, not soft-deleted, and project owner or member (`422` otherwise)

**Success:** `201` with created task in `data`.

---

### PUT /api/v1/tasks/{id}

**Status:** ✅ Phase 5.2 implemented · authz Phase 5.5

Update a task.

**Authentication:** `auth:sanctum`

**Request body:** `title`, `description`, `priority`, `due_date` only. `status`, `assigned_to`, `project_id`, and `created_by` are not updatable here.

**Success:** `200` with updated task in `data`.

---

### DELETE /api/v1/tasks/{id}

**Status:** ✅ Phase 5.2 implemented · authz Phase 5.5

Soft-delete a task.

**Authentication:** `auth:sanctum`

**Success:** `200` with standard envelope.

---

### PATCH /api/v1/tasks/{id}/assignment

**Status:** ✅ Phase 5.3 implemented · authz Phase 5.5

Set or clear the single task assignee.

**Authentication:** `auth:sanctum`

**Request body:**

```json
{
  "assigned_to": 3
}
```

Clear assignee:

```json
{
  "assigned_to": null
}
```

**Notes:**

- `assigned_to` key is required (`present`); value may be `null` to unassign
- When non-null: must be active, not soft-deleted, and project owner or member (`422` otherwise)
- Does not change `status`, `created_by`, or `project_id`

**Success:** `200` with updated task in `data`.

---

### PATCH /api/v1/tasks/{id}/status

**Status:** ✅ Phase 5.6 implemented

Update task status only.

**Authentication:** `auth:sanctum`  
**Authorization:** Administrator, Project Manager (all tasks); Employee only when assigned to self (and project accessible)

**Request body:**

```json
{
  "status": "in_progress"
}
```

Accepted `status` values (`TaskStatus`): `todo`, `in_progress`, `in_review`, `blocked`, `completed`, `cancelled`.  
No restricted transition graph in Milestone 5.  
Does not change title, priority, assignee, dates, or project.

**Success:** `200` with updated task in `data`. Invalid status → `422`. Unauthorized → `403`.

---

## Remarks

> Planned — future

Remark endpoints will be defined when the Remarks milestone begins.

---

## Dashboard

> Milestone 6 — ✅ Implemented (API) · Phase 9.1 — ✅ Dashboard UI (`opsflow-web`)  
> Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · ADR: [decisions/Dashboard.md](decisions/Dashboard.md) · UI: [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Show dashboard

```http
GET /api/v1/dashboard
Authorization: Sanctum session (auth:sanctum)
```

**Query parameters**

| Param | Rules | Default |
|-------|--------|---------|
| `recent_limit` | optional integer; clamped to 1–25 | `10` |

**Authorization:** All authenticated roles may view. Aggregate data is scoped: Admin/PM org-wide; Employee owned-or-member projects (and tasks therein).

**Success `data` shape**

```json
{
  "projects": {
    "total": 0,
    "by_status": {
      "planning": 0,
      "active": 0,
      "on_hold": 0,
      "completed": 0,
      "archived": 0
    }
  },
  "tasks": {
    "total": 0,
    "by_status": {
      "todo": 0,
      "in_progress": 0,
      "in_review": 0,
      "blocked": 0,
      "completed": 0,
      "cancelled": 0
    },
    "by_priority": {
      "low": 0,
      "medium": 0,
      "high": 0,
      "urgent": 0
    },
    "overdue": 0,
    "assigned_to_me": 0
  },
  "recent": []
}
```

**Recent item discriminators**

| `type` | Fields |
|--------|--------|
| `task` | `id`, `title`, `status`, `project_id`, `updated_at` |
| `project` | `id`, `name`, `status`, `updated_at` |

**Notes**

- Read-only; no create/update/delete dashboard routes
- Soft-deleted projects/tasks excluded
- Overdue: `due_date` &lt; today and status not `completed` / `cancelled`
- `by_status` / `by_priority` keys always present (zero-filled)
- Recent feed is derived work items (not Activity Logs)
- Guest → `401`

**Classes:** `DashboardController`, `DashboardService`, `ShowDashboardRequest`, `DashboardResource`, `DashboardPolicy` (`Gate::define('viewDashboard')`)

---

## Reports

> Milestone 7 — ✅ Implemented  
> Spec: [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) · ADR: [decisions/Reports.md](decisions/Reports.md)  
> Vue UI: Phase 9.5 ✅ — project/employee list+detail pages; date filters; Tailwind bars; no chart library; no exports — [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### List project reports

```http
GET /api/v1/reports/projects
Authorization: Sanctum session (auth:sanctum)
```

**Query parameters:** `search`, `status` (`ProjectStatus`), `from_date`, `to_date`, `sort` (`name`|`status`|`created_at`), `direction`, `page`, `per_page` (default 15, max 100 clamp).

**Authorization:** Admin/PM all projects; Employee owned-or-member only.

**Success:** paginated `data` items shaped as Project Report (see below); pagination `meta`.

### Show project report

```http
GET /api/v1/reports/projects/{project}
Authorization: Sanctum session (auth:sanctum)
```

**Query parameters:** `from_date`, `to_date` (optional inclusive `Y-m-d`; filters tasks by `created_at` date).

**Authorization:** Admin/PM any project; Employee accessible projects only (`403` otherwise).

### Project report `data` shape

```json
{
  "project": {
    "id": 3,
    "name": "OpsFlow Launch",
    "status": "active",
    "start_date": null,
    "due_date": "2026-09-01",
    "created_at": "2026-07-01T10:00:00+00:00"
  },
  "tasks": {
    "total": 0,
    "by_status": {
      "todo": 0,
      "in_progress": 0,
      "in_review": 0,
      "blocked": 0,
      "completed": 0,
      "cancelled": 0
    },
    "by_priority": {
      "low": 0,
      "medium": 0,
      "high": 0,
      "urgent": 0
    },
    "overdue": 0,
    "unassigned": 0
  },
  "members_count": 0
}
```

### List employee reports

```http
GET /api/v1/reports/employees
Authorization: Sanctum session (auth:sanctum)
```

**Query parameters:** `search`, `role_id`, `department_id`, `status` (`UserStatus`), `from_date`, `to_date`, `sort`, `direction`, `page`, `per_page`.

**Authorization:** Administrator and Project Manager only (`403` for Employee).

### Show employee report

```http
GET /api/v1/reports/employees/{user}
Authorization: Sanctum session (auth:sanctum)
```

**Query parameters:** `from_date`, `to_date`.

**Authorization:** Admin/PM any user; Employee **self only**.

### Employee report `data` shape

```json
{
  "user": {
    "id": 12,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com",
    "status": "active"
  },
  "tasks": {
    "total": 0,
    "by_status": {
      "todo": 0,
      "in_progress": 0,
      "in_review": 0,
      "blocked": 0,
      "completed": 0,
      "cancelled": 0
    },
    "by_priority": {
      "low": 0,
      "medium": 0,
      "high": 0,
      "urgent": 0
    },
    "overdue": 0,
    "by_project": [
      {
        "project_id": 3,
        "name": "OpsFlow Launch",
        "total": 10
      }
    ]
  }
}
```

**Notes**

- Read-only; no export/download routes in Milestone 7
- Soft-deleted projects/tasks/users excluded
- Date range filters tasks by `created_at` date (inclusive)
- Overdue reuses Dashboard definition
- List employee items omit `by_project`; detail includes it
- Guest → `401`

**Classes:** `ReportController`, `ReportService`, Form Requests under `Reports/`, `ProjectReportResource`, `EmployeeReportResource`, `ReportPolicy` (Gate abilities)

---

## Activity Logs

> Planned — future (not Milestone 6 or 7)

`GET /api/v1/activity-logs` will be defined when the Activity Logs milestone begins. Milestone 6 dashboard “recent” and Milestone 7 reports do **not** use this table.
