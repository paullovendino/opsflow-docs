# OpsFlow — Project Handoff

**Audience:** New Cursor / AI development session  
**Purpose:** Continue OpsFlow development without losing architectural consistency  
**Last updated:** 2026-08-02  
**Product version:** v1.0.0 (Development)

> Read this document first in a new chat, then follow links to detailed docs under `opsflow-docs/`.  
> **Do not invent architecture.** If unclear, ask. Prefer ADRs in `decisions/`.

---

## 1. Current project overview

OpsFlow is a production-quality SaaS **project and operations management** platform for teams.

It lets administrators, project managers, and employees:

- Organize projects and tasks
- Assign work
- Monitor progress
- Improve operational visibility via dashboards/reports (planned)

**Repositories (sibling folders under `OpsFlow/`):**

| Repo | Role |
|------|------|
| `opsflow-api` | Laravel REST API (active backend) |
| `opsflow-docs` | Documentation + ADRs (this repo) |
| `opsflow-web` | Vue 3 SPA (exists as folder; frontend auth/app not built yet) |

**Current focus:** Backend API is past Foundation + Authentication. Next major backend milestone is **User Management** (Phase 3). Vue Pinia auth is still pending.

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

| Layer | Responsibility |
|-------|----------------|
| Controllers `Api/V1` | Thin HTTP only |
| Form Requests | Validation |
| API Resources | Response shaping (never raw models) |
| Services | Business logic |
| Models | Persistence / relationships |
| Enums | Domain constants (no magic strings) |
| `ApiExceptionRenderer` | Consistent API errors |

Auth example path:

`AuthController` → `LoginRequest` → `AuthenticationService` → session/`User` → `UserResource`

Details: `ARCHITECTURE.md`, `CODING_STANDARDS.md`

---

## 4. Database design summary

### Implemented now

- **`roles`**: `id`, `name` (unique snake_case), `description`, timestamps  
  Seeded: `administrator`, `project_manager`, `employee` (`RoleName` enum)
- Laravel default **`users`** (still `name`/`email`/`password` — **not** full ERD yet)
- Sanctum **`personal_access_tokens`**
- Sessions / cache / jobs as Laravel defaults require

### ERD targets (later phases — do not invent early)

- Users: `role_id`, `first_name`, `last_name`, `avatar`, `status`, …
- Projects, Tasks, Activity Logs

### Morph map (`Relation::enforceMorphMap`)

Registered only for existing models:

- `user` → `App\Models\User`
- `role` → `App\Models\Role`

Add aliases incrementally when models are introduced.

ADRs: `DATABASE_DESIGN.md`, `decisions/Database.md`

---

## 5. Folder structure

Canonical backend layout (`CODING_STANDARDS.md`):

```text
app/
├── Actions/
├── Enums/                  # e.g. RoleName
├── Exceptions/             # ApiExceptionRenderer, InvalidCredentialsException
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/         # AuthController, HealthController, …
│   ├── Middleware/
│   ├── Requests/Api/V1/…
│   └── Resources/Api/V1/…
├── Models/                 # User, Role
├── Policies/
├── Providers/              # AppServiceProvider (morph map, RateLimiter)
├── Repositories/
├── Services/               # e.g. Services/Auth/AuthenticationService
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
- Pagination: items in `data`, page info in `meta`
- Use `App\Traits\ApiResponse` via `BaseApiController`
- Never return raw Eloquent models

Auth note: login puts the user under **`data.user`** for future extensibility; `/me` returns the user object in **`data`**.

---

## 8. Authentication implementation summary

**Mode:** Sanctum SPA cookie auth (`web` guard + session), not bearer-token login for the SPA.

### Endpoints

| Method | Path | Middleware | Notes |
|--------|------|------------|-------|
| GET | `/sanctum/csrf-cookie` | — | CSRF cookie |
| POST | `/api/v1/auth/login` | `guest`, `throttle:login` | Session login |
| POST | `/api/v1/auth/logout` | `auth:sanctum` | Invalidate session |
| GET | `/api/v1/auth/me` | `auth:sanctum` | Current user |

### Behaviors

- Login: attempt **only** `email` + `password`; regenerate session
- Logout: logout + invalidate session + regenerate CSRF token
- Invalid credentials → `401` (`InvalidCredentialsException`)
- Validation failure → `422`
- Already authenticated login → `403` (`Already authenticated.`)
- Rate limit: named limiter `login` — **5/min** per `email|ip` → `429`
- Out of scope: register, reset password, email verify, social login, RBAC enforcement

### Key classes

- `App\Http\Controllers\Api\V1\AuthController`
- `App\Services\Auth\AuthenticationService`
- `App\Http\Requests\Api\V1\Auth\LoginRequest`
- `App\Http\Resources\Api\V1\UserResource`

### SPA client requirements

- `Origin`/`Referer` on stateful domains (local: `http://localhost:5173`)
- Axios `withCredentials: true`
- CSRF header from `XSRF-TOKEN` cookie

