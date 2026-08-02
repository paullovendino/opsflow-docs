# OpsFlow — Project Handoff

**Audience:** New Cursor / AI development session  
**Purpose:** Continue OpsFlow development without losing architectural consistency  
**Last updated:** 2026-08-02  
**Product version:** v1.0.0 (Development)

> Read this document first in a new chat, then follow links to detailed docs under `opsflow-docs/`.  
> **Do not invent architecture.** If unclear, ask. Prefer ADRs in `decisions/`.  
> **Domain reference:** [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)

---

## 1. Current project overview

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

**Current focus:** Milestone 3 Phases **3.1–3.3 are implemented**. Next backend work:

1. **Phase 3.4** — Lookup APIs  
2. **Phase 3.5** — Search, Filtering & Pagination  
3. **Phase 3.6** — Authorization (RBAC)

Vue Pinia auth is still pending.

---

## 2. Tech stack

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

ADR: `decisions/Tech-Stack.md`

---

## 3. Approved architecture

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
| Policies               | Coarse authorization (Phase 3.6)    |
| `ApiExceptionRenderer` | Consistent API errors               |

Auth example path:

`AuthController` → `LoginRequest` → `AuthenticationService` → session/`User` → `UserResource`

User Management example path:

`UserController` → Form Request → `UserService` → `User` + relations → `UserResource`

Details: `ARCHITECTURE.md`, `CODING_STANDARDS.md`, `docs/DOMAIN_MODEL.md`

---

## 4. Database design summary

### Implemented now

- **`roles`**: seeded `administrator`, `project_manager`, `employee` (`RoleName` enum)
- **`departments`**: human-readable `name` + unique `code` + soft deletes; seeded (e.g. Administration/`ADMIN`)
- **`job_titles`**: same shape; seeded (e.g. Project Manager/`PM`)
- **`users`** (ERD aligned):
  - `role_id` (required, RESTRICT)
  - `department_id` / `job_title_id` (nullable, RESTRICT)
  - `first_name`, `middle_name` (nullable), `last_name` (nullable for legacy name-split compatibility)
  - `email`, `email_verified_at` (kept), `password`, `avatar` (nullable)
  - `status` (`UserStatus`: `active` / `inactive`)
  - `last_login_at` (nullable)
  - timestamps + soft deletes
- Sanctum **`personal_access_tokens`**, sessions / cache / jobs as Laravel defaults require

### Milestone 3 phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **3.1** | Departments / Job Titles foundation | ✅ Implemented |
| **3.2** | Users ERD + relations + auth inactive `403` + `UserResource` | ✅ Implemented |
| **3.3** | User CRUD APIs (`/api/v1/users`) | ✅ Implemented |
| **3.4** | Lookup APIs (roles / departments / job titles) | Pending |
| **3.5** | Search, filtering & pagination | Pending |
| **3.6** | Authorization (coarse RBAC) | Pending |

### Later phases — do not invent early

- Projects, Tasks, Remarks, Activity Logs
- `organizations` table / multi-tenant org settings

### Morph map (`Relation::enforceMorphMap`)

- `user` → `App\Models\User`
- `role` → `App\Models\Role`
- `department` → `App\Models\Department`
- `job_title` → `App\Models\JobTitle`

ADRs: `DATABASE_DESIGN.md`, `decisions/Database.md`, `decisions/Organization-User-Management.md`

---

## 5. Folder structure

Canonical backend layout (`CODING_STANDARDS.md`):

```text
app/
├── Actions/
├── Enums/                  # RoleName, UserStatus, DepartmentCode, JobTitleCode
├── Exceptions/             # ApiExceptionRenderer, InvalidCredentialsException, AccountInactiveException
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/         # AuthController, HealthController, UserController
│   ├── Middleware/
│   ├── Requests/Api/V1/…   # Auth/, Users/
│   └── Resources/Api/V1/…  # User, Role, Department, JobTitle
├── Models/                 # User, Role, Department, JobTitle
├── Policies/               # Coarse UserPolicy planned in Phase 3.6
├── Providers/              # AppServiceProvider (morph map, RateLimiter)
├── Repositories/
├── Services/
│   ├── Auth/AuthenticationService.php
│   └── Users/UserService.php
└── Traits/                 # ApiResponse
```

Routes: `routes/api.php` (prefixed `/api` + `v1` groups)

