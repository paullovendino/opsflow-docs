# OpsFlow — Project Handoff

**Audience:** New Cursor / AI development session  
**Purpose:** Continue OpsFlow development without losing architectural consistency  
**Last updated:** 2026-08-02  
**Product version:** v1.0.0 (Development)

> **Start here.** Then read [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md), [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md), [ROADMAP.md](ROADMAP.md), and the relevant ADR under `decisions/`.  
> **Do not invent architecture.** If unclear, ask. Prefer ADRs in `decisions/`.  
> **Do not implement the next phase until the user explicitly approves scope.**

---

## 1. Project purpose

OpsFlow is a production-quality SaaS **project and operations management** platform for teams.

It lets administrators, project managers, and employees:

- Organize projects and tasks
- Assign work
- Monitor progress
- Improve operational visibility via dashboards/reports (planned)

**Organizational people model:**

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

**Repositories (sibling folders under `OpsFlow/`):**

| Repo           | Role                                                          |
| -------------- | ------------------------------------------------------------- |
| `opsflow-api`  | Laravel REST API (active backend)                             |
| `opsflow-docs` | Documentation + ADRs (this repo)                              |
| `opsflow-web`  | Vue 3 SPA (exists as folder; frontend auth/app not built yet) |

**Current status:** ✅ **Phase 4.3 — Project Members complete**. Next: **Phase 4.4 — Project Queries** — wait for explicit approval.

| Field | Value |
|-------|--------|
| Current milestone | Milestone 4 — Project Management |
| Current phase | Phase 4.3 complete · next is **4.4** |
| Completed (M4) | ✅ 4.1 Domain Foundation · ✅ 4.2 CRUD · ✅ 4.3 Members |
| Remaining (M4) | ⏳ 4.4 Project Queries · ⏳ 4.5 Project Authorization |

---

## 2. Current architecture

Separated SPA + API:

```text
opsflow-web (Vue)  --cookie + CSRF-->  opsflow-api (Laravel)  -->  PostgreSQL
```

### Backend layering (must preserve)

| Layer                  | Responsibility                      |
| ---------------------- | ----------------------------------- |
| Controllers `Api/V1`   | Thin HTTP only                      |
| Form Requests          | Validation                          |
| API Resources          | Response shaping (never raw models) |
| Services               | Business logic                      |
| Models                 | Persistence / relationships         |
| Enums                  | Domain constants (no magic strings) |
| Policies               | Coarse authorization (`UserPolicy`; `ProjectPolicy` in 4.5) |
| Queries                | List search / filter / sort / paginate (`UserQuery`; `ProjectQuery` in 4.4) |
| `ApiExceptionRenderer` | Consistent API errors (incl. project member `409`) |

### Request paths (implemented)

```text
Auth:     AuthController → LoginRequest → AuthenticationService → UserResource
Users:    UserController → authorize(UserPolicy) → Form Request → UserService → UserResource
          index → IndexUsersRequest → UserService → UserQuery → UserResource + pagination meta
Projects: ProjectController → Form Request → ProjectService → ProjectResource
          members → StoreProjectMemberRequest → ProjectService → ProjectMemberResource
          (ProjectQuery in 4.4; ProjectPolicy in 4.5)
Lookups:  LookupController → LookupService → RoleResource / DepartmentResource / JobTitleResource
```

Details: [ARCHITECTURE.md](ARCHITECTURE.md), [CODING_STANDARDS.md](CODING_STANDARDS.md), [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)

### Folder structure (`opsflow-api`)

