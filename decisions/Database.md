# Decision: Database

## Status

Accepted — Milestone 3 complete (Phases 3.1–3.6); Milestone 4 Phases 4.1–4.3 (`projects` / `project_members` + member APIs)

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
- Columns: `name` (unique, human-readable), `code` (unique, stable identifier), `description` (nullable), timestamps, soft deletes
- Soft deletes supported
- Seeded with the **approved** lists in `DATABASE_DESIGN.md`; API is read-only (CRUD postponed)

Approved department seeds (`name` / `code`): Administration/`ADMIN`, Operations/`OPS`, Engineering/`ENG`, Human Resources/`HR`, Finance/`FIN`

Approved job title seeds (`name` / `code`): Administrator/`ADMIN`, Project Manager/`PM`, Software Engineer/`SE`, Operations Specialist/`OPS_SPEC`, Human Resources Specialist/`HR_SPEC`

### Users ERD (Milestone 3)

Approved columns:

- `role_id` (required)
- `department_id` (nullable)
- `job_title_id` (nullable)
- `first_name`, `middle_name` (nullable), `last_name` (nullable for legacy name-split compatibility)
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
| Phase 3 | Users ERD + `departments` + `job_titles` (**complete**) |
| Phase 4 | `projects` + `project_members` (**Phase 4.1 complete**; member APIs **Phase 4.3 complete**) |
| Phase 5+ | Tasks, Remarks, Activity Logs |

### Projects ERD (Milestone 4 — Phase 4.1 migrated; member APIs Phase 4.3)

`projects`: `name`, `description` (nullable), `status` (`ProjectStatus`), `start_date` / `due_date` (nullable), `created_by` → `users.id` (**RESTRICT**), timestamps, soft deletes.

`project_members`: `project_id`, `user_id`, `joined_at` (server-set), timestamps; unique (`project_id`, `user_id`); no member roles / invitations / pivot permissions; FKs **RESTRICT**. Duplicate membership via API → HTTP `409`. Only active, non-soft-deleted users may be added.

Status values: `planning`, `active`, `on_hold`, `completed`, `archived`.

Do not invent schema beyond the approved ERD without updating this decision and `DATABASE_DESIGN.md`.

### Morph Map

- Use `Relation::enforceMorphMap`
- Register aliases only for models that already exist
- Current aliases: `user`, `role`, `department`, `job_title`, `project`
- Future: `task`, `remark`, `activity_log`

## Consequences

- Local and production environments must provide PostgreSQL and `pdo_pgsql`
- SQLite is not the application database target
- Enum-backed role, user-status, and project-status values keep seeders and domain logic consistent
- Nullable department/job title supports onboarding without forcing incomplete org data
- RESTRICT prevents hard-deleting referenced departments, job titles, roles, project owners, or members while references exist
- Soft deletes on departments/job titles/projects/users do not remove rows; RESTRICT applies to hard deletes

## References

- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [decisions/Organization-User-Management.md](Organization-User-Management.md)
- [decisions/Project-Management.md](Project-Management.md)
