# Roadmap

OpsFlow development roadmap by phase.

**Status legend:** Completed · In progress · Pending

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

## Phase 3 — User Management

**Status: Pending**

- Align users schema with ERD
- CRUD Users
- Role assignment
- Activate/Deactivate users

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

### v1.2

- Kanban Board
- Calendar View
- Team Chat

### v2.0

- Time Tracking
- Gantt Chart
- Mobile Application
- Analytics
