# Changelog

All notable OpsFlow milestones are recorded here.

Format follows a lightweight Keep a Changelog style.

---

## [Unreleased] — v1.0.0 (Development)

### Reports (Milestone 7) — ✅ Implemented · Milestone 7 complete

- `GET /api/v1/reports/projects`, `GET /api/v1/reports/projects/{project}`
- `GET /api/v1/reports/employees`, `GET /api/v1/reports/employees/{user}` under `auth:sanctum`
- `ReportController` / `ReportService` / Form Requests / `ProjectReportResource` / `EmployeeReportResource`
- `ReportPolicy` via Gate abilities; Admin/PM org-wide; Employee project scoped + self-only employee detail
- Optional `from_date` / `to_date` on task `created_at`; overdue / unassigned / by_project (detail)
- No new tables
- PHPUnit Feature suites: `ProjectReportApiTest`, `EmployeeReportApiTest`, `ReportAuthorizationTest`
- Docs synchronized; next is Phase 8 — Testing (pending approval)

### Reports design package (Milestone 7) — 📋 Documentation only

- Created [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) and [decisions/Reports.md](decisions/Reports.md)
- Project + Employee report endpoints; optional date range; no new tables; visibility mirrors Projects/Tasks/Users
- Companion docs synchronized; **implementation completed in Milestone 7**

### Dashboard (Milestone 6) — ✅ Implemented · Milestone 6 complete

- `GET /api/v1/dashboard` under `auth:sanctum`
- `DashboardController` / `DashboardService` / `ShowDashboardRequest` / `DashboardResource`
- `DashboardPolicy` via `Gate::define('viewDashboard')`; Admin/PM org-wide; Employee owned-or-member scoping
- Statistics: project/task totals, by_status, by_priority; overdue; assigned_to_me
- Recent work items merged by `updated_at`; `recent_limit` default 10, max 25 (clamp)
- No new tables; soft-deleted rows excluded
- PHPUnit Feature suites: `DashboardApiTest`, `DashboardAuthorizationTest`
- Docs synchronized; next is Milestone 7 — Reports (design package)

### Dashboard design package (Milestone 6) — 📋 Documentation only

- Created [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) and [decisions/Dashboard.md](decisions/Dashboard.md)
- Read-only `GET /api/v1/dashboard`; no new tables; statistics + derived recent work items; visibility mirrors Projects/Tasks
- Companion docs synchronized; **implementation completed in Milestone 6**

### Task Status Workflow (Phase 5.6) — ✅ Implemented · Milestone 5 complete

- `PATCH /api/v1/tasks/{task}/status` under `auth:sanctum`
- `UpdateTaskStatusRequest` / `TaskController::updateStatus` / `TaskService::changeStatus`
- Any `TaskStatus` value accepted; no transition graph; create/update remain status-free
- Authorize via existing `TaskPolicy::updateStatus` (Employee: assigned to self + accessible project)
- PHPUnit Feature suite: `tests/Feature/Task/TaskStatusApiTest.php`
- Milestone 5 (Task Management) complete; docs synchronized; next is Milestone 6 — Dashboard (design package)

### Task Authorization (Phase 5.5) — ✅ Implemented

- `TaskPolicy` registered via `Gate::policy` in `AppServiceProvider`
- `$this->authorize()` on all task controller actions
- Administrator & Project Manager: full task management
- Employee: list/view tasks in owned **or** member projects only; mutations denied
- Employee `updateStatus` ability only when assigned to self (HTTP status endpoint in 5.6)
- Employee list scoping in `TaskQuery::applyVisibility`
- PHPUnit Feature suite: `tests/Feature/Task/TaskAuthorizationTest.php`
- Docs synchronized; Phase 5.6 — Task Status Workflow completed next

### Task Queries (Phase 5.4) — ✅ Implemented

- `TaskQuery` / `IndexTasksRequest` wired through `TaskService::list()` / `TaskController::index`
- Search (`title`, `description`), filters (`status`, `priority`, `project_id`, `assigned_to`, `created_by`), sorting, pagination `meta`
- Defaults: `created_at` desc with stable `id` tie-break; `per_page` 15; max 100 clamped
- Invalid query params → `422`; guest → `401`
- PHPUnit Feature suite: `tests/Feature/Task/TaskListQueryTest.php`
- Docs synchronized; Phase 5.5 — Task Authorization completed next