```text
app/
├── Actions/
├── Enums/                  # RoleName, UserStatus, DepartmentCode, JobTitleCode, ProjectStatus
├── Exceptions/             # ApiExceptionRenderer, InvalidCredentialsException, AccountInactiveException, DuplicateProjectMemberException
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/         # AuthController, HealthController, UserController, LookupController, ProjectController
│   ├── Middleware/
│   ├── Requests/Api/V1/…   # Auth/, Users/, Projects/ (incl. StoreProjectMemberRequest)
│   └── Resources/Api/V1/…  # User, Role, Department, JobTitle, Project, ProjectMember
├── Models/                 # User, Role, Department, JobTitle, Project
├── Policies/               # UserPolicy; ProjectPolicy in Phase 4.5
├── Providers/              # AppServiceProvider (morph map, RateLimiter)
├── Queries/
│   └── Users/UserQuery.php # List search / filters / sorting / pagination (+ Projects/ProjectQuery in Phase 4.4)
├── Repositories/
├── Services/
│   ├── Auth/AuthenticationService.php
│   ├── Lookups/LookupService.php
│   ├── Users/UserService.php
│   └── Projects/ProjectService.php
└── Traits/                 # ApiResponse (incl. paginatedResponse)
```

Routes: `routes/api.php` (prefixed `/api` + `v1` groups)

---

## 3. Tech stack

### Backend (`opsflow-api`)

- **Laravel 13** (do **not** downgrade to Laravel 12)
- **PHP 8.3+**
- **PostgreSQL** only (app DB; not SQLite for runtime)
- **Laravel Sanctum** — SPA cookie authentication
- **REST API** under `/api/v1`

### Frontend (planned / separate repo)

- Vue 3 Composition API + TypeScript
- Pinia, Vue Router, Tailwind CSS, Axios (`withCredentials: true`)

### Tooling

- Cursor, Git/GitHub, Postman/Bruno/Insomnia, pgAdmin, PHPUnit

ADR: [decisions/Tech-Stack.md](decisions/Tech-Stack.md)

---

## 4. Completed milestones

| Phase | Milestone | Status |
| ----- | --------- | ------ |
| 1 | Backend Foundation (API) | **Completed** |
| 2 | Authentication (API) | **Completed** |
| **3** | **Organization & User Management** | ✅ **Complete** |
| 3.1 | Organization Foundation | ✅ **Implemented** |
| 3.2 | User Domain Foundation | ✅ **Implemented** |
| 3.3 | User Management APIs | ✅ **Implemented** |
| 3.4 | Lookup APIs | ✅ **Implemented** |
| 3.5 | Search, Filtering & Pagination | ✅ **Implemented** |
| 3.6 | Authorization (RBAC) | ✅ **Implemented** |
| **4** | **Project Management** | In progress |
| 4.1 | Project Domain Foundation | ✅ **Implemented** |
| 4.2 | Project CRUD | ✅ **Implemented** |
| 4.3 | Project Members | ✅ **Implemented** |
| 4.4 | Project Queries | ⏳ Pending |
| 4.5 | Project Authorization | ⏳ Pending |

**Phase 1:** PostgreSQL, Sanctum, `/api/v1`, CORS, envelope, exception renderer, morph map, roles seed, health check, folder structure.

**Phase 2:** login/logout/me, rate limit, guest login, credential allowlist, feature tests, auth docs.

**Phase 3.1:** `departments` / `job_titles` tables (`name` + `code`), models, soft deletes, seeders, morph aliases.

**Phase 3.2:** Users ERD (`role_id`, names, `department_id`, `job_title_id`, `status`, soft deletes, keep `email_verified_at`), relations, `UserStatus`, inactive login `403`, expanded `UserResource`.

**Phase 3.3:** User CRUD + status patch (`UserController` / `UserService` / Form Requests / nested resources).

**Phase 3.4:** Lookup collections under `/api/v1/lookups` (`LookupController` / `LookupService`).

**Phase 3.5:** User list search / filters / sorting / pagination (`UserQuery` / `IndexUsersRequest`).

**Phase 3.6:** Coarse User Management authorization (`UserPolicy` + controller `$this->authorize()`).

**Phase 4.1:** `projects` / `project_members` tables; `Project` model; `ProjectStatus`; relations; morph `project`; `ProjectFactory`; Feature tests.

**Phase 4.2:** Project CRUD + status patch (`ProjectController` / `ProjectService` / Form Requests / `ProjectResource`).

**Phase 4.3:** Project Members list/add/remove (`StoreProjectMemberRequest` / `ProjectMemberResource`; duplicate `409`).

