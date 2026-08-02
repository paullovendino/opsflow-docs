# Milestone 3 — Organization & User Management

**Status:** Design approved — awaiting implementation approval  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-02

> This document is the **implementation specification** for Phase 3.  
> Final architecture decisions below are approved. Wait for explicit go-ahead before writing code.  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)

---

## 1. Goal

Establish OpsFlow’s organizational people model and deliver User Management on the API:

- Expand Users to the approved ERD
- Introduce Departments and Job Titles (seeded, read-only)
- Keep Roles seeded and read-only
- Expose User CRUD + status changes
- Apply coarse role-based authorization for User Management only

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

## 3. In Scope

### Schema

- Align `users` with the approved ERD (see [DATABASE_DESIGN.md](../DATABASE_DESIGN.md))
- Add `departments` table (soft deletes)
- Add `job_titles` table (soft deletes)
- Keep `roles` as seeded read-only reference data
- Register morph aliases when models are introduced: `department`, `job_title`

### Seed data (approved)

**Departments** (`name` / `code`):

| name | code |
|------|------|
| Administration | `ADMIN` |
| Operations | `OPS` |
| Engineering | `ENG` |
| Human Resources | `HR` |
| Finance | `FIN` |

**Job Titles** (`name` / `code`):

| name | code |
|------|------|
| Administrator | `ADMIN` |
| Project Manager | `PM` |
| Software Engineer | `SE` |
| Operations Specialist | `OPS_SPEC` |
| Human Resources Specialist | `HR_SPEC` |

**Roles** (unchanged; snake_case `name`): `administrator`, `project_manager`, `employee`

Full schema: [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)

### Implementation phasing

| Phase | Scope |
|-------|-------|
| **3.1 Organization Foundation** | `departments` / `job_titles` tables (incl. `code`), models, seeders, morph aliases; `Department`/`JobTitle` → `users` relation methods only |
| **3.2 User Management** | Users ERD + FKs; `User::department()` / `User::jobTitle()`; CRUD/APIs/auth status rules |

### API (all Planned until implemented)

- Users: list, show, create, update, soft-delete, status patch
- Roles: list, show (all authenticated users)
- Departments: list, show (all authenticated users)
- Job Titles: list, show (all authenticated users)

### Filtering (Users list — approved)

| Parameter | Type |
|-----------|------|
| `search` | string |
| `role_id` | ID |
| `department_id` | ID |
| `job_title_id` | ID |
| `status` | `active` \| `inactive` |
| `page` / `per_page` | pagination (`meta`) |

Use IDs for FK filters — not names.

### Authorization (coarse only)

| Role | Capability |
|------|------------|
| Administrator | Full User Management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

Lookup endpoints (roles / departments / job titles): **all authenticated users**.

### Auth / account status (approved)

- Inactive users (`status = inactive`) **must not** log in
- Response: HTTP `403` with dedicated message (e.g. `Account is inactive.`)
- Do not treat inactive accounts as generic invalid credentials (`401`)

### Quality bar for completion

- Passing feature tests for User Management + lookup endpoints
- Updated documentation reflecting implemented behavior
- Git commit (and push when requested)

---

## 4. Out of Scope (Future Work)

Documented deferred items:

- Department CRUD
- Job Title CRUD
- Multi-role users
- Multi-department users
- Teams
- Branches
- Organization Settings
- Invitation Emails
- Force Password Change
- Permission Management
- Advanced RBAC
- Projects, Tasks, Remarks, Activity Logs (later phases)
- Vue UI for User Management (frontend milestone; API-first here)

---

## 5. Business Rules Summary

### Departments

- Read-only in this milestone
- Seeded by default
- Soft deletes supported at persistence layer
- CRUD postponed

### Job Titles

- Read-only in this milestone
- Seeded by default
- Soft deletes supported at persistence layer
- CRUD postponed

### Roles

- Continue using Administrator, Project Manager, Employee
- Remain read-only (list/show)
- Drive coarse authorization only

### Users

