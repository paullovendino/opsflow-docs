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
| **Milestone 5** | **Task Management** | ✅ **Complete** (5.1–5.6) |
| Phase 5.1 | Task Domain Foundation | ✅ Implemented |
| Phase 5.2 | Task CRUD | ✅ Implemented |
| Phase 5.3 | Task Assignment | ✅ Implemented |
| Phase 5.4 | Task Queries | ✅ Implemented |
| Phase 5.5 | Task Authorization | ✅ Implemented |
| Phase 5.6 | Task Status Workflow | ✅ Implemented |
| **Milestone 6** | **Dashboard** | ✅ **Complete** (6.1–6.4) |
| Phase 6.1 | Dashboard API Foundation | ✅ Implemented |
| Phase 6.2 | Project & Task Statistics | ✅ Implemented |
| Phase 6.3 | Recent Work Items | ✅ Implemented |
| Phase 6.4 | Dashboard Authorization | ✅ Implemented |
| **Milestone 7** | **Reports** | ✅ **Complete** (7.1–7.4) |
| Phase 7.1 | Reports API Foundation | ✅ Implemented |
| Phase 7.2 | Project Reports | ✅ Implemented |
| Phase 7.3 | Employee Reports | ✅ Implemented |
| Phase 7.4 | Reports Authorization | ✅ Implemented |
| **Milestone 8** | **Frontend Foundation** | 📋 **Design package** (awaiting implementation) |
| Phase 8.1 | Application Foundation | 📋 Designed |
| Phase 8.2 | Authentication Foundation | 📋 Designed |
| Phase 8.3 | UI Foundation | 📋 Designed |

---

## Phase 1 — Project Setup / Backend Foundation

**Status: ✅ Completed (backend)**

- [x] Laravel 13 API (`opsflow-api`)
- [x] Vue 3 frontend repository scaffold (`opsflow-web` create-vue; product app = Milestone 8)
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

**Status: ✅ Completed (API)** · Vue/Pinia auth → Milestone 8

- [x] Sanctum SPA cookie login/logout
- [x] CSRF cookie flow (configured; frontend consumes `/sanctum/csrf-cookie` in Milestone 8)
- [x] Protected API routes (`auth:sanctum`)
- [x] Current authenticated user endpoint (`GET /api/v1/auth/me`)
- [x] Login rate limiting (`throttle:login`)
- [x] Guest-only login
- [x] Credential allowlist for `Auth::attempt()`
- [x] Auth feature tests
- [x] Authentication documentation
- [ ] Pinia authentication on Vue (**Milestone 8.2**)

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

## Phase 5 — Task Management (Milestone 5)

**Status: ✅ Milestone 5 complete** (Phases 5.1–5.6)

Specification:

- [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md)
- [decisions/Task-Management.md](decisions/Task-Management.md)

### Phase 5.1 — Task Domain Foundation

**Status: ✅ Implemented**

- [x] `tasks` table (soft deletes; FKs RESTRICT; `TaskStatus` / `TaskPriority`)
- [x] Models + relations + morph alias `task`
- [x] `TaskFactory`
- [x] PHPUnit Feature tests (`TaskDomainFoundationTest`)

### Phase 5.2 — Task CRUD

**Status: ✅ Implemented**

- [x] Task CRUD (`TaskController` / `TaskService` / Form Requests / `TaskResource`)
- [x] Create: status always `todo`; optional `assigned_to`; `created_by` server-side
- [x] Update: title/description/priority/due_date only
- [x] PHPUnit Feature tests (`TaskManagementApiTest`)

### Phase 5.3 — Task Assignment

**Status: ✅ Implemented**

- [x] `PATCH /api/v1/tasks/{task}/assignment`
- [x] Single nullable assignee; active + project owner/member only
- [x] PHPUnit Feature tests (`TaskAssignmentApiTest`)

### Phase 5.4 — Task Queries

**Status: ✅ Implemented**

- [x] Task list `search` (`title`, `description`)
- [x] Filters: `status`, `priority`, `project_id`, `assigned_to`, `created_by`
- [x] Sorting + pagination (follow ProjectQuery conventions)
- [x] `TaskQuery` + `IndexTasksRequest`; PHPUnit list query tests (`TaskListQueryTest`)

### Phase 5.5 — Task Authorization

**Status: ✅ Implemented**

- [x] Coarse authorization (Administrator / Project Manager / Employee)
- [x] `TaskPolicy` + registration; controller `$this->authorize()`
- [x] Employee: project-scoped list/view; status ability only when assigned to self
- [x] PHPUnit authorization tests (`TaskAuthorizationTest`)

### Phase 5.6 — Task Status Workflow

**Status: ✅ Implemented**

- [x] `PATCH /api/v1/tasks/{task}/status`
- [x] Status not accepted on create/update; any `TaskStatus` value allowed (no transition graph)
- [x] PHPUnit Feature tests (`TaskStatusApiTest`)

### Explicitly out of scope (Milestone 5)

Multiple assignees, attachments, checklists, labels, time tracking, dependencies/subtasks, activity logs, remarks/comments, notifications, recurring tasks, Vue Task UI. (Dashboard is Milestone 6.)

---

## Phase 6 — Dashboard

