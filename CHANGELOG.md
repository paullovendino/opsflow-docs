# Changelog

All notable OpsFlow milestones are recorded here.

Format follows a lightweight Keep a Changelog style.

---

## [Unreleased] — v1.0.0 (Development)

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