See [CHANGELOG.md](CHANGELOG.md), [ROADMAP.md](ROADMAP.md).

---

## 5. Remaining roadmap

### Immediate next (backend)

**Phase 4.4 — Project Queries** (see §12) — wait for explicit implementation approval

Completed: Phase 4.1 Domain Foundation · Phase 4.2 Project CRUD · Phase 4.3 Project Members.

Remaining Milestone 4:

- ⏳ Phase 4.4 — Project Queries
- ⏳ Phase 4.5 — Project Authorization

Specification: [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) · [decisions/Project-Management.md](decisions/Project-Management.md)

### Still pending from earlier phases

- Vue 3 / Pinia authentication (`opsflow-web`)
- GitHub repos finalized (as applicable)

### Later phases (do not invent early)

Tasks → Dashboard → Reports → broader Testing → Deployment

### Future versions

Notifications, remarks, kanban, time tracking, mobile, multi-org, etc. ([ROADMAP.md](ROADMAP.md))

### Explicitly out of scope for remaining Phase 4 work / later milestones

- Phase 4.4–4.5 until explicitly approved
- Tasks / Remarks / Dashboard / Reports
- Advanced permission matrices beyond coarse role policies
- Frontend User Management / Project UI

---

## 6. Authentication contract

**Mode:** Sanctum SPA cookie auth (`web` guard + session), not bearer-token login for the SPA.

| Method | Path                   | Middleware                | Notes              |
| ------ | ---------------------- | ------------------------- | ------------------ |
| GET    | `/sanctum/csrf-cookie` | —                         | CSRF cookie        |
| POST   | `/api/v1/auth/login`   | `guest`, `throttle:login` | Session login      |
| POST   | `/api/v1/auth/logout`  | `auth:sanctum`            | Invalidate session |
| GET    | `/api/v1/auth/me`      | `auth:sanctum`            | Current user       |

### Behaviors

- Login: attempt **only** `email` + `password`; regenerate session; set `last_login_at`
- Eager-load `role`, `department`, `jobTitle` for auth responses
- Logout: logout + invalidate session + regenerate CSRF token
- Invalid credentials → `401` (`InvalidCredentialsException`)
- Inactive account → `403` (`AccountInactiveException`, message `Account is inactive.`)
- Validation failure → `422`
- Already authenticated login → `403` (`Already authenticated.`)
- Rate limit: named limiter `login` — **5/min** per `email|ip` → `429`
- Out of scope: register, reset password, email verify, social login, advanced RBAC

### Key classes

- `App\Http\Controllers\Api\V1\AuthController`
- `App\Services\Auth\AuthenticationService`
- `App\Http\Requests\Api\V1\Auth\LoginRequest`
- `App\Http\Resources\Api\V1\UserResource`
- `App\Exceptions\AccountInactiveException`

### SPA client requirements

- `Origin`/`Referer` on stateful domains (local: `http://localhost:5173`)
- Axios `withCredentials: true`
- CSRF header from `XSRF-TOKEN` cookie

Docs: [AUTHENTICATION.md](AUTHENTICATION.md), [decisions/Authentication.md](decisions/Authentication.md), [API_SPECIFICATION.md](API_SPECIFICATION.md)

---

## 7. User Management implementation summary (Phase 3.3)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/users` | List with search/filters/sorting/pagination (Phase 3.5) |
| POST | `/api/v1/users` | Create (`StoreUserRequest`) |
| GET | `/api/v1/users/{user}` | Show |
| PUT | `/api/v1/users/{user}` | Update (`UpdateUserRequest`; password optional) |
| DELETE | `/api/v1/users/{user}` | Soft delete |
| PATCH | `/api/v1/users/{user}/status` | Status only (`UpdateUserStatusRequest`; `active` / `inactive`) |

**Auth:** `auth:sanctum`  
**Authorization:** `UserPolicy` (Phase 3.6) — Administrator full; Project Manager read-only; Employee own profile only

**Behaviors:** `UserService` owns create/update/delete/status/list; list delegates to `UserQuery`; passwords via `Hash::make`; nested resources via `whenLoaded()`.