Docs: `AUTHENTICATION.md`, `decisions/Authentication.md`, `API_SPECIFICATION.md`

---

## 9. Completed milestones

| Phase | Milestone | Status |
|-------|-----------|--------|
| 1 | Backend Foundation (API) | **Completed** |
| 2 | Authentication (API) | **Completed** |

Phase 1 includes: PostgreSQL config, Sanctum install, `/api/v1`, CORS, envelope, exception renderer, morph map, roles seed, health check, folder structure.

Phase 2 API includes: login/logout/me, rate limit, guest login, credential allowlist, feature tests, auth docs.

See `CHANGELOG.md`, `ROADMAP.md`.

---

## 10. Remaining roadmap

**Immediate next (backend):** Phase 3 — User Management

- Align `users` schema with ERD (`role_id`, names, status, …)
- User CRUD, role assignment, activate/deactivate
- Then RBAC enforcement (policies/gates)

**Still pending from earlier phases:**

- Vue 3 repo / Pinia authentication
- GitHub repos finalized (as applicable)

**Later phases:** Projects → Tasks → Dashboard → Reports → broader Testing → Deployment  

**Future versions:** notifications, kanban, time tracking, mobile, etc. (`ROADMAP.md`)

---

## 11. Important architectural decisions

| Decision | Summary |
|----------|---------|
| Laravel 13 | Keep; do not recreate/downgrade to 12 |
| PostgreSQL | Sole app database |
| Sanctum SPA cookies | First-party Vue auth; tokens deferred for mobile |
| Auth paths | `/api/v1/auth/login\|logout\|me` (not `/api/v1/login`) |
| Service layer | Business logic in Services |
| Envelope | Fixed JSON shape for all API responses |
| Morph map | `enforceMorphMap`; existing models only |
| Roles seed | snake_case unique names via enum |
| Users ERD | Deferred until User Management |
| Packages | No new packages unless approved |
| Ask vs assume | Unclear requirements → ask first |

ADRs: `decisions/Tech-Stack.md`, `Database.md`, `Authentication.md`

---

## 12. Documentation structure

Root of `opsflow-docs/` (plus `docs/HANDOFF.md`):

| Document | Use |
|----------|-----|
| `README.md` | Docs index + status |
| `docs/HANDOFF.md` | **This handoff** |
| `PROJECT_OVERVIEW.md` | Product overview |
| `REQUIREMENTS.md` | Functional requirements |
| `ARCHITECTURE.md` | System architecture |
| `AUTHENTICATION.md` | Auth module guide |
| `API_SPECIFICATION.md` | Endpoint contracts |
| `DATABASE_DESIGN.md` | ERD |
| `CODING_STANDARDS.md` | Conventions |
| `CURSOR_RULES.md` | Agent rules |
| `SETUP.md` | Local setup |
| `TESTING.md` | Test guide |
| `ROADMAP.md` | Canonical roadmap |
| `DEVELOPMENT_ROADMAP.md` | Pointer to `ROADMAP.md` |
| `CHANGELOG.md` | Milestone changelog |
| `UI_PAGES.md` | UI inventory |
| `decisions/` | ADRs |
| `diagrams/` | Draw.io placeholders (empty — future) |

---

## 13. Testing strategy

- PHPUnit via `php artisan test`
- Auth suite: `tests/Feature/Auth/AuthenticationTest.php`
- Run: `php artisan test --filter=AuthenticationTest`
- DB: PostgreSQL **`opsflow_testing`** (configured in `phpunit.xml`; no SQLite driver in current PHP)
- `RefreshDatabase`; session driver `cookie` in tests
- Stateful SPA simulated with `Origin: http://localhost:5173`
- After logout/guest follow-ups, tests may call `auth()->forgetGuards()` (Laravel 13 test-client quirk)

