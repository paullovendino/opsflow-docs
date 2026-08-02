# Decision: Organization & User Management

## Status

Accepted — Phases 3.1–3.3 implemented; 3.4–3.6 remaining

## Context

OpsFlow completed API foundation and authentication with a default Laravel `users` table and a seeded `roles` table. The product needed a clear organizational people model before User Management.

Phase 3 was expanded from “User Management” alone to **Organization & User Management**, introducing Departments and Job Titles as independent concepts alongside Roles and Users.

## Decision

### Organizational model

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

- **Organization** is a logical single-tenant workplace context in v1.0 (no `organizations` table in Milestone 3).
- **Role**, **Department**, and **Job Title** are independent.
- A **User** has exactly one Role (required), and at most one Department and one Job Title (both nullable).

### Approved seed lists

**Departments** (`name` / `code`): Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`

**Job Titles** (`name` / `code`): Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`

**Roles** (snake_case `name`): `administrator`, `project_manager`, `employee`

### Implementation status

| Area | Status |
|------|--------|
| Departments / Job Titles foundation (3.1) | Implemented |
| Users ERD + auth compatibility (3.2) | Implemented |
| User CRUD APIs (3.3) | Implemented |
| Lookup APIs (3.4) | Pending |
| Search / filters / pagination (3.5) | Pending |
| Coarse authorization (3.6) | Pending |

### Users ERD

Users store: `role_id`, `department_id` (nullable), `job_title_id` (nullable), `first_name`, `middle_name` (nullable), `last_name` (nullable for legacy splits), `email`, `email_verified_at` (kept), `password`, `avatar` (nullable), `status`, `last_login_at` (nullable), timestamps, soft deletes.

FK delete: **RESTRICT** for role / department / job title.

### Inactive account login

Inactive users (`status = inactive`) must not authenticate → HTTP `403` (`Account is inactive.`). **Implemented** in Phase 3.2.

### API paths

**Implemented (3.3):**

- Users: `GET/POST /api/v1/users`, `GET/PUT/DELETE /api/v1/users/{user}`, `PATCH /api/v1/users/{user}/status`

**Pending (3.4):**

- Roles / Departments / Job Titles list + show

### Filtering (Phase 3.5)

User list filters: `search`, `role_id`, `department_id`, `job_title_id`, `status`, plus pagination.

### Coarse authorization (Phase 3.6)

| Role | User Management |
|------|-----------------|
| Administrator | Full user management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

## Consequences

- Auth `UserResource` uses structured profile + nested Role/Department/JobTitle resources
- User Management currently authenticates only (`auth:sanctum`); role policies arrive in 3.6
- Hard-deleting referenced departments/job titles/roles is blocked while users reference them
- Advanced RBAC, invitations, multi-role/multi-department, and org settings remain future work

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](../docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [decisions/Database.md](Database.md)
- [decisions/Authentication.md](Authentication.md)
