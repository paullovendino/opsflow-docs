# Roadmap

OpsFlow development roadmap by phase.

**Status legend:** ✅ Completed / Implemented · Pending

---

## Completed / implemented milestones

| Phase | Milestone | Status |
|-------|-----------|--------|
| Phase 1 | Backend Foundation (API) | ✅ Completed |
| Phase 2 | Authentication (API) | ✅ Completed |
| Phase 3.1 | Organization Foundation | ✅ Implemented |
| Phase 3.2 | User Domain Foundation | ✅ Implemented |
| Phase 3.3 | User Management APIs | ✅ Implemented |

---

## Phase 1 — Project Setup / Backend Foundation

**Status: ✅ Completed (backend)**

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

**Status: ✅ Completed (API)** · Pinia/Vue pending

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

**Status: Phases 3.1–3.3 implemented · Phases 3.4–3.6 remaining**

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

### Phase 3.1 — Organization Foundation

**Status: ✅ Implemented**

- [x] `departments` table (`name`, `code`, …) + approved seeder (soft deletes)
- [x] `job_titles` table (`name`, `code`, …) + approved seeder (soft deletes)
- [x] Models + morph aliases
- [x] PHPUnit Feature tests (`OrganizationFoundationTest`)

### Phase 3.2 — User Domain Foundation

**Status: ✅ Implemented**

- [x] Align `users` schema with ERD (`role_id`, names, `department_id`, `job_title_id`, `status`, soft deletes, keep `email_verified_at`, …)
- [x] `User::department()` / `User::jobTitle()` / `User::role()` with FK RESTRICT
- [x] `UserStatus` enum + `full_name` accessor
- [x] Expanded `UserResource` + auth compatibility (inactive login `403`, `last_login_at`)
- [x] PHPUnit Feature tests (`UserDomainFoundationTest`) + existing auth suite green

### Phase 3.3 — User Management APIs

**Status: ✅ Implemented**

- [x] User CRUD + `PATCH /users/{id}/status` (`UserController` / `UserService`)
- [x] `StoreUserRequest` / `UpdateUserRequest` / `UpdateUserStatusRequest`
- [x] `RoleResource` / `DepartmentResource` / `JobTitleResource` + updated `UserResource`
- [x] Soft delete; password hashing via `Hash::make`; status-only patch
- [x] PHPUnit Feature tests (`UserManagementApiTest`) + prior suites green
- [x] Auth remains `auth:sanctum` (role policies deferred to 3.6)

### Phase 3.4 — Lookup APIs

**Status: Pending**

- [ ] `GET /api/v1/roles`, `GET /api/v1/roles/{id}`
- [ ] `GET /api/v1/departments`, `GET /api/v1/departments/{id}`
- [ ] `GET /api/v1/job-titles`, `GET /api/v1/job-titles/{id}`
- [ ] Accessible to all authenticated users

### Phase 3.5 — Search, Filtering & Pagination

**Status: Pending**

- [ ] User list `search`
- [ ] Filters: `role_id`, `department_id`, `job_title_id`, `status`
- [ ] Pagination with standard `meta` fields

### Phase 3.6 — Authorization (RBAC)

**Status: Pending**

- [ ] Coarse authorization (Administrator / Project Manager / Employee)
- [ ] Policies/gates for User Management
- [ ] Not advanced permission matrices

### Explicitly out of scope (Milestone 3)

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
