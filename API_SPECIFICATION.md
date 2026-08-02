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

**Error responses:**

| HTTP | Condition | Example message |
|------|-----------|-----------------|
| `422` | Validation failure | `The given data was invalid.` |
| `401` | Invalid credentials | `Invalid credentials.` |
| `403` | Already authenticated | `Already authenticated.` |
| `429` | Rate limit exceeded | `Too Many Attempts.` |

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

## Users

> Planned — Phase 3

GET /api/v1/users

GET /api/v1/users/{id}

POST /api/v1/users

PUT /api/v1/users/{id}

DELETE /api/v1/users/{id}

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

## Dashboard

> Planned

GET /api/v1/dashboard

---

## Activity Logs

> Planned

GET /api/v1/activity-logs
