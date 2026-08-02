# Milestone 3 — Organization & User Management

**Status:** Phases 3.1–3.3 implemented · Phases 3.4–3.6 remaining  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-02

> Implementation specification for Phase 3.  
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
| **3.4 Lookup APIs** | Roles / Departments / Job Titles list + show | Pending |
| **3.5 Search, Filtering & Pagination** | User list `search`, ID filters, pagination `meta` | Pending |
| **3.6 Authorization (RBAC)** | Coarse role matrix for User Management | Pending |

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
| GET | `/api/v1/users` | List users (no filters/pagination yet) |
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
| Store validation | `StoreUserRequest` |
| Update validation | `UpdateUserRequest` |
| Status validation | `UpdateUserStatusRequest` |
| Resources | `UserResource`, `RoleResource`, `DepartmentResource`, `JobTitleResource` |
| Status enum | `App\Enums\UserStatus` (`active`, `inactive`) |

### Behaviors

- Passwords hashed with `Hash::make`; never returned; unchanged on update when omitted
- Soft delete only (no hard delete)
- Status endpoint accepts `UserStatus` values only; does not modify other fields
- Relations eager-loaded for responses; nested via API Resources + `whenLoaded()`
- Authentication (login/logout/me) remains compatible; inactive login still `403`

### Not in 3.3

Lookup APIs · search/filters/pagination · policies/authorization

---

## 6. Remaining API surface

### Roles / Departments / Job Titles (Phase 3.4)

| Method | Path |
|--------|------|
| GET | `/api/v1/roles`, `/api/v1/roles/{id}` |
| GET | `/api/v1/departments`, `/api/v1/departments/{id}` |
| GET | `/api/v1/job-titles`, `/api/v1/job-titles/{id}` |

### Filtering (Phase 3.5)

| Parameter | Type |
|-----------|------|
| `search` | string |
| `role_id` | ID |
| `department_id` | ID |
| `job_title_id` | ID |
| `status` | `active` \| `inactive` |
| `page` / `per_page` | pagination (`meta`) |

### Authorization (Phase 3.6)

| Role | Capability |
|------|------------|
| Administrator | Full User Management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

---

## 7. Out of Scope (Future Work)

- Department CRUD
- Job Title CRUD
- Multi-role users
- Multi-department users
- Teams / Branches / Organization Settings
- Invitation Emails / Force Password Change
- Permission Management / Advanced RBAC
- Vue UI for User Management

---

## 8. Acceptance Criteria

### Done (3.1–3.3)

- [x] Users schema matches ERD (soft deletes + FKs RESTRICT)
- [x] Departments and Job Titles migrated, seeded, soft-deletable
- [x] Morph map includes `department`, `job_title`
- [x] User CRUD + status endpoints with standard envelope
- [x] Inactive login rejected with dedicated `403`
- [x] `last_login_at` updated on successful login
- [x] `email_verified_at` retained
- [x] Feature tests green for 3.1–3.3
- [x] Docs synchronized for implemented behavior

### Remaining

- [ ] Lookup endpoints (3.4)
- [ ] User list filters + pagination (3.5)
- [ ] Coarse authorization enforced (3.6)

---

## 9. References

- [DOMAIN_MODEL.md](DOMAIN_MODEL.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [decisions/Organization-User-Management.md](../decisions/Organization-User-Management.md)
- [decisions/Database.md](../decisions/Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
