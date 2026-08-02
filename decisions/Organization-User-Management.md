# Decision: Organization & User Management

## Status

Accepted (Documentation) — Design decisions finalized; implementation not started

## Context

OpsFlow completed API foundation and authentication with a default Laravel `users` table and a seeded `roles` table. The product now needs a clear organizational people model before User Management is implemented.

Earlier docs treated Phase 3 as “User Management” only. The approved direction expands that milestone to **Organization & User Management**, introducing Departments and Job Titles as independent concepts alongside Roles and Users.

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

**Departments:** `administration`, `operations`, `engineering`, `human_resources`, `finance`

**Job Titles:** `administrator`, `project_manager`, `software_engineer`, `operations_specialist`, `human_resources_specialist`

**Roles (existing):** `administrator`, `project_manager`, `employee`

### Milestone 3 scope

| Area | Decision |
|------|----------|
| Departments | Seeded (approved list); read-only API (list/show); soft deletes; CRUD postponed |
| Job Titles | Seeded (approved list); read-only API (list/show); soft deletes; CRUD postponed |
| Roles | Keep seeded roles; read-only API |
| Users | Full ERD alignment; CRUD + status patch; soft deletes |
| Authorization | Coarse role checks only (not advanced RBAC) |
| Lookups | All authenticated users may read roles, departments, job titles |
| Organization settings / teams / branches | Out of scope |

### Coarse authorization

| Role | User Management |
|------|-----------------|
| Administrator | Full user management |
| Project Manager | Read-only user directory |
| Employee | View own profile |

### Users ERD (approved)

Users store: `role_id`, `department_id` (nullable), `job_title_id` (nullable), `first_name`, `middle_name` (nullable), `last_name`, `email`, `email_verified_at` (kept for future compatibility), `password`, `avatar` (nullable), `status`, `last_login_at` (nullable), timestamps, soft deletes.

Legacy single `name` column is replaced by structured name fields during ERD alignment. Keep `email_verified_at`.

### Foreign keys (approved)

`users.role_id`, `users.department_id`, and `users.job_title_id` use **ON DELETE RESTRICT** while referencing users exist. Do **not** use `SET NULL`.

### Inactive account login (approved)

Inactive users (`status = inactive`) must not authenticate. Login returns HTTP `403` with a dedicated inactive-account message (for example `Account is inactive.`), distinct from invalid credentials (`401`).

### API paths (Planned)

- Users: `/api/v1/users`, `/api/v1/users/{id}`, `/api/v1/users/{id}/status`
- Roles: `/api/v1/roles`, `/api/v1/roles/{id}`
- Departments: `/api/v1/departments`, `/api/v1/departments/{id}`
- Job Titles: `/api/v1/job-titles`, `/api/v1/job-titles/{id}`

### Filtering (approved)

User list filters:

| Parameter | Notes |
|-----------|-------|
| `search` | Name fields / email |
| `role_id` | ID only |
| `department_id` | ID only |
| `job_title_id` | ID only |
| `status` | `active` / `inactive` |
| pagination | `meta` page fields |

## Consequences

- Phase 3 documentation and roadmap refer to **Organization & User Management**
- `DATABASE_DESIGN.md`, `API_SPECIFICATION.md`, and `docs/DOMAIN_MODEL.md` are the specs for implementation
- Auth `UserResource` must be updated when the users schema changes
- Authentication must enforce inactive-account rejection (`403`)
- `User::role()` becomes usable only after `role_id` exists
- Morph aliases `department` and `job_title` are registered when those models are introduced
- Hard-deleting referenced departments/job titles/roles is blocked while users reference them
- Advanced RBAC, invitations, multi-role/multi-department, and org settings remain future work

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](../docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [decisions/Database.md](Database.md)
- [decisions/Authentication.md](Authentication.md)