### Key classes

- `App\Http\Controllers\Api\V1\UserController`
- `App\Services\Users\UserService`
- `App\Queries\Users\UserQuery`
- `App\Http\Requests\Api\V1\Users\IndexUsersRequest`
- `App\Http\Requests\Api\V1\Users\StoreUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserStatusRequest`
- `App\Enums\UserStatus`
- `App\Policies\UserPolicy`
- Resources: `UserResource`, `RoleResource`, `DepartmentResource`, `JobTitleResource`

---

## 8. Lookup API implementation summary (Phase 3.4)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/lookups/roles` | `RoleResource` collection; ordered by `name` |
| GET | `/api/v1/lookups/departments` | Soft-deleted excluded; ordered by `name` |
| GET | `/api/v1/lookups/job-titles` | Soft-deleted excluded; ordered by `name` |

**Auth:** `auth:sanctum` (all authenticated users)  
**Classes:** `LookupController`, `LookupService`  
**Scope:** Collections only — no show/`{id}`, no CRUD, no filters/pagination on lookups.

---

## 8b. Search, Filtering & Pagination summary (Phase 3.5)

| Concern | Behavior |
|---------|----------|
| Search | `search` on `first_name`, `middle_name`, `last_name`, `email` (case-insensitive) |
| Filtering | `role_id`, `department_id`, `job_title_id`, `status` (composable) |
| Sorting | `sort` + `direction`; allowed `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100 clamped); standard `meta` |

**Classes:** `IndexUsersRequest`, `UserQuery`, `UserService::list()`, `paginatedResponse` + `UserResource::collection`

---

## 8c. Authorization summary (Phase 3.6)

| Role | List | View | Create | Update | Delete | Status |
|------|------|------|--------|--------|--------|--------|
| Administrator | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Employee | ❌ | own only | ❌ | ❌ | ❌ | ❌ |

**Classes:** `UserPolicy`; `Gate::policy` in `AppServiceProvider`; `$this->authorize()` in `UserController`

---

## 8d. Project CRUD summary (Phase 4.2)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/projects` | Simple collection (query polish in 4.4) |
| POST | `/api/v1/projects` | Create; `created_by` = auth user; status always `planning` |
| GET | `/api/v1/projects/{project}` | Show with nested `owner` |
| PUT | `/api/v1/projects/{project}` | Update name/description/dates only (not status/owner) |
| DELETE | `/api/v1/projects/{project}` | Soft delete |
| PATCH | `/api/v1/projects/{project}/status` | Status only (`ProjectStatus`) |

**Auth:** `auth:sanctum` (policies in Phase 4.5)  
**Classes:** `ProjectController`, `ProjectService`, `StoreProjectRequest`, `UpdateProjectRequest`, `UpdateProjectStatusRequest`, `ProjectResource`

---

## 8e. Project Members summary (Phase 4.3)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/projects/{project}/members` | List members (`ProjectMemberResource`) |
| POST | `/api/v1/projects/{project}/members` | Add member (`user_id`) |
| DELETE | `/api/v1/projects/{project}/members/{user}` | Remove member (pivot hard delete) |

**Auth:** `auth:sanctum` (policies in Phase 4.5)

**Approved business rules:**

- Owner (`created_by`) is independent from membership — **not** auto-added to `project_members`
- `joined_at` is server-generated only; client-supplied `joined_at` is ignored
- Duplicate membership → HTTP `409` (`DuplicateProjectMemberException` / `User is already a member of this project.`)
- Only **active**, non-soft-deleted users may be added (`422` otherwise)
- Removing a member does not change `created_by`
- Unknown membership on delete → `404`

**Classes:** `ProjectController` member actions; `ProjectService::listMembers` / `addMember` / `removeMember`; `StoreProjectMemberRequest`; `ProjectMemberResource`

---

## 9. Current database status

### Implemented tables

