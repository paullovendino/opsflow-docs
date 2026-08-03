# Decision: Project Management

## Status

Accepted — Phases 4.1–4.5 implemented; Milestone 4 complete

## Context

OpsFlow completed Milestone 3 (Organization & User Management). The next product milestone is Project Management: persist projects, associate members, expose CRUD/status/member APIs, list queries, and coarse authorization — without inventing Tasks or frontend UI yet.

This ADR records approved Phase 4 architecture so implementation can follow documentation without ad-hoc invention.

## Decision

### Project model

- A **Project** is a body of work with name, optional description, status, optional start/due dates, and one **owner**.
- Owner is stored as `projects.created_by` → `users.id` with **ON DELETE RESTRICT**.
- Projects use **soft deletes**.
- `created_by` is set from the authenticated user on create and is **not** transferable in Phase 4.

### Project status

Represented by `App\Enums\ProjectStatus` (snake_case stored values):

| Value | Meaning |
|-------|---------|
| `planning` | Planning |
| `active` | Active |
| `on_hold` | On Hold |
| `completed` | Completed |
| `archived` | Archived |

Default on create: `planning` (status is **not** accepted on create/update bodies). Status-only updates use `PATCH /api/v1/projects/{project}/status`.

### Project members (Phase 4.3 — implemented)

- Many-to-many via pivot table **`project_members`**
- Columns: `id`, `project_id`, `user_id`, `joined_at`, `created_at`, `updated_at`
- Unique (`project_id`, `user_id`)
- **No** member roles, invitation workflow, or permissions on the pivot
- Owner is **not** auto-added to `project_members`; membership is explicit and independent of `created_by`
- `joined_at` is **server-generated only**; client-supplied `joined_at` is ignored
- Duplicate membership → HTTP **`409 Conflict`**
- Only **active**, non-soft-deleted users may be added (`422` otherwise)
- Employee access (Phase 4.5) = projects where the user is **owner OR member**

### Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| 4.1 | Domain foundation (schema, models, enum, morph alias, relations, factory, tests) | ✅ Implemented |
| 4.2 | Project CRUD + status patch | ✅ Implemented |
| 4.3 | Project Members APIs | ✅ Implemented |
| 4.4 | Search / filter / sort / pagination | ✅ Implemented |
| 4.5 | Coarse `ProjectPolicy` authorization | ✅ Implemented |

### API paths (implemented through 4.5)

- Projects: `GET/POST /api/v1/projects`, `GET/PUT/DELETE /api/v1/projects/{project}`, `PATCH /api/v1/projects/{project}/status`
- Members: `GET/POST /api/v1/projects/{project}/members`, `DELETE /api/v1/projects/{project}/members/{user}`

### Coarse authorization (Phase 4.5 — implemented)

| Role | Project Management |
|------|--------------------|
| Administrator | Full access to all projects (CRUD, status, members) |
| Project Manager | Full access to all projects (CRUD, status, members) |
| Employee | List/view only for projects they own or belong to as members |

Enforced via `ProjectPolicy` + controller `$this->authorize()` + Employee list scoping in `ProjectQuery`. Unauthorized → `403` API envelope.

### Explicitly out of scope for Phase 4

Tasks (see Milestone 5 ADR), Remarks, Activity Logs, Dashboard, Reports, Vue UI, ownership transfer, member roles/invitations, advanced RBAC.

## Consequences

- Morph map includes `project` (Phase 4.1)
- Hard-deleting a referenced User is blocked while they own projects or appear in `project_members` (RESTRICT)
- Soft-deleted projects remain excluded from default listings
- Layering mirrors User Management: thin controller, Form Requests, Service, Query object (4.4), Policy (4.5), API Resources
- Database ADR / `DATABASE_DESIGN.md` must stay aligned with this decision

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](../docs/MILESTONE_4_PROJECT_MANAGEMENT.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [decisions/Database.md](Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