### Task Assignment API (Phase 5.3) — ✅ Implemented

- `PATCH /api/v1/tasks/{task}/assignment` under `auth:sanctum`
- `UpdateTaskAssignmentRequest` / `TaskController::updateAssignment` / `TaskService::changeAssignment`
- Assignee rules: active, not soft-deleted, project owner or member; `null` clears assignment
- Does not change `status`, `created_by`, or `project_id`
- PHPUnit Feature suite: `tests/Feature/Task/TaskAssignmentApiTest.php`
- Milestone 5 phase order corrected: 5.3 Assignment · 5.4 Queries · 5.5 Authorization · 5.6 Status
- Docs synchronized; Phase 5.4 — Task Queries completed next

### Task CRUD APIs (Phase 5.2) — ✅ Implemented

- `TaskController` / `TaskService` / `StoreTaskRequest` / `UpdateTaskRequest` / `TaskResource`
- Routes: `GET/POST /api/v1/tasks`, `GET/PUT/DELETE /api/v1/tasks/{task}` under `auth:sanctum`
- Create: `created_by` = auth user; status always `todo`; optional `assigned_to` (active + project owner/member); default priority `medium`
- Update: title/description/priority/due_date only (not status, assignment, project, or creator)
- Soft delete; list pagination/filters delivered in Phase 5.4; policies delivered in Phase 5.5
- PHPUnit Feature suite: `tests/Feature/Task/TaskManagementApiTest.php`
- Docs synchronized; Phase 5.3 — Task Assignment completed next

### Task Domain Foundation (Phase 5.1) — ✅ Implemented

- `tasks` table with soft deletes; FKs `project_id`, `assigned_to` (nullable), `created_by` (**RESTRICT**)
- `App\Enums\TaskStatus` / `TaskPriority`; `App\Models\Task`; `TaskFactory`
- Relations on `Task`, `Project::tasks`, `User::createdTasks` / `assignedTasks`
- Morph map alias `task`
- PHPUnit Feature suite: `tests/Feature/Task/TaskDomainFoundationTest.php`
- Docs synchronized; Phase 5.2 — Task CRUD completed next

### Milestone 5 design package — ✅ Approved (docs)

- Added [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md) and [decisions/Task-Management.md](decisions/Task-Management.md)
- Phases 5.1–5.6: Domain Foundation · CRUD · Assignment · Queries · Authorization · Status
- Approved ERD: `tasks` (soft deletes; FKs RESTRICT); `TaskStatus` / `TaskPriority` enums
- Assignment: single nullable `assigned_to` (active + project owner/member)
- APIs: `/api/v1/tasks` CRUD + `PATCH .../assignment` + `PATCH .../status` + list query contract
- Authz matrix: Admin/PM full; Employee project-scoped list/view; status when assigned to self
- Companion docs synchronized (DOMAIN_MODEL, DATABASE_DESIGN, API_SPEC, HANDOFF, ROADMAP, etc.)
- Next: Phase 5.1 — Task Domain Foundation (pending implementation approval)

### Project Authorization (Phase 4.5) — ✅ Implemented · Milestone 4 complete

- `ProjectPolicy` registered via `Gate::policy` in `AppServiceProvider`
- `$this->authorize()` on all project/member controller actions
- Administrator & Project Manager: full project management
- Employee: list/view owned **or** member projects only; mutations denied
- Employee list scoping in `ProjectQuery::applyVisibility`
- PHPUnit Feature suite: `tests/Feature/Project/ProjectAuthorizationTest.php`
- Docs synchronized; Milestone 5 design completed next

### Project Queries (Phase 4.4) — ✅ Implemented

- `ProjectQuery` / `IndexProjectsRequest` wired through `ProjectService::list()` / `ProjectController::index`
- Search (`name`, `description`), filters (`status`, `created_by`), sorting, pagination `meta`
- Defaults: `created_at` desc; `per_page` 15; max 100 clamped
- Invalid query params → `422`; guest → `401`
- PHPUnit Feature suite: `tests/Feature/Project/ProjectListQueryTest.php`
- Docs synchronized; Phase 4.5 Project Authorization completed next