- **`roles`**: seeded `administrator`, `project_manager`, `employee` (`RoleName` enum); no soft deletes in Milestone 3
- **`departments`**: human-readable `name` + unique `code` + soft deletes; seeded (Administration/`ADMIN`, …)
- **`job_titles`**: same shape; seeded (Project Manager/`PM`, …)
- **`users`** (ERD aligned):
  - `role_id` (required, RESTRICT)
  - `department_id` / `job_title_id` (nullable, RESTRICT)
  - `first_name`, `middle_name` (nullable), `last_name` (nullable for legacy name-split compatibility)
  - `email`, `email_verified_at` (kept), `password`, `avatar` (nullable)
  - `status` (`UserStatus`: `active` / `inactive`)
  - `last_login_at` (nullable)
  - timestamps + soft deletes
- **`projects`** (Phase 4.1):
  - `name`, `description` (nullable), `status` (`ProjectStatus`), `start_date` / `due_date` (nullable)
  - `created_by` → `users.id` (**RESTRICT**)
  - timestamps + soft deletes
- **`project_members`** (Phase 4.1):
  - `project_id`, `user_id` (**RESTRICT**), `joined_at`, timestamps
  - unique (`project_id`, `user_id`); no soft deletes; no member roles
- Sanctum **`personal_access_tokens`**, sessions / cache / jobs as Laravel defaults require

### Morph map (`Relation::enforceMorphMap`)

- `user` → `App\Models\User`
- `role` → `App\Models\Role`
- `department` → `App\Models\Department`
- `job_title` → `App\Models\JobTitle`
- `project` → `App\Models\Project`

### Not yet

- `organizations` table / multi-tenant org settings
- Project Queries (Phase 4.4), Policies (4.5)
- Tasks, Remarks, Activity Logs tables

ADRs: [DATABASE_DESIGN.md](DATABASE_DESIGN.md), [decisions/Database.md](decisions/Database.md), [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md), [decisions/Project-Management.md](decisions/Project-Management.md)

---

## 10. Engineering conventions

### Response envelope

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

