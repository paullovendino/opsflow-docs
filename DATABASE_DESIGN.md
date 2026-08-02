# Database Design

Physical schema companion to [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md).

**Milestone 3 status:** Phases 3.1–3.3 implemented (org foundation, users ERD, User CRUD APIs). Remaining: 3.4 lookups, 3.5 filters/pagination, 3.6 authorization.

---

## Organizational Model (persistence)

Logical organization (single-tenant v1.0) persists as:

| Table | Milestone 3 |
|-------|-------------|
| `departments` | New — seeded, soft deletes |
| `job_titles` | New — seeded, soft deletes |
| `roles` | Existing — seeded, read-only usage |
| `users` | Align to ERD — soft deletes |

No `organizations` table in Milestone 3.

---

## Tables

### Departments

| Column | Notes |
|--------|-------|
| id | PK |
| name | unique, human-readable display name |
| code | unique, stable identifier (uppercase) |
| description | nullable text |
| created_at | timestamp |
| updated_at | timestamp |
| deleted_at | soft deletes |

**Milestone 3:** seeded by default; read-only via API; CRUD postponed.

**Convention:** `name` is human-readable; `code` is the stable machine identifier.

**Approved seed list:**

| name | code | description |
|------|------|-------------|
| Administration | `ADMIN` | Company administration and leadership |
| Operations | `OPS` | Day-to-day operations |
| Engineering | `ENG` | Product and engineering |
| Human Resources | `HR` | People and talent |
| Finance | `FIN` | Finance and accounting |

---

### Job Titles

| Column | Notes |
|--------|-------|
| id | PK |
| name | unique, human-readable display name |
| code | unique, stable identifier (uppercase) |
| description | nullable text |
| created_at | timestamp |
| updated_at | timestamp |
| deleted_at | soft deletes |

**Milestone 3:** seeded by default; read-only via API; CRUD postponed.

**Convention:** `name` is human-readable; `code` is the stable machine identifier.

**Approved seed list:**

| name | code | description |
|------|------|-------------|
| Administrator | `ADMIN` | Company / system administrator position |
| Project Manager | `PM` | Delivers projects and coordinates teams |
| Software Engineer | `SE` | Builds and maintains software |
| Operations Specialist | `OPS_SPEC` | Supports operational processes |
| Human Resources Specialist | `HR_SPEC` | Supports HR processes |

> Job title names/codes may coincide with role language linguistically; they are **not** the same records or permissions.

---

### Roles

| Column | Notes |
|--------|-------|
| id | PK |
| name | unique, lowercase snake_case |
| description | text |
| created_at | timestamp |
| updated_at | timestamp |

Seeded roles:

| name | description |
|------|-------------|
| `administrator` | Full system access |
| `project_manager` | Manage projects and tasks |
| `employee` | Assigned work and updates |

**Milestone 3:** remain read-only. Soft deletes are not required for roles in this milestone.

Represented in code with `App\Enums\RoleName`.

---

### Users

| Column | Notes |
|--------|-------|
| id | PK |
| role_id | required FK → `roles.id` |
| department_id | nullable FK → `departments.id` |
| job_title_id | nullable FK → `job_titles.id` |
| first_name | string |
| middle_name | nullable string |
| last_name | nullable string (nullable to preserve best-effort legacy `name` splits; CRUD may require later) |
| email | unique string (login identifier) |
| email_verified_at | nullable timestamp — **kept** for future compatibility (not enforced in Milestone 3) |
| password | hashed string |
| avatar | nullable string (path/URL) |
| status | enum-backed string (`UserStatus`: `active`, `inactive`) |
| last_login_at | nullable timestamp |
| created_at | timestamp |
| updated_at | timestamp |
| deleted_at | soft deletes |

**Status values** (`UserStatus`):

| value | Meaning |
|-------|---------|
| `active` | Account may authenticate and use the system |
| `inactive` | Account deactivated; login blocked with HTTP `403` |

**ERD alignment notes:**

- Replaces legacy Laravel `name` with structured name fields (Phase 3.2 migration)
- Existing `name` values are split best-effort; single-token names keep `last_name` null
- Existing users receive default role `employee` during migration when backfilling `role_id`
- **Keep** Laravel `email_verified_at` for future compatibility (verification not required in Milestone 3)
- `role_id` required; `department_id` / `job_title_id` nullable for onboarding flexibility
- Indexes: FKs + `status`; unique `email`

**Phase status:** Phase 3.2 schema/model/auth + Phase 3.3 User CRUD APIs implemented. Lookups/filters/authz remain 3.4–3.6.

---

### Projects

> Planned — later phase

- id
- name
- description
- status
- start_date
- due_date
- created_by
- created_at
- updated_at

---

### Tasks

> Planned — later phase

- id
- project_id
- assigned_to
- title
- description
- priority
- status
- due_date
- created_by
- created_at
- updated_at

---

### Remarks

> Planned — later phase (schema TBD)

Conceptual only in [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md). Physical columns decided when the Remarks milestone begins.

---

### Activity Logs

> Planned — later phase

- id
- user_id
- action
- module
- description
- ip_address
- created_at

---

## Relationships

```text
Department 1 ──► * Users
JobTitle   1 ──► * Users
Role       1 ──► * Users

User * ──► 1 Role          (required)
User * ──► 0..1 Department (nullable)
User * ──► 0..1 JobTitle   (nullable)

User 1 ──► * Projects      (planned)
Project 1 ──► * Tasks      (planned)
User 1 ──► * Tasks         (planned; assignment)
User 1 ──► * Activity Logs (planned)
```

Eloquent expectations (Milestone 3):

| Model | Relations |
|-------|-----------|
| `Department` | `hasMany(User::class)` |
| `JobTitle` | `hasMany(User::class)` |
| `Role` | `hasMany(User::class)` |
| `User` | `belongsTo(Role)`, `belongsTo(Department)`, `belongsTo(JobTitle)` |

---

## Morph Map

Use Laravel `Relation::enforceMorphMap`.

### Registered today (existing models only)

- `user` → `App\Models\User`
- `role` → `App\Models\Role`
- `department` → `App\Models\Department` (Phase 3.1)
- `job_title` → `App\Models\JobTitle` (Phase 3.1)

### Future aliases (later phases)

- `project` → `App\Models\Project`
- `task` → `App\Models\Task`
- `remark` → `App\Models\Remark`
- `activity_log` → `App\Models\ActivityLog`

---

## Foreign Key / Delete Behavior (approved)

| FK | On delete |
|----|-----------|
| `users.role_id` → `roles.id` | **RESTRICT** while users reference the role |
| `users.department_id` → `departments.id` | **RESTRICT** while users reference the department |
| `users.job_title_id` → `job_titles.id` | **RESTRICT** while users reference the job title |

Do **not** use `SET NULL` on these foreign keys.

Notes:

- Nullable `department_id` / `job_title_id` still allows users without a department or job title
- Hard-deleting a referenced Department, Job Title, or Role must fail while users exist
- Soft deletes on Departments / Job Titles do not remove the row; RESTRICT applies to hard deletes
- List endpoints typically exclude soft-deleted reference rows unless explicitly requested later

---

## Related Decisions

- [decisions/Database.md](decisions/Database.md)
- [decisions/Organization-User-Management.md](decisions/Organization-User-Management.md)
- [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