### Documentation sync — Phase 4.3 complete / handoff for Phase 4.4

- HANDOFF updated as next-session starting context (Phases 4.1–4.3 complete; next is Phase 4.4)
- Documented Project Member APIs + business rules (owner ≠ member, server `joined_at`, duplicate `409`, active-only)
- Synchronized ROADMAP, DEVELOPMENT_ROADMAP, README, API_SPEC, DATABASE_DESIGN, ARCHITECTURE, TESTING, REQUIREMENTS, DOMAIN_MODEL, ADRs, and related docs
- Technical debt noted: project list not paginated yet; project routes not policy-gated until 4.5

### Project Members APIs (Phase 4.3) — ✅ Implemented

- Member routes on `ProjectController`: list / add / remove
- `StoreProjectMemberRequest` — active, non-soft-deleted `user_id` only
- `ProjectMemberResource` — user summary + server `joined_at`
- Duplicate membership → HTTP `409` (`DuplicateProjectMemberException`)
- Owner remains independent of membership; client `joined_at` ignored
- PHPUnit Feature suite: `tests/Feature/Project/ProjectMembersApiTest.php`
- Docs synchronized; Phase 4.4 Project Queries completed next

### Project CRUD APIs (Phase 4.2) — ✅ Implemented

- `ProjectController` / `ProjectService` / Form Requests / `ProjectResource`
- Routes: CRUD + `PATCH /api/v1/projects/{project}/status` under `auth:sanctum`
- Create: `created_by` = auth user; status always `planning`; client `created_by`/`status` ignored
- Update: name/description/dates only (not status or owner)
- Soft delete; status-only patch via `UpdateProjectStatusRequest`
- List pagination/filters delivered in Phase 4.4; no policies yet (4.5)
- PHPUnit Feature suite: `tests/Feature/Project/ProjectManagementApiTest.php`
- Docs synchronized; next is Phase 4.3 — Project Members (pending approval)

### Project Domain Foundation (Phase 4.1) — ✅ Implemented

- `projects` table with soft deletes, `ProjectStatus`, `created_by` RESTRICT
- `project_members` pivot with unique (`project_id`, `user_id`), `joined_at`, FK RESTRICT
- `App\Models\Project` + `User` owned/member relations; morph alias `project`
- `App\Enums\ProjectStatus`; `ProjectFactory`
- PHPUnit Feature suite: `tests/Feature/Project/ProjectDomainFoundationTest.php` (15 tests)
- Docs synchronized; next is Phase 4.2 — Project CRUD (pending approval)

### Documentation — Milestone 4 Design Package

- Created [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) and [decisions/Project-Management.md](decisions/Project-Management.md)
- Approved Phase 4 architecture: soft deletes; `ProjectStatus` enum; owner `created_by` RESTRICT; `project_members` pivot (no member roles/invitations)
- Phased into 4.1 Domain Foundation → 4.2 CRUD → 4.3 Members → 4.4 Queries → 4.5 Authorization
- Synchronized DOMAIN_MODEL, DATABASE_DESIGN, API_SPECIFICATION, ROADMAP, DEVELOPMENT_ROADMAP, ARCHITECTURE, REQUIREMENTS, TESTING, HANDOFF, CODING_STANDARDS, CURSOR_RULES, README, and Database ADR
- **No Phase 4 implementation code yet** — wait for explicit approval to start Phase 4.1

### Documentation sync — Milestone 3 complete / handoff for Phase 4

- HANDOFF rewritten as next-session starting context (Milestone 3 complete)
- Roadmap marks **✅ Milestone 3 — Complete**; next implementation milestone is **Phase 4 — Project Management**
- Phase 4 approved scope listed: Project CRUD, Members, Status, Queries, Policies, Tests
- Synchronized README, ROADMAP, DEVELOPMENT_ROADMAP, ADRs, API spec, AUTHENTICATION, ARCHITECTURE, REQUIREMENTS, TESTING, CURSOR_RULES, and related docs
- Stale “Phase 3.6 pending / deferred” wording removed from ADRs and roadmap notes