- Success: `success: true`, `errors: null`
- Errors: `success: false`; validation details in `errors` (HTTP `422`)
- Pagination (Phase 3.5 — implemented): items in `data`, page info in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`)
- Use `App\Traits\ApiResponse` via `BaseApiController` (`paginatedResponse` for list endpoints)
- Never return raw Eloquent models
- Auth note: login puts the user under **`data.user`**; `/me` returns the user object in **`data`**

### Must preserve

1. **Ask before assuming** unclear requirements
2. **No unapproved packages**
3. **`/api/v1` only** for versioned API routes
4. **Standard response envelope** on all API success/error paths
5. **Services for business logic**; thin controllers; **query objects** for list search/filter/sort/pagination
6. **Form Requests + API Resources** (Form Requests not required for parameter-less lookup GETs)
7. **PHP Enums** instead of magic strings
8. **Sanctum SPA cookies** for first-party auth — do not replace with ad-hoc JWT without an ADR
9. **Guest + `throttle:login`** on login; **`auth:sanctum`** on logout/me/users/lookups/projects
10. **Credential allowlist** into `Auth::attempt()`
11. **Morph aliases only for existing models**
12. **Do not expand schema beyond the approved ERD** without updating `decisions/Database.md`
13. **Do not implement later modules early** (Phase 4.4–4.5 / Tasks/Remarks) unless the milestone phase is approved
14. **Authorize via Policies** — do not scatter manual role checks outside policies (`ProjectPolicy` arrives in 4.5)
15. **CORS / Sanctum domains stay environment-driven**
16. **Update docs/ADRs when behavior changes**
17. **One feature = one commit** when asked to commit
18. **Roles ≠ Departments ≠ Job Titles** — keep concepts independent
19. **Read domain model** (`docs/DOMAIN_MODEL.md`) and milestone specs before inventing entities
20. **Project owner ≠ automatic member** — `created_by` and `project_members` are independent; duplicate members → `409`

### Coding standards snapshot

- Laravel 13 + PSR-12 + `declare(strict_types=1)`
- DI; no magic strings; no duplicated logic
- Match patterns: `UserController` / `UserService` / `UserQuery` / `UserPolicy`; `LookupController` / `LookupService`; `ProjectController` / `ProjectService`

Details: [CODING_STANDARDS.md](CODING_STANDARDS.md), [CURSOR_RULES.md](CURSOR_RULES.md)

---

## 11. Known technical debt

1. **`phpunit.xml` embeds DB credentials** — move to env/local override later
2. **Laravel 13 HTTP tests** — guard caching requires `forgetGuards()` in some multi-request auth tests
3. **Draw.io diagrams empty** — future docs milestone
4. **No Postman/Bruno collection checked in** — future
5. **`opsflow-web` / Pinia auth not implemented**
6. **Historical “Laravel 12” wording** in old git commits vs actual Laravel 13
7. **Resource wrapping inconsistency** — login uses `data.user` + `resolve()`; `/me` uses Resource in `data` (intentional for now)
8. **Environment-config hardening** deferred (secrets, production cookie domain strategy)
9. **`last_name` nullable** — supports legacy name split; CRUD validation currently requires `last_name` on create/update
10. **Combined commit naming** — `opsflow-api` commit `ef14ade` (`feat(lookup)`) includes both Phase 3.3 User Management APIs and Phase 3.4 Lookup APIs
11. **Project list has no pagination yet** — Phase 4.2 returns a simple collection; Phase 4.4 adds `ProjectQuery`
12. **Project routes not yet policy-gated** — any authenticated user can mutate projects/members until Phase 4.5

---

## 12. Immediate next milestone

### Phase 4.4 — Project Queries

**Status:** Pending — wait for explicit user approval before coding.

**Completed:** Phase 4.1 Domain Foundation · Phase 4.2 Project CRUD · Phase 4.3 Project Members.

**Specification:** [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) §8 · [decisions/Project-Management.md](decisions/Project-Management.md)

**Phase 4.4 scope:**

| Area | Notes |
|------|-------|
| Search | `name`, `description` |
| Filters | `status`, `created_by` |
| Sorting / pagination | Follow `UserQuery` conventions |
| Classes | `ProjectQuery`, `IndexProjectsRequest`; wire `ProjectService::list()` |

**Out of scope for 4.4:** `ProjectPolicy` (4.5), Tasks, Vue UI.

**Pattern to match:** `UserQuery` / `IndexUsersRequest`

---

## Current API routes (implemented)

| Method | Path | Auth | Authorization |
|--------|------|------|---------------|
| GET | `/api/v1/health` | none | — |
| GET | `/sanctum/csrf-cookie` | none | — |
| POST | `/api/v1/auth/login` | guest + throttle | — |
| POST | `/api/v1/auth/logout` | `auth:sanctum` | — |
| GET | `/api/v1/auth/me` | `auth:sanctum` | — |
| GET | `/api/v1/users` | `auth:sanctum` | Admin, Project Manager |
| POST | `/api/v1/users` | `auth:sanctum` | Administrator |
| GET | `/api/v1/users/{user}` | `auth:sanctum` | Admin, PM; Employee own only |
| PUT | `/api/v1/users/{user}` | `auth:sanctum` | Administrator |
| DELETE | `/api/v1/users/{user}` | `auth:sanctum` | Administrator |
| PATCH | `/api/v1/users/{user}/status` | `auth:sanctum` | Administrator |
| GET | `/api/v1/lookups/roles` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/lookups/departments` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/lookups/job-titles` | `auth:sanctum` | All authenticated |
| GET | `/api/v1/projects` | `auth:sanctum` | All authenticated (policy in 4.5) |
| POST | `/api/v1/projects` | `auth:sanctum` | All authenticated (policy in 4.5) |
| GET | `/api/v1/projects/{project}` | `auth:sanctum` | All authenticated (policy in 4.5) |
| PUT | `/api/v1/projects/{project}` | `auth:sanctum` | All authenticated (policy in 4.5) |
| DELETE | `/api/v1/projects/{project}` | `auth:sanctum` | All authenticated (policy in 4.5) |
| PATCH | `/api/v1/projects/{project}/status` | `auth:sanctum` | All authenticated (policy in 4.5) |
| GET | `/api/v1/projects/{project}/members` | `auth:sanctum` | All authenticated (policy in 4.5) |
| POST | `/api/v1/projects/{project}/members` | `auth:sanctum` | All authenticated (policy in 4.5) |
| DELETE | `/api/v1/projects/{project}/members/{user}` | `auth:sanctum` | All authenticated (policy in 4.5) |

---

## Milestone 3 decisions (approved)

1. Department seeds: Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`
2. Job Title seeds: Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`
3. User list (Phase 3.5): `search`, filters (`role_id`, `department_id`, `job_title_id`, `status`), **sorting**, pagination `meta` — **implemented**
4. Inactive users cannot log in → HTTP `403` (`Account is inactive.`) — **implemented**
5. Keep `email_verified_at` for future compatibility — **implemented**
6. FK **RESTRICT** for Department, Job Title, and Role — **implemented**
7. Lookup collections under `/api/v1/lookups` — **implemented** (Phase 3.4)
8. Coarse authorization matrix — **implemented** (Phase 3.6)

Specs: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) · [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) · [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md)

---

## Testing status

- PHPUnit via `php artisan test`
- DB: PostgreSQL **`opsflow_testing`** (`phpunit.xml`; no SQLite driver in current PHP)
- `RefreshDatabase`; session driver `cookie` in tests
- Stateful SPA simulated with `Origin: http://localhost:5173`
- After logout/guest follow-ups, tests may call `auth()->forgetGuards()` (Laravel 13 test-client quirk)
- **Last known full suite:** 100 tests passed (Phases 1–4.3 coverage)