**Status: ✅ Complete (Phases 6.1–6.4)**  
**Spec:** [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · **ADR:** [decisions/Dashboard.md](decisions/Dashboard.md)

> Read-only aggregates over existing Projects/Tasks. **No new tables.** Recent feed = derived work items (not Activity Logs). Vue UI deferred.

### Phase 6.1 — Dashboard API Foundation

**Status: ✅ Implemented**

- [x] `GET /api/v1/dashboard` under `auth:sanctum`
- [x] `DashboardController` / `DashboardService` / `DashboardResource` / `ShowDashboardRequest`
- [x] Guest `401`; response envelope + approved `data` shape scaffold
- [x] PHPUnit Feature tests (foundation)

### Phase 6.2 — Project & Task Statistics

**Status: ✅ Implemented**

- [x] Project `total` + `by_status` (zero-filled `ProjectStatus`)
- [x] Task `total` + `by_status` + `by_priority` + `overdue` + `assigned_to_me`
- [x] Visibility scoping (Admin/PM all; Employee owned-or-member)
- [x] PHPUnit Feature tests (statistics)

### Phase 6.3 — Recent Work Items

**Status: ✅ Implemented**

- [x] Merged recent projects + tasks by `updated_at` desc
- [x] `recent_limit` (default 10, max 25, clamp)
- [x] PHPUnit Feature tests (recent feed)

### Phase 6.4 — Dashboard Authorization

**Status: ✅ Implemented**

- [x] `DashboardPolicy` (view for all authenticated roles) + `$this->authorize('viewDashboard')`
- [x] Employee isolation Feature tests
- [x] PHPUnit Feature suite: `DashboardAuthorizationTest`

### Explicitly out of scope (Milestone 6)

Vue Dashboard UI, Activity Logs, Remarks, Reports, caching/materialized views, date-range analytics, user-directory stats, nested project dashboards, live updates.

---

## Phase 7 — Reports

**Status: ✅ Complete (Phases 7.1–7.4)**  
**Spec:** [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) · **ADR:** [decisions/Reports.md](decisions/Reports.md)

> Read-only Project / Employee analytical summaries over existing data. **No new tables.** Optional date range on task aggregates. Exports / Vue UI deferred.

### Phase 7.1 — Reports API Foundation

**Status: ✅ Implemented**

- [x] `/api/v1/reports/*` route group under `auth:sanctum`
- [x] `ReportController` / `ReportService` / `ReportPolicy`
- [x] Guest `401` on all report routes
- [x] PHPUnit Feature tests (foundation)

### Phase 7.2 — Project Reports

**Status: ✅ Implemented**

- [x] `GET /api/v1/reports/projects` (paginated summaries)
- [x] `GET /api/v1/reports/projects/{project}`
- [x] Search / status / sort / date range; task aggregates + `members_count`
- [x] PHPUnit Feature suite: `ProjectReportApiTest`

### Phase 7.3 — Employee Reports

**Status: ✅ Implemented**

- [x] `GET /api/v1/reports/employees` (Admin/PM)
- [x] `GET /api/v1/reports/employees/{user}`
- [x] Assignment aggregates; detail `by_project`
- [x] PHPUnit Feature suite: `EmployeeReportApiTest`

### Phase 7.4 — Reports Authorization

**Status: ✅ Implemented**

- [x] Full Admin / PM / Employee matrix
- [x] PHPUnit Feature suite: `ReportAuthorizationTest`

### Explicitly out of scope (Milestone 7)

PDF/CSV/Excel export, scheduled/email reports, Activity Logs, Remarks analytics, custom builders, caching/queues, Vue Reports UI, time tracking, replacing Dashboard.

---

## Phase 8 — Frontend Foundation

**Status: 📋 Design package complete** — awaiting implementation approval

Specification:

- [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md)
- [decisions/Frontend-Foundation.md](decisions/Frontend-Foundation.md)

### Phase 8.1 — Application Foundation

- [ ] Folder structure (`layouts`, `views`, `router`, `stores`, `services`, `types`, `components`, …)
- [ ] Install Pinia, Vue Router, Axios, Tailwind CSS
- [ ] Environment variables (`VITE_API_BASE_URL`, `VITE_APP_NAME`)
- [ ] Axios API client (`withCredentials`, CSRF headers, envelope types)
- [ ] Router skeleton; remove create-vue welcome UI

### Phase 8.2 — Authentication Foundation

- [ ] Sanctum CSRF → login → logout → `/me` bootstrap
- [ ] Pinia auth store + session restore via cookies
- [ ] Route guards (`requiresAuth` / `guest`)
- [ ] GuestLayout vs AuthLayout
- [ ] Login page

### Phase 8.3 — UI Foundation

- [ ] Sidebar, topbar, app shell, responsive navigation
- [ ] Loading / empty / error pages
- [ ] Toast strategy (first-party Pinia + host)
- [ ] Shared UI components (`AppButton`, `AppInput`, `AppSpinner`, …)

### Explicitly out of scope (Milestone 8)

Dashboard / Users / Projects / Tasks / Reports pages · automated frontend testing · deployment · UI frameworks · new API/schema

---

## Phase 9 — Testing

**Status: Pending** (after Milestone 8)

- API Testing (remaining gaps + broader suites)
- Frontend Testing
- Bug Fixes

---

## Phase 10 — Deployment

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