---

## 6. Coding standards

- Laravel 13 + PSR-12
- `declare(strict_types=1)` where applicable
- Thin controllers; logic in Services
- Form Requests + API Resources
- DI; PHP Enums; no unapproved packages
- One feature = one commit; descriptive messages
- If requirements are unclear: **ask, do not assume**
- Follow ADRs in `opsflow-docs/decisions/`
- Keep Roles, Departments, and Job Titles conceptually independent

---

## 7. API response standard

Every API response:

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

Rules:

- Success: `success: true`, `errors: null`
- Errors: `success: false`; validation details in `errors` (HTTP `422`)
- Pagination: items in `data`, page info in `meta` (Phase 3.5 — not yet on users list)
- Use `App\Traits\ApiResponse` via `BaseApiController`
- Never return raw Eloquent models

Auth note: login puts the user under **`data.user`**; `/me` returns the user object in **`data`**.

---

## 8. Authentication implementation summary

**Mode:** Sanctum SPA cookie auth (`web` guard + session), not bearer-token login for the SPA.

### Endpoints

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

Docs: `AUTHENTICATION.md`, `decisions/Authentication.md`, `API_SPECIFICATION.md`

---

## 9. Milestone 3 — Organization & User Management

Specs:

| Document | Role |
|----------|------|
| [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) | Business domain |
| [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Implementation specification |
| [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md) | ADR |
| [DATABASE_DESIGN.md](DATABASE_DESIGN.md) | Physical ERD |
| [API_SPECIFICATION.md](API_SPECIFICATION.md) | Endpoint contracts |

### Final architecture decisions (approved)

1. Department seeds: Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`
2. Job Title seeds: Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`
3. User list filters (Phase 3.5): `search`, `role_id`, `department_id`, `job_title_id`, `status` + pagination
4. Inactive users cannot log in → HTTP `403` (`Account is inactive.`) — **implemented**
5. Keep `email_verified_at` for future compatibility — **implemented**
6. FK **RESTRICT** for Department, Job Title, and Role — **implemented**
7. Lookup endpoints for Roles / Departments / Job Titles: all authenticated users — **Phase 3.4**
8. Coarse authorization matrix — **Phase 3.6**

### Implemented User APIs (Phase 3.3)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/users` | List (no filters/pagination yet) |
| POST | `/api/v1/users` | Create (`StoreUserRequest`) |
| GET | `/api/v1/users/{user}` | Show |
| PUT | `/api/v1/users/{user}` | Update (`UpdateUserRequest`; password optional) |
| DELETE | `/api/v1/users/{user}` | Soft delete |
| PATCH | `/api/v1/users/{user}/status` | Status only (`UpdateUserStatusRequest`; `active` / `inactive`) |

**Auth:** `auth:sanctum`  
**Authorization:** not enforced yet (any authenticated user) — Phase 3.6

**Behaviors:** `UserService` owns create/update/delete/status/list/find; passwords via `Hash::make`; nested resources via `whenLoaded()`.

### Key classes (User Management)

- `App\Http\Controllers\Api\V1\UserController` — thin HTTP layer
- `App\Services\Users\UserService` — business logic
- `App\Http\Requests\Api\V1\Users\StoreUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserRequest`
- `App\Http\Requests\Api\V1\Users\UpdateUserStatusRequest`
- `App\Enums\UserStatus`
- Resources: `UserResource`, `RoleResource`, `DepartmentResource`, `JobTitleResource`

### Remaining (Phases 3.4–3.6)

**3.4 Lookup APIs**

- `GET /api/v1/roles`, `GET /api/v1/roles/{id}`
- `GET /api/v1/departments`, `GET /api/v1/departments/{id}`
- `GET /api/v1/job-titles`, `GET /api/v1/job-titles/{id}`

**3.5 Search, Filtering & Pagination** — user list `search`, ID filters, `meta` pagination

**3.6 Authorization (RBAC)** — coarse matrix only:

| Role | Capability |
|------|------------|
| Administrator | Full User Management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

### Out of scope (later)

Department/Job Title CRUD, multi-role/multi-department users, teams, branches, organization settings, invitation emails, force password change, permission management, advanced RBAC.

---

## 10. Completed milestones

| Phase | Milestone | Status |
| ----- | --------- | ------ |
| 1 | Backend Foundation (API) | **Completed** |
| 2 | Authentication (API) | **Completed** |
| 3.1 | Organization Foundation | ✅ **Implemented** |
| 3.2 | User Domain Foundation | ✅ **Implemented** |
| 3.3 | User Management APIs | ✅ **Implemented** |

Phase 1: PostgreSQL, Sanctum, `/api/v1`, CORS, envelope, exception renderer, morph map, roles seed, health check, folder structure.

Phase 2: login/logout/me, rate limit, guest login, credential allowlist, feature tests, auth docs.

Phase 3.1–3.3: org reference data, users ERD, user CRUD APIs — see `CHANGELOG.md`, `ROADMAP.md`.

---

## 11. Remaining roadmap

**Immediate next (backend):** Phase **3.4** — Lookup APIs

Then: **3.5** Search/Filtering/Pagination → **3.6** Authorization (RBAC)

**Still pending from earlier phases:**

- Vue 3 / Pinia authentication
- GitHub repos finalized (as applicable)

**Later phases:** Projects → Tasks → Dashboard → Reports → broader Testing → Deployment

**Future versions:** notifications, remarks, kanban, time tracking, mobile, multi-org, etc. (`ROADMAP.md`)

---

## 12. Important architectural decisions

| Decision            | Summary                                                |
| ------------------- | ------------------------------------------------------ |
| Laravel 13          | Keep; do not recreate/downgrade to 12                  |
| PostgreSQL          | Sole application database                              |
| Sanctum SPA cookies | First-party Vue auth; tokens deferred for mobile       |
| Auth paths          | `/api/v1/auth/login\|logout\|me` (not `/api/v1/login`) |
| Service layer       | Business logic in Services                             |
| Envelope            | Fixed JSON shape for all API responses                 |
| Morph map           | `enforceMorphMap`; existing models only                |
| Roles seed          | snake_case unique names via enum                       |
| Dept/Job Title name | Human-readable `name` + stable unique `code`           |
| Org model           | Dept / Job Title / Role / User are independent         |
| Organization table  | Not in Milestone 3 (logical single-tenant)             |
| Users ERD           | Implemented in Phase 3.2                               |
| User CRUD APIs      | Implemented in Phase 3.3 (authz deferred)              |
| User list filters   | Designed; implement in Phase 3.5                       |
| FK deletes          | RESTRICT for role / department / job title             |
| Inactive login      | HTTP `403` dedicated inactive-account response         |
| Lookups             | All authenticated users (Phase 3.4)                    |
| Coarse authz        | Phase 3.6                                              |
| Packages            | No new packages unless approved                        |
| Ask vs assume       | Unclear requirements → ask first                       |

ADRs: `decisions/Tech-Stack.md`, `Database.md`, `Authentication.md`, `Organization-User-Management.md`

---

## 13. Documentation structure

Root of `opsflow-docs/` (plus `docs/` for domain/milestone specs):

| Document                 | Use                                   |
| ------------------------ | ------------------------------------- |
| `README.md`              | Docs index + status                   |
| `HANDOFF.md`             | **This handoff**                      |
| `docs/DOMAIN_MODEL.md`   | **Primary business domain reference** |
| `docs/MILESTONE_3_…`     | Milestone 3 implementation spec       |
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
| `decisions/`             | ADRs                                  |
| `diagrams/`              | Draw.io placeholders (empty — future) |

---

## 14. Testing strategy

- PHPUnit via `php artisan test`
- DB: PostgreSQL **`opsflow_testing`** (`phpunit.xml`; no SQLite driver in current PHP)
- `RefreshDatabase`; session driver `cookie` in tests
- Stateful SPA simulated with `Origin: http://localhost:5173`
- After logout/guest follow-ups, tests may call `auth()->forgetGuards()` (Laravel 13 test-client quirk)

### Suites

| Suite | Path | Coverage |
|-------|------|----------|
| Auth | `tests/Feature/Auth/AuthenticationTest.php` | login/logout/me, guest, validation |
| Organization | `tests/Feature/Organization/OrganizationFoundationTest.php` | dept/job title migrate/seed/unique/soft delete |
| User domain | `tests/Feature/User/UserDomainFoundationTest.php` | users ERD, relations, `full_name`, inactive login |
| User APIs | `tests/Feature/User/UserManagementApiTest.php` | CRUD, status, validation, hashing, resources |

```bash
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest"
```

Deferred: dedicated `429` test, CSRF failure cases, lookup/filter/authz tests (Phases 3.4–3.6), frontend tests.

Details: `TESTING.md`

---

## 15. Current Git status

_(Captured 2026-08-02 — re-check with `git status` before committing.)_

### `opsflow-api`

- Branch: `feature/backend-foundation` (tracks `origin/feature/backend-foundation`)
- Recent commits on branch include:
  - `feat(auth): implement authentication module`
  - `feat(organization): add organization foundation`
  - `feat(user): implement user domain foundation`
- Working tree (Phase 3.3 largely uncommitted at capture time):
  - Modified: `UserResource.php`, `routes/api.php`
  - Untracked: `UserController`, `UserService`, Users Form Requests, Role/Department/JobTitle Resources, `UserManagementApiTest`

### `opsflow-docs`

- Branch: `main` (tracks `origin/main`)
- Working tree: doc updates for Milestone 3 / Phase 3.x (including this handoff) often uncommitted — re-check before commit

### Monorepo folder `OpsFlow/`

- Not a meaningful single git root; treat **api** and **docs** as separate repos
- `opsflow-web/` present as sibling (frontend not started in earnest)

**Convention:** one feature = one commit. Prefer separate focused commits (e.g. auth, organization foundation, user domain, user APIs, docs).

---

## 16. Known technical debt

1. **Lookup APIs not implemented** — roles/departments/job-titles list/show; Phase 3.4
2. **Users list has no filters/pagination** — designed; Phase 3.5
3. **No coarse authorization yet** — User APIs accept any authenticated user; Phase 3.6 policies/gates
4. **`phpunit.xml` embeds DB credentials** — move to env/local override later
5. **Laravel 13 HTTP tests** — guard caching requires `forgetGuards()` in some multi-request auth tests
6. **Draw.io diagrams empty** — future docs milestone
7. **No Postman/Bruno collection checked in** — future
8. **`opsflow-web` / Pinia auth not implemented**
9. **Historical “Laravel 12” wording** in old git commits vs actual Laravel 13
10. **Resource wrapping inconsistency** — login uses `data.user` + `resolve()`; `/me` uses Resource in `data` (intentional for now)
11. **Environment-config hardening** deferred (secrets, production cookie domain strategy)
12. **`last_name` nullable** — supports legacy name split; CRUD validation currently requires `last_name` on create/update

---

## 17. Important conventions that must be preserved

1. **Ask before assuming** unclear requirements
2. **No unapproved packages**
3. **`/api/v1` only** for versioned API routes
4. **Standard response envelope** on all API success/error paths
5. **Services for business logic**; thin controllers
6. **Form Requests + API Resources**
7. **PHP Enums** instead of magic strings
8. **Sanctum SPA cookies** for first-party auth — do not replace with ad-hoc JWT without an ADR
9. **Guest + `throttle:login`** on login; **`auth:sanctum`** on logout/me/users
10. **Credential allowlist** into `Auth::attempt()`
11. **Morph aliases only for existing models**
12. **Do not expand schema beyond the approved ERD** without updating `decisions/Database.md`
13. **Do not implement later modules early** (Projects/Tasks/Remarks/advanced RBAC) unless the milestone says so
14. **CORS / Sanctum domains stay environment-driven**
15. **Update docs/ADRs when behavior changes**
16. **One feature = one commit** when asked to commit
17. **Roles ≠ Departments ≠ Job Titles** — keep concepts independent
18. **Read domain model** (`docs/DOMAIN_MODEL.md`) before inventing entities

---

## Quick start for the next session

1. Read this handoff + `docs/DOMAIN_MODEL.md` + `ROADMAP.md` + relevant ADR
2. Confirm git branch/status in `opsflow-api` / `opsflow-docs`
3. Next backend work is typically **Phase 3.4 Lookup APIs** — wait for explicit user approval/scope
4. Do **not** modify code when the user asks for docs-only tasks
5. Prefer matching patterns in `UserService` / `UserController` / `AuthenticationService` / `ApiResponse` for new modules

### Essential commands

```bash
# API
cd opsflow-api
composer install
cp .env.example .env   # if needed
php artisan migrate
php artisan db:seed
php artisan serve
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest"

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