| Suite | Path | Coverage |
|-------|------|----------|
| Auth | `tests/Feature/Auth/AuthenticationTest.php` | login/logout/me, guest, validation |
| Organization | `tests/Feature/Organization/OrganizationFoundationTest.php` | dept/job title migrate/seed/unique/soft delete |
| User domain | `tests/Feature/User/UserDomainFoundationTest.php` | users ERD, relations, `full_name`, inactive login |
| User APIs | `tests/Feature/User/UserManagementApiTest.php` | CRUD, status, validation, hashing, resources |
| User list query | `tests/Feature/User/UserListQueryTest.php` | search, filters, sort, pagination, clamp, validation |
| User authz | `tests/Feature/User/UserAuthorizationTest.php` | Admin / PM / Employee policy matrix |
| Lookups | `tests/Feature/Lookup/LookupApiTest.php` | `/lookups/*` auth, soft-delete exclusion, sort by name |
| Project domain | `tests/Feature/Project/ProjectDomainFoundationTest.php` | projects/members schema, relations, soft delete, enum, FK RESTRICT, factory |
| Project APIs | `tests/Feature/Project/ProjectManagementApiTest.php` | CRUD, status, validation, owner assignment, guest `401`, resource shape |
| Project members | `tests/Feature/Project/ProjectMembersApiTest.php` | list/add/remove, duplicate `409`, active-only, soft-deleted rejected |

```bash
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest|UserListQueryTest|LookupApiTest|UserAuthorizationTest|ProjectDomainFoundationTest|ProjectManagementApiTest|ProjectMembersApiTest"
```

Deferred: dedicated `429` test, CSRF failure cases, frontend tests, Phase 4 project suites (designed in TESTING.md).

Details: [TESTING.md](TESTING.md)

---

## Current Git status

_(Re-check with `git status` before committing.)_

### `opsflow-api`

- Branch: `feature/backend-foundation` (tracks `origin/feature/backend-foundation`)
- Working tree may include **uncommitted** Phase 3.5–3.6 implementation — re-check `git status` before committing
- Recent commits (historical):
  - `ef14ade` `feat(lookup): implement lookup APIs` — includes Phase **3.3** User Management + Phase **3.4** Lookups
  - `2db5137` `feat(user): implement user domain foundation`
  - `d43af15` `feat(organization): add organization foundation`
  - `cfacf33` `feat(auth): implement authentication module`
  - `1ed3bb7` `feat: setup backend foundation`

### `opsflow-docs`

