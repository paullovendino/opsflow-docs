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

GET /api/v1/health

---

## Authentication

Sanctum SPA cookie authentication.

GET /sanctum/csrf-cookie

POST /api/v1/login

POST /api/v1/logout

GET /api/v1/user

---

## Users

GET /api/v1/users

GET /api/v1/users/{id}

POST /api/v1/users

PUT /api/v1/users/{id}

DELETE /api/v1/users/{id}

---

## Projects

GET /api/v1/projects

GET /api/v1/projects/{id}

POST /api/v1/projects

PUT /api/v1/projects/{id}

DELETE /api/v1/projects/{id}

---

## Tasks

GET /api/v1/tasks

GET /api/v1/tasks/{id}

POST /api/v1/tasks

PUT /api/v1/tasks/{id}

DELETE /api/v1/tasks/{id}

---

## Dashboard

GET /api/v1/dashboard

---

## Activity Logs

GET /api/v1/activity-logs
