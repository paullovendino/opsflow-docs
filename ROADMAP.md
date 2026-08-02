# Roadmap

OpsFlow development roadmap by phase.

**Status legend:** ✅ Completed / Implemented · Pending

---

## Completed / implemented milestones

| Phase | Milestone | Status |
|-------|-----------|--------|
| Phase 1 | Backend Foundation (API) | ✅ Completed |
| Phase 2 | Authentication (API) | ✅ Completed |
| **Milestone 3** | **Organization & User Management** | ✅ **Complete** |
| Phase 3.1 | Organization Foundation | ✅ Implemented |
| Phase 3.2 | User Domain Foundation | ✅ Implemented |
| Phase 3.3 | User Management APIs | ✅ Implemented |
| Phase 3.4 | Lookup APIs | ✅ Implemented |
| Phase 3.5 | Search, Filtering & Pagination | ✅ Implemented |
| Phase 3.6 | Authorization (RBAC) | ✅ Implemented |
| **Milestone 4** | **Project Management** | ✅ **Complete** |
| Phase 4.1 | Project Domain Foundation | ✅ Implemented |
| Phase 4.2 | Project CRUD | ✅ Implemented |
| Phase 4.3 | Project Members | ✅ Implemented |
| Phase 4.4 | Project Queries | ✅ Implemented |
| Phase 4.5 | Project Authorization | ✅ Implemented |

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

**Status: Milestone 3 complete (Phases 3.1–3.6 implemented)**

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
- [x] `auth:sanctum` required; coarse role policies added in Phase 3.6

### Phase 3.4 — Lookup APIs

**Status: ✅ Implemented**

- [x] `GET /api/v1/lookups/roles`
- [x] `GET /api/v1/lookups/departments`
- [x] `GET /api/v1/lookups/job-titles`
- [x] Shared `LookupController` / `LookupService` (collections only; no show)
- [x] Soft-deleted departments/job titles excluded; sorted by `name`
- [x] Accessible to all authenticated users
- [x] PHPUnit Feature tests (`LookupApiTest`) + prior suites green

### Phase 3.5 — Search, Filtering & Pagination

**Status: ✅ Implemented**

- [x] User list `search` (`first_name`, `middle_name`, `last_name`, `email`)
- [x] Filters: `role_id`, `department_id`, `job_title_id`, `status` (composable)
- [x] Sorting: `sort` + `direction` (allowed: `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status`; default `created_at`/`desc`)
- [x] Pagination: `page` / `per_page` (default 15, max 100 clamped) with standard `meta` fields
- [x] `UserQuery` + `IndexUsersRequest`; PHPUnit `UserListQueryTest`
- [x] Out of scope preserved: Authorization (RBAC), Frontend, Projects, Tasks, Remarks, Dashboard, Reports

### Phase 3.6 — Authorization (RBAC)

**Status: ✅ Implemented**

- [x] Coarse authorization (Administrator / Project Manager / Employee)
- [x] `UserPolicy` + registration; controller `$this->authorize()`
- [x] PHPUnit `UserAuthorizationTest`
- [x] Not advanced permission matrices

### Explicitly out of scope (Milestone 3)

Department/Job Title CRUD, multi-role / multi-department users, teams, branches, organization settings, invitation emails, force password change, permission management, advanced RBAC.

---

## Phase 4 — Project Management

**Status: ✅ Phases 4.1–4.5 implemented · Milestone 4 complete**

Specification:

- [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md)
- [decisions/Project-Management.md](decisions/Project-Management.md)

### Phase 4.1 — Project Domain Foundation

**Status: ✅ Implemented**

- [x] `projects` table (soft deletes; `created_by` RESTRICT; `ProjectStatus`)
- [x] `project_members` pivot (`project_id`, `user_id`, `joined_at`, timestamps; unique pair)
- [x] Models + relations + morph alias `project`
- [x] `ProjectFactory`
- [x] PHPUnit Feature tests (`ProjectDomainFoundationTest`)

### Phase 4.2 — Project CRUD

**Status: ✅ Implemented**

- [x] Project CRUD + `PATCH /projects/{id}/status` (`ProjectController` / `ProjectService`)
- [x] Form Requests + `ProjectResource`
- [x] Soft delete; status-only patch; `created_by` set server-side; create status always `planning`
- [x] PHPUnit Feature tests (`ProjectManagementApiTest`)

### Phase 4.3 — Project Members

**Status: ✅ Implemented**

- [x] `GET/POST /api/v1/projects/{project}/members`
- [x] `DELETE /api/v1/projects/{project}/members/{user}`
- [x] No member roles / invitations / pivot permissions; duplicate → `409`; active users only
- [x] PHPUnit Feature tests (`ProjectMembersApiTest`)

### Phase 4.4 — Project Queries

**Status: ✅ Implemented**

- [x] Project list `search` (`name`, `description`)
- [x] Filters: `status`, `created_by` (composable)
- [x] Sorting + pagination (follow UserQuery conventions)
- [x] `ProjectQuery` + `IndexProjectsRequest`; PHPUnit list query tests (`ProjectListQueryTest`)

### Phase 4.5 — Project Authorization

**Status: ✅ Implemented**

- [x] Coarse authorization (Administrator / Project Manager / Employee)
- [x] `ProjectPolicy` + registration; controller `$this->authorize()`
- [x] Employee: owned **or** member visibility only (`ProjectQuery` list scoping)
- [x] PHPUnit authorization tests (`ProjectAuthorizationTest`)

### Explicitly out of scope (Milestone 4)

Tasks, Remarks, Activity Logs, Dashboard, Reports, ownership transfer, member roles/invitations, advanced RBAC, Vue Project UI.

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
