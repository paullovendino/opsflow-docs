# Decision: Database

## Status

Accepted

## Context

OpsFlow needs a production-ready relational database with support for JSONB, strong relational integrity, and clean polymorphic typing for future features.

Milestone 3 expands the people model beyond Roles alone: Departments and Job Titles become first-class reference tables, and Users align to the approved ERD.

## Decision

Use **PostgreSQL** as the only application database.

### Organizational persistence

- **Organization** is logical/single-tenant in v1.0 — no `organizations` table in Milestone 3
- Persist structure as `departments`, `job_titles`, `roles`, and `users`
- Role, Department, and Job Title are independent concepts

### Roles

- Roles are stored in a dedicated `roles` table
- Role `name` values are unique lowercase snake_case
- Seeded roles:
  - `administrator`
  - `project_manager`
  - `employee`
- Role names are represented in code with the `App\Enums\RoleName` enum
- Roles remain read-only in Milestone 3

### Departments & Job Titles (Milestone 3)

- Dedicated `departments` and `job_titles` tables
- Unique lowercase snake_case `name`
- Soft deletes supported
- Seeded with the **approved** lists in `DATABASE_DESIGN.md`; API is read-only (CRUD postponed)

Approved department seeds: `administration`, `operations`, `engineering`, `human_resources`, `finance`

Approved job title seeds: `administrator`, `project_manager`, `software_engineer`, `operations_specialist`, `human_resources_specialist`

### Users ERD (Milestone 3)

Approved columns:

- `role_id` (required)
- `department_id` (nullable)
- `job_title_id` (nullable)
- `first_name`, `middle_name` (nullable), `last_name`
- `email`, `email_verified_at` (kept for future compatibility; not enforced in M3)
- `password`
- `avatar` (nullable)
- `status` (enum-backed: `active`, `inactive`)
- `last_login_at` (nullable)
- timestamps + soft deletes

Legacy single `name` column is replaced during ERD alignment. Keep `email_verified_at`.

### Foreign keys (approved)

`users.role_id`, `users.department_id`, and `users.job_title_id` use **ON DELETE RESTRICT**. Do not use `SET NULL`.

### Schema Scope by Phase

| Phase | Schema |
|-------|--------|
| Phase 1 | `roles` (+ Sanctum tokens); default Laravel `users` |
| Phase 2 | Auth only — no ERD expansion |
| Phase 3 | Users ERD + `departments` + `job_titles` |
| Later | Projects, Tasks, Remarks, Activity Logs |

Do not invent schema beyond the approved ERD without updating this decision and `DATABASE_DESIGN.md`.

### Morph Map

- Use `Relation::enforceMorphMap`
- Register aliases only for existing models
- Current aliases: `user`, `role`
- Milestone 3 aliases when models exist: `department`, `job_title`
- Future: `project`, `task`, `remark`, `activity_log`

## Consequences

- Local and production environments must provide PostgreSQL and `pdo_pgsql`
- SQLite is not the application database target
- Enum-backed role and user-status values keep seeders and domain logic consistent
- Nullable department/job title supports onboarding without forcing incomplete org data
- RESTRICT prevents hard-deleting referenced departments, job titles, or roles while users exist
- Soft deletes on departments/job titles do not remove rows; RESTRICT applies to hard deletes

## References

- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [decisions/Organization-User-Management.md](Organization-User-Management.md)