Required / notable fields:

| Field | Notes |
|-------|-------|
| `role_id` | Required FK → roles |
| `department_id` | Nullable FK → departments |
| `job_title_id` | Nullable FK → job_titles |
| `first_name` | Required |
| `middle_name` | Nullable |
| `last_name` | Required |
| `email` | Unique; login identifier |
| `email_verified_at` | Kept for future compatibility; not enforced in M3 |
| `password` | Hashed; never returned |
| `avatar` | Nullable |
| `status` | Enum-backed (`active` / `inactive`) |
| `last_login_at` | Nullable; set on successful login |
| timestamps | `created_at`, `updated_at` |
| soft deletes | `deleted_at` |

Replace legacy single `name` column with structured name fields as part of ERD alignment. Keep `email_verified_at`.

FK delete behavior: **RESTRICT** for `role_id`, `department_id`, and `job_title_id` while users exist. Do not use `SET NULL`.

---

## 6. Relationships

| From | Type | To |
|------|------|-----|
| Department | hasMany | Users |
| JobTitle | hasMany | Users |
| Role | hasMany | Users |
| User | belongsTo | Department (optional) |
| User | belongsTo | JobTitle (optional) |
| User | belongsTo | Role (required) |

---

## 7. API Surface (Planned)

All endpoints below are **Planned** — not implemented yet.

### Users

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/users` | Directory; filterable; paginated |
| GET | `/api/v1/users/{id}` | Show user |
| POST | `/api/v1/users` | Create user |
| PUT | `/api/v1/users/{id}` | Update user |
| DELETE | `/api/v1/users/{id}` | Soft delete |
| PATCH | `/api/v1/users/{id}/status` | Activate / deactivate |

### Roles

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/roles` | List roles |
| GET | `/api/v1/roles/{id}` | Show role |

### Departments

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/departments` | List departments |
| GET | `/api/v1/departments/{id}` | Show department |

### Job Titles

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/v1/job-titles` | List job titles |
| GET | `/api/v1/job-titles/{id}` | Show job title |

Full request/response contracts: [API_SPECIFICATION.md](../API_SPECIFICATION.md).

---

## 8. Implementation Layering (when coding begins)

Follow existing auth patterns:

| Concern | Location |
|---------|----------|
| Controllers | `App\Http\Controllers\Api\V1\...` |
| Form Requests | `App\Http\Requests\Api\V1\...` |
| API Resources | `App\Http\Resources\Api\V1\...` |
| Services | `App\Services\...` (e.g. User management service) |
| Enums | `App\Enums\...` (`RoleName`, `UserStatus`, …) |
| Policies (coarse) | `App\Policies\UserPolicy` (or equivalent) |
| Seeders | Departments, Job Titles, Roles (existing) |

Auth responses (`/auth/login`, `/auth/me`) must be updated to the expanded `UserResource` shape when the users schema changes.

---

## 9. Acceptance Criteria (Definition of Done)

- [ ] Users schema matches ERD (including soft deletes and FKs)
- [ ] Departments and Job Titles migrated, seeded, soft-deletable
- [ ] Morph map includes aliases for introduced models
- [ ] Planned endpoints implemented with standard envelope
- [ ] User list supports approved ID filters + `search` + `status` + pagination
- [ ] Coarse authorization enforced per role matrix
- [ ] Lookups available to all authenticated users
- [ ] Inactive login rejected with dedicated `403`
- [ ] `last_login_at` updated on successful login
- [ ] FK constraints use RESTRICT for role/department/job title
- [ ] `email_verified_at` retained on users
- [ ] Feature tests green
- [ ] Docs updated from Planned → Implemented where accurate
- [ ] Focused git commit for the milestone

---

## 10. References

- [DOMAIN_MODEL.md](DOMAIN_MODEL.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [decisions/Organization-User-Management.md](../decisions/Organization-User-Management.md)
- [decisions/Database.md](../decisions/Database.md)
- [ROADMAP.md](../ROADMAP.md)