- Branch: `main` (tracks `origin/main`)
- Working tree typically has **uncommitted** Phase 4.3 / Milestone 4 documentation sync — commit when asked

### Monorepo folder `OpsFlow/`

- Not a meaningful single git root; treat **api** and **docs** as separate repos
- `opsflow-web/` present as sibling (frontend not started in earnest)

**Convention:** one feature = one commit when asked to commit.

---

## Documentation index

| Document                 | Use                                   |
| ------------------------ | ------------------------------------- |
| `README.md`              | Docs index + status                   |
| `HANDOFF.md`             | **This handoff**                      |
| `docs/DOMAIN_MODEL.md`   | **Primary business domain reference** |
| `docs/MILESTONE_3_…`     | Milestone 3 implementation spec       |
| `docs/MILESTONE_4_…`     | Milestone 4 implementation spec       |
| `PROJECT_OVERVIEW.md`    | Product overview                      |
| `REQUIREMENTS.md`        | Functional requirements               |
| `ARCHITECTURE.md`        | System architecture                   |
| `AUTHENTICATION.md`      | Auth module guide                     |
| `API_SPECIFICATION.md`   | Endpoint contracts                    |
| `DATABASE_DESIGN.md`     | ERD                                   |
| `CODING_STANDARDS.md`    | Conventions                           |
| `CURSOR_RULES.md`        | Agent rules                           |
| `SETUP.md`               | Local setup                           |
| `TESTING.md`             | Test guide                            |
| `ROADMAP.md`             | Canonical roadmap                     |
| `DEVELOPMENT_ROADMAP.md` | Pointer to `ROADMAP.md`               |
| `CHANGELOG.md`           | Milestone changelog                   |
| `UI_PAGES.md`            | UI inventory                          |
| `decisions/`             | ADRs (incl. `Project-Management.md`)  |
| `diagrams/`              | Draw.io placeholders (empty — future) |

---

## Quick start for the next session

1. Read this handoff + `docs/DOMAIN_MODEL.md` + `docs/MILESTONE_4_PROJECT_MANAGEMENT.md` §8 + `decisions/Project-Management.md`
2. Confirm git branch/status in `opsflow-api` / `opsflow-docs`
3. ✅ Phase 4.3 is complete — next is **Phase 4.4 — Project Queries** — **wait for explicit implementation approval**
4. Follow the Milestone 4 spec / ADR — do **not** invent beyond approved decisions
5. Do **not** modify code when the user asks for docs-only tasks
6. Prefer matching patterns in `ProjectController` / `ProjectService` / `UserQuery` / `IndexUsersRequest` / `UserService` / `ApiResponse`

### Essential commands

```bash
# API
cd opsflow-api
composer install
cp .env.example .env   # if needed
php artisan migrate
php artisan db:seed
php artisan serve
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest|UserListQueryTest|LookupApiTest|UserAuthorizationTest|ProjectDomainFoundationTest|ProjectManagementApiTest|ProjectMembersApiTest"

# Docs
cd opsflow-docs
# edit markdown only unless asked otherwise
```

---

## Document control

| Field      | Value                                          |
| ---------- | ---------------------------------------------- |
| Location   | `opsflow-docs/HANDOFF.md`                      |
| Supersedes | Ad-hoc chat memory                             |
| Maintain   | Update when milestones complete or ADRs change |
| Ready for  | Next session → Phase 4.4 (after implementation approval) |

## Project Principles

OpsFlow follows these engineering principles:

- Architecture first, implementation second.
- Documentation is updated before a milestone is considered complete.
- One milestone at a time; do not implement future modules.
- Controllers must remain thin.
- Business logic belongs in Services.
- Validation belongs in Form Requests.
- API responses must always use the approved response envelope.
- API Resources must be used instead of returning raw models.
- Prefer PHP Enums over magic strings.
- Use Laravel best practices and PSR-12.
- Database changes require approval before implementation.
- No additional packages without approval.
- Keep the code simple, maintainable, and production-ready.
- Every completed milestone should include:
  - Passing tests
  - Updated documentation
  - Git commit
  - Git push