Coverage today: login success/fail/validation, guest rejection, `/me` auth/unauth, logout.

Deferred: dedicated `429` test, CSRF failure cases, frontend tests.

Details: `TESTING.md`

---

## 14. Current Git status

*(Captured 2026-08-02 — re-check with `git status` before committing.)*

### `opsflow-api`

- Branch: `feature/backend-foundation` (tracks `origin/feature/backend-foundation`)
- Latest commit on branch: `feat: setup backend foundation`
- **Note:** `main` commit message still says “Initialize Laravel 12 project” but app runs **Laravel 13** — trust `composer.json` / Tech Stack ADR
- Working tree: **Authentication work largely uncommitted**
  - Modified: `ApiExceptionRenderer`, `AppServiceProvider`, `bootstrap/app.php`, `phpunit.xml`, `routes/api.php`, `tests/TestCase.php`
  - Untracked: Auth controller/service/requests/resources, `InvalidCredentialsException`, `tests/Feature/Auth/`

### `opsflow-docs`

- Branch: `main` (tracks `origin/main`)
- Latest: `updated docs`
- Working tree: **many doc updates uncommitted** (API spec, auth docs, roadmap, changelog, tech stack ADR, etc.)

### Monorepo folder `OpsFlow/`

- Not a meaningful single git root (`No commits yet on master`); treat **api** and **docs** as separate repos
- `opsflow-web/` present as untracked sibling (frontend not started in earnest)

**Convention:** one feature = one commit. Consider committing auth API and docs as separate focused commits when the user requests.

---

## 15. Known technical debt

1. **Users schema ≠ ERD** — still default Laravel `users`; `role_id` / profile fields deferred to Phase 3
2. **`User::role()` exists without `role_id` column** — do not eager-load/use until migration exists
3. **`phpunit.xml` embeds DB credentials** — move to env/local override later
4. **Laravel 13 HTTP tests** — guard caching requires `forgetGuards()` in some multi-request auth tests
5. **Draw.io diagrams empty** — future docs milestone
6. **No Postman/Bruno collection checked in** — future
7. **`opsflow-web` / Pinia auth not implemented**
8. **Historical “Laravel 12” wording** in old git commits vs actual Laravel 13
9. **Resource wrapping inconsistency** — login uses `data.user` + `resolve()`; `/me` uses Resource in `data` (intentional for now; unify carefully later)
10. **Environment-config hardening** deferred (secrets, production cookie domain strategy)

---

## 16. Important conventions that must be preserved

1. **Ask before assuming** unclear requirements  
2. **No unapproved packages**  
3. **`/api/v1` only** for versioned API routes  
4. **Standard response envelope** on all API success/error paths  
5. **Services for business logic**; thin controllers  
6. **Form Requests + API Resources**  
7. **PHP Enums** instead of magic strings  
8. **Sanctum SPA cookies** for first-party auth — do not replace with ad-hoc JWT without an ADR  
9. **Guest + `throttle:login`** on login; **`auth:sanctum`** on logout/me  
10. **Credential allowlist** into `Auth::attempt()`  
11. **Morph aliases only for existing models**  
12. **Do not expand schema beyond the approved ERD** without updating `decisions/Database.md`  
13. **Do not implement later modules early** (Projects/Tasks/RBAC) unless the milestone says so  
14. **CORS / Sanctum domains stay environment-driven**  
15. **Update docs/ADRs when behavior changes**  
16. **One feature = one commit** when asked to commit  

---

## Quick start for the next session

1. Read this handoff + `ROADMAP.md` + relevant ADR  
2. Confirm git branch/status in `opsflow-api` / `opsflow-docs`  
3. Next backend work is typically **Phase 3 User Management** — wait for explicit user approval/scope  
4. Do **not** modify code when the user asks for docs-only tasks  
5. Prefer matching patterns in `AuthenticationService` / `AuthController` / `ApiResponse` for new modules  

### Essential commands

```bash
# API
cd opsflow-api
composer install
cp .env.example .env   # if needed
php artisan migrate
php artisan db:seed
php artisan serve
php artisan test --filter=AuthenticationTest

# Docs
cd opsflow-docs
# edit markdown only unless asked otherwise
```

---

## Document control

| Field | Value |
|-------|-------|
| Location | `opsflow-docs/docs/HANDOFF.md` |
| Supersedes | Ad-hoc chat memory |
| Maintain | Update when milestones complete or ADRs change |