### Authorization Foundation (Phase 3.6) — ✅ Implemented

- Coarse User Management RBAC via `UserPolicy` (Administrator / Project Manager / Employee)
- Policy registered in `AppServiceProvider`; controller `$this->authorize()` on user endpoints
- Unauthorized actions return API envelope with HTTP `403`
- PHPUnit Feature suite: `tests/Feature/User/UserAuthorizationTest.php`
- Existing user list/management tests updated to use Administrator actors
- Docs synchronized; Milestone 3 backend complete

### Search, Filtering & Pagination (Phase 3.5) — ✅ Implemented

- `GET /api/v1/users` supports composable `search`, filters (`role_id`, `department_id`, `job_title_id`, `status`), sorting, and pagination
- Search fields: `first_name`, `middle_name`, `last_name`, `email` (case-insensitive)
- Sort: `sort` + `direction` (allowed: `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status`); default `created_at` / `desc`
- Pagination: `page` / `per_page` (default 15, max 100; values above 100 clamped); standard `meta` fields
- `UserQuery` owns list query concerns; `IndexUsersRequest` validates query params; `UserService` delegates listing
- `paginatedResponse` accepts Resource-shaped `data` (never raw models)
- PHPUnit Feature suite: `tests/Feature/User/UserListQueryTest.php`
- Docs synchronized; Milestone 3 backend complete; next is Phase 4 (see later documentation sync)

### Documentation sync — Phase 3.4 complete / handoff for 3.5

- HANDOFF rewritten as next-session starting context (Phases 3.1–3.4 complete)
- Phase 3.5 planned scope clarified: Search, Filtering, **Sorting**, Pagination
- Phase 3.5 out of scope stated: Authorization (RBAC), Frontend, Projects, Tasks, Remarks, Dashboard, Reports
- Synchronized ROADMAP, milestone spec, ADR, API spec, REQUIREMENTS, TESTING, ARCHITECTURE, and related docs

### Lookup APIs (Phase 3.4) — ✅ Implemented

- `GET /api/v1/lookups/roles`, `/lookups/departments`, `/lookups/job-titles`
- Shared `LookupController` + `LookupService` (collections only; no show/`{id}`)
- Soft-deleted departments/job titles excluded; results ordered by `name`
- Existing `RoleResource` / `DepartmentResource` / `JobTitleResource`; `auth:sanctum`
- PHPUnit Feature suite: `tests/Feature/Lookup/LookupApiTest.php`
- Docs synchronized to `/lookups` prefix contract

### Documentation sync — Phase 3.3 complete

- Roadmap split remaining work into Phase 3.4 Lookup APIs, 3.5 Search/Filtering/Pagination, 3.6 Authorization (RBAC)
- Synchronized README, HANDOFF, ROADMAP, DEVELOPMENT_ROADMAP, ARCHITECTURE, REQUIREMENTS, TESTING, API spec, milestone spec, ADRs

### User Management APIs (Phase 3.3) — ✅ Implemented

- `GET/POST /api/v1/users`, `GET/PUT/DELETE /api/v1/users/{user}`, `PATCH /api/v1/users/{user}/status`
- `UserController` + `UserService` (soft delete, status-only patch, Hash::make passwords)
- Form Requests: store/update/status; Resources: Role/Department/JobTitle + UserResource `whenLoaded`
- Auth required (`auth:sanctum`); role policies / filters / pagination / lookup APIs deferred to 3.4–3.6
- PHPUnit Feature suite: `tests/Feature/User/UserManagementApiTest.php`

### User Domain Foundation (Phase 3.2) — ✅ Implemented

- Users ERD migration (`role_id`, `department_id`, `job_title_id`, structured names, `avatar`, `status`, `last_login_at`, soft deletes)
- Best-effort legacy `name` → `first_name` / `middle_name` / `last_name`; drop `name`
- `UserStatus` enum; User SoftDeletes; belongsTo Role / Department / JobTitle; `full_name` accessor
- `UserResource` expanded; auth loads relations; inactive login → `403` `Account is inactive.`
- `last_login_at` set on successful login
- PHPUnit Feature suite: `tests/Feature/User/UserDomainFoundationTest.php`
- No User CRUD / controllers / services / lookup APIs

