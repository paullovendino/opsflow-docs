# Milestone 3 — Organization & User Management

**Status:** Phases 3.1–3.6 implemented (Milestone 3 complete)  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-02

> Implementation specification for Milestone 3 — **complete**.  
> Next product milestone: Phase 4 — Project Management ([docs/MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) · [ROADMAP.md](../ROADMAP.md)).  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)

---

## 1. Goal

Establish OpsFlow’s organizational people model and deliver User Management on the API:

- Expand Users to the approved ERD
- Introduce Departments and Job Titles (seeded, read-only)
- Keep Roles seeded and read-only
- Expose User CRUD + status changes
- Apply coarse role-based authorization for User Management only (Phase 3.6)

---

## 2. Organizational Model

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

Independent concepts:

- **Roles** → permissions
- **Departments** → organizational grouping
- **Job Titles** → employee positions
- **Users** → people who authenticate and act

Each user belongs to **one Role** (required), **one Department** (nullable), **one Job Title** (nullable).

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **3.1 Organization Foundation** | `departments` / `job_titles` tables (`name` + `code`), models, seeders, morph aliases | ✅ Implemented |
| **3.2 User Domain Foundation** | Users ERD + FKs; relations; `UserStatus`; auth inactive `403`; `UserResource` | ✅ Implemented |
| **3.3 User Management APIs** | `UserController` / `UserService` / Form Requests / CRUD + status | ✅ Implemented |
| **3.4 Lookup APIs** | `/lookups` collections for Roles / Departments / Job Titles | ✅ Implemented |
| **3.5 Search, Filtering & Pagination** | User list search, filters, sorting, pagination `meta` | ✅ Implemented |
| **3.6 Authorization (RBAC)** | Coarse role matrix for User Management | ✅ Implemented |

---

## 4. Seed data (approved)

**Departments** (`name` / `code`): Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`

**Job Titles** (`name` / `code`): Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`

**Roles** (snake_case `name`): `administrator`, `project_manager`, `employee`

Full schema: [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)

---

## 5. Phase 3.3 — User Management APIs (implemented)

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/users` | List users (search/filters/sorting/pagination — Phase 3.5) |
| GET | `/api/v1/users/{user}` | Show user |
| POST | `/api/v1/users` | Create user |
| PUT | `/api/v1/users/{user}` | Update user |
| DELETE | `/api/v1/users/{user}` | Soft delete |
| PATCH | `/api/v1/users/{user}/status` | Update `status` only |

### Layering

| Concern | Class |
|---------|-------|
| Controller | `App\Http\Controllers\Api\V1\UserController` (thin) |
| Service | `App\Services\Users\UserService` |
| List query | `App\Queries\Users\UserQuery` |
| Authorization | `App\Policies\UserPolicy` (Phase 3.6) |
| Store validation | `StoreUserRequest` |
| Update validation | `UpdateUserRequest` |
| Status validation | `UpdateUserStatusRequest` |
| Index validation | `IndexUsersRequest` |
| Resources | `UserResource`, `RoleResource`, `DepartmentResource`, `JobTitleResource` |
| Status enum | `App\Enums\UserStatus` (`active`, `inactive`) |

### Behaviors

- Passwords hashed with `Hash::make`; never returned; unchanged on update when omitted
- Soft delete only (no hard delete)
- Status endpoint accepts `UserStatus` values only; does not modify other fields
- Relations eager-loaded for responses; nested via API Resources + `whenLoaded()`
- Authentication (login/logout/me) remains compatible; inactive login still `403`
- Authorization enforced via Laravel Policies (Phase 3.6)

---

## 6. Phase 3.4 — Lookup APIs (implemented)

| Method | Path |
|--------|------|
| GET | `/api/v1/lookups/roles` |
| GET | `/api/v1/lookups/departments` |
| GET | `/api/v1/lookups/job-titles` |

`LookupController` + `LookupService`; collections only (no show/`{id}`); soft-deleted rows excluded; ordered by `name`; `auth:sanctum` (all authenticated users; not role-gated).

---

## 7. Phase 3.5 — Search, Filtering & Pagination (implemented)

Target: `GET /api/v1/users`

| Concern | Behavior |
|---------|----------|
| Search | `search` against `first_name`, `middle_name`, `last_name`, `email` (case-insensitive) |
| Filtering | `role_id`, `department_id`, `job_title_id`, `status` (composable with search) |
| Sorting | `sort` + `direction`; allowed: `first_name`, `last_name`, `email`, `created_at`, `last_login_at`, `status`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100; values above 100 clamped to 100); items in `data`, page info in `meta` |

**Classes:** `IndexUsersRequest`, `UserQuery`, `UserService::list()`, `UserController::index()`, `ApiResponse::paginatedResponse()`

---

## 8. Phase 3.6 — Authorization (RBAC) (implemented)

Coarse role matrix for User Management only:

| Role | Capability |
|------|------------|
| Administrator | Full User Management (list, view, create, update, delete, status) |
| Project Manager | Read-only user directory (list, view) |
| Employee | View own profile only (`GET /users/{self}`) |

**Classes:** `App\Policies\UserPolicy`; registered via `Gate::policy` in `AppServiceProvider`; enforced with `$this->authorize()` in `UserController`.

Unauthorized → HTTP `403` with standard API envelope (`This action is unauthorized.`).

**Out of scope:** Permission tables, dynamic permissions, project/task/remark policies, frontend, advanced RBAC.

---

## 9. Out of Scope (Future Work)

- Department CRUD
- Job Title CRUD
- Multi-role users
- Multi-department users
- Teams / Branches / Organization Settings
- Invitation Emails / Force Password Change
- Permission Management / Advanced RBAC
- Vue UI for User Management

---

## 10. Acceptance Criteria

### Done (3.1–3.6)

- [x] Users schema matches ERD (soft deletes + FKs RESTRICT)
- [x] Departments and Job Titles migrated, seeded, soft-deletable
- [x] Morph map includes `department`, `job_title`
- [x] User CRUD + status endpoints with standard envelope
- [x] Lookup collections under `/api/v1/lookups` (roles / departments / job titles)
- [x] Inactive login rejected with dedicated `403`
- [x] `last_login_at` updated on successful login
- [x] `email_verified_at` retained
- [x] User list search + filters + sorting + pagination (3.5)
- [x] Coarse authorization enforced (3.6)
- [x] Feature tests green for 3.1–3.6
- [x] Docs synchronized for implemented behavior

---

## 11. References

- [DOMAIN_MODEL.md](DOMAIN_MODEL.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [decisions/Organization-User-Management.md](../decisions/Organization-User-Management.md)
- [decisions/Database.md](../decisions/Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
