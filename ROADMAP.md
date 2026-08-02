# Roadmap

OpsFlow development roadmap by phase.

**Status legend:** Completed · In progress · Pending · Designed (docs only)

---

## Completed milestones

| Phase | Milestone | Status |
|-------|-----------|--------|
| Phase 1 | Backend Foundation (API) | Completed |
| Phase 2 | Authentication (API) | Completed |

---

## Phase 1 — Project Setup / Backend Foundation

**Status: Completed (backend)**

- [x] Laravel 13 API (`opsflow-api`)
- [ ] Vue 3 frontend repository
- [x] PostgreSQL configuration
- [x] Laravel Sanctum installed (SPA cookie mode)
- [x] API versioning (`/api/v1`)
- [x] CORS configuration
- [x] Base API controller and response envelope
- [x] Global API exception handling
- [x] Morph map foundation
- [x] Roles table, model, and seeder
- [x] Health check endpoint
- [x] Agreed folder structure
- [x] Documentation
- [ ] GitHub repositories finalized

---

## Phase 2 — Authentication

**Status: Completed (API)** · Pinia/Vue pending

- [x] Sanctum SPA cookie login/logout
- [x] CSRF cookie flow (configured; frontend consumes `/sanctum/csrf-cookie`)
- [x] Protected API routes (`auth:sanctum`)
- [x] Current authenticated user endpoint (`GET /api/v1/auth/me`)
- [x] Login rate limiting (`throttle:login`)
- [x] Guest-only login
- [x] Credential allowlist for `Auth::attempt()`
- [x] Auth feature tests
- [x] Authentication documentation
- [ ] Pinia authentication on Vue

---

## Phase 3 — Organization & User Management

**Status: Design approved — awaiting implementation approval**

Specification:

- [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
- [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md)

### Organizational model

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

### Planned work

- [ ] Align `users` schema with ERD (`role_id`, names, `department_id`, `job_title_id`, `status`, soft deletes, keep `email_verified_at`, …)
- [ ] `departments` table + approved seeder (read-only API; soft deletes)
- [ ] `job_titles` table + approved seeder (read-only API; soft deletes)
- [ ] Roles remain seeded and read-only (`GET /roles`, `GET /roles/{id}`) — all authenticated users
- [ ] Department / Job Title lookups — all authenticated users
- [ ] User CRUD + `PATCH /users/{id}/status`
- [ ] User list filters: `search`, `role_id`, `department_id`, `job_title_id`, `status` + pagination
- [ ] FK RESTRICT for role / department / job title references
- [ ] Inactive login → dedicated `403`
- [ ] Coarse authorization (Administrator / Project Manager / Employee)
- [ ] Update auth `UserResource` for expanded profile
- [ ] Feature tests + documentation status flip Planned → Implemented

### Explicitly out of scope

Department/Job Title CRUD, multi-role / multi-department users, teams, branches, organization settings, invitation emails, force password change, permission management, advanced RBAC.

---

## Phase 4 — Project Management

**Status: Pending**

- CRUD Projects
- Project Dashboard

---

## Phase 5 — Task Management

**Status: Pending**

- CRUD Tasks
- Task Assignment
- Task Status
- Priority

---

## Phase 6 — Dashboard

**Status: Pending**

- Charts
- Statistics
- Recent Activities

---

## Phase 7 — Reports

**Status: Pending**

- Project Reports
- Employee Reports

---

## Phase 8 — Testing

**Status: Pending**

- API Testing
- Frontend Testing
- Bug Fixes

---

## Phase 9 — Deployment

**Status: Pending**

- Backend Deployment
- Frontend Deployment
- Production Environment

---

## Future Versions

### v1.1

- Notifications
- Email Alerts
- File Attachments
- Remarks (domain entity)

### v1.2

- Kanban Board
- Calendar View
- Team Chat

### v2.0

- Time Tracking
- Gantt Chart
- Mobile Application
- Analytics
- Multi-organization / organization settings