### Organization Foundation (Phase 3.1) — ✅ Implemented

- `departments` and `job_titles` migrations (`name`, `code`, nullable `description`, soft deletes)
- `Department` and `JobTitle` models with SoftDeletes and `users()` hasMany
- `DepartmentCode` / `JobTitleCode` enums (stable codes; human-readable labels)
- `DepartmentSeeder` / `JobTitleSeeder` registered in `DatabaseSeeder` (RolesSeeder unchanged)
- Morph map aliases: `department`, `job_title`
- PHPUnit Feature suite: `tests/Feature/Organization/OrganizationFoundationTest.php`
- No users table changes; no User belongsTo; no User Management APIs

### Documentation — Milestone 3 Organization & User Management (Design approved)

- Departments / Job Titles: human-readable `name` + unique stable `code` (e.g. Administration/`ADMIN`, Project Manager/`PM`)
- Split Phase 3 into 3.1 Organization Foundation and 3.2 User Management (User belongsTo relations deferred to 3.2 with FKs)
- Adopted organizational domain model: Organization → Departments, Job Titles, Roles, Users
- Added [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) as the primary business-domain reference
- Added [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) implementation specification
- Added ADR [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md)
- Expanded ERD for Users (`role_id`, `department_id`, `job_title_id`, structured names, `status`, `last_login_at`, soft deletes; keep `email_verified_at`)
- Approved Department and Job Title seed lists
- Designed `departments` and `job_titles` (seeded, soft deletes, read-only in Milestone 3)
- Standardized user list filters on IDs: `search`, `role_id`, `department_id`, `job_title_id`, plus `status` + pagination
- Approved FK **RESTRICT** for role / department / job title references (no `SET NULL`)
- Approved inactive-account login rejection: HTTP `403` (`Account is inactive.`)
- Approved lookup access for all authenticated users (roles, departments, job titles)
- Documented Planned APIs for users, roles, departments, job titles (including filters and coarse authorization)
- Renamed Phase 3 on the roadmap to **Organization & User Management**
- Updated architecture, requirements, database ADR, handoff, and related docs

### Backend Foundation (Phase 1) — Completed

- Laravel 13 API application (`opsflow-api`)
- PostgreSQL as the application database
- Laravel Sanctum installed and configured for SPA cookie mode
- API versioning under `/api/v1`
- Environment-driven CORS for the Vue SPA origins
- Agreed backend folder structure (Services, Actions, Enums, Requests, Resources, etc.)
- `BaseApiController` and standard JSON response envelope
- Global API exception rendering (`ApiExceptionRenderer`)
- Morph map foundation (`user`, `role`)
- Roles migration, `Role` model, `RoleName` enum, and `RolesSeeder`
- Health check endpoint: `GET /api/v1/health`
- Foundation documentation and ADRs

### Authentication (Phase 2, API) — Completed

- Sanctum SPA cookie authentication for the first-party Vue SPA
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/me`
- `AuthenticationService` (service-layer auth logic)
- `LoginRequest` Form Request validation
- `UserResource` API Resource responses
- Named `login` RateLimiter (`throttle:login`, 5/min per email + IP)
- Guest-only login (`guest` middleware; API `403` when already authenticated)
- Credential allowlist for `Auth::attempt()` (`email`, `password` only)
- Authentication feature test suite
- Authentication documentation (`AUTHENTICATION.md`, setup/testing/API/ADR updates)

### Changed

- Auth routes standardized under `/api/v1/auth/*`
- Documentation aligned to Laravel 13 / PHP 8.3+
- Tech Stack ADR completed (`decisions/Tech-Stack.md`)
- Phase 3 scope expanded from “User Management” to **Organization & User Management**

### Deferred

- Milestone 3 API implementation (migrations, models, controllers, services)
- Vue Pinia authentication (`opsflow-web`)
- Department / Job Title CRUD
- Advanced RBAC / permission management
- Multi-role and multi-department users
- Teams, branches, organization settings
- Invitation emails, force password change
- Registration, password reset, email verification, social login
- Draw.io diagrams, API collections, environment-config hardening
