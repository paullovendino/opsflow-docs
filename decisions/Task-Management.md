# Decision: Task Management

## Status

Accepted — Milestone 5 complete (Phases 5.1–5.6)

## Context

OpsFlow completed Milestone 4 (Project Management). The next product milestone is Task Management: persist tasks under projects, support single assignment, expose CRUD/status/assignment APIs, list queries, and coarse authorization — without inventing Remarks, Activity Logs, dashboard, or frontend UI yet.

This ADR records approved Milestone 5 architecture so implementation can follow documentation without ad-hoc invention.

Companion specification: [docs/MILESTONE_5_TASK_MANAGEMENT.md](../docs/MILESTONE_5_TASK_MANAGEMENT.md)

## Decision

### Task model

- A **Task** is a unit of assignable work within a **Project**.
- Stored in table **`tasks`** with: `project_id`, `title`, optional `description`, `status`, `priority`, optional `due_date`, optional `assigned_to`, `created_by`, timestamps, **soft deletes**.
- `project_id` → `projects.id` with **ON DELETE RESTRICT** (required; not changeable after create in Milestone 5).
- `created_by` → `users.id` with **ON DELETE RESTRICT**; set from the authenticated user on create; **not** transferable.
- `assigned_to` → `users.id` with **ON DELETE RESTRICT**; **nullable** (unassigned allowed); **single assignee only**.

### Task status

Represented by `App\Enums\TaskStatus` (snake_case stored values):

| Value | Meaning |
|-------|---------|
| `todo` | To Do |
| `in_progress` | In Progress |
| `in_review` | In Review |
| `blocked` | Blocked |
| `completed` | Completed |
| `cancelled` | Cancelled |

Default on create: `todo` (status is **not** accepted on create/update bodies). Status-only updates use `PATCH /api/v1/tasks/{task}/status`.

Milestone 5 does **not** enforce a restricted status transition graph — any `TaskStatus` value may be applied via the status endpoint (consistent with Project status freedom).

### Task priority

Represented by `App\Enums\TaskPriority`:

| Value | Meaning |
|-------|---------|
| `low` | Low |
| `medium` | Medium |
| `high` | High |
| `urgent` | Urgent |

Default on create when omitted: `medium`. Priority may be set on create and update.

### Assignment

- Optional single user via `assigned_to`
- When non-null, assignee must be **active**, not soft-deleted, and either the **project owner** or a **project member**
- Create may include optional `assigned_to`
- General update must **not** change assignment — use `PATCH /api/v1/tasks/{task}/assignment` with `{ "assigned_to": <id|null> }`
- Multiple assignees, invitation workflows, and assignee roles are out of scope

### Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| 5.1 | Domain foundation (schema, model, enums, morph alias, relations, factory, tests) | ✅ Implemented |
| 5.2 | Task CRUD | ✅ Implemented |
| 5.3 | Assignment (`PATCH .../assignment`) | ✅ Implemented |
| 5.4 | Search / filter / sort / pagination | ✅ Implemented |
| 5.5 | Coarse `TaskPolicy` authorization | ✅ Implemented |
| 5.6 | Status workflow (`PATCH .../status`) | ✅ Implemented |

### API paths (approved design)

- Tasks: `GET/POST /api/v1/tasks`, `GET/PUT/DELETE /api/v1/tasks/{task}`
- Status: `PATCH /api/v1/tasks/{task}/status`
- Assignment: `PATCH /api/v1/tasks/{task}/assignment`

Primary collection is top-level `/api/v1/tasks` (filterable by `project_id`). Nested `/projects/{project}/tasks` is not required for Milestone 5.

### List query (Phase 5.4)

- Search: `title`, `description` (case-insensitive)
- Filters: `status`, `priority`, `project_id`, `assigned_to`, `created_by`
- Sort: `title`, `status`, `priority`, `due_date`, `created_at` (default `created_at` / `desc`)
- Pagination: `page` / `per_page` (default 15, max 100 clamped)
- Pattern: `TaskQuery` / `IndexTasksRequest` mirroring `ProjectQuery` / `IndexProjectsRequest`

### Coarse authorization (Phase 5.5)

| Role | Task Management |
|------|-----------------|
| Administrator | Full access to all tasks (CRUD, status, assignment) |
| Project Manager | Full access to all tasks (CRUD, status, assignment) |
| Employee | List/view tasks in projects they own or belong to as members; may `updateStatus` **only** when assigned to self; no create/update/delete/assignment |

Enforced via `TaskPolicy` + controller `$this->authorize()` + Employee list scoping in `TaskQuery`. Unauthorized → `403` API envelope.

Task routes require `auth:sanctum` and `TaskPolicy` checks (Phase 5.5).

### Layering

Mirror Project / User Management:

- Thin `TaskController`
- Form Requests for validation
- `TaskService` for business logic
- `TaskQuery` for list concerns
- `TaskPolicy` for authorization
- `TaskResource` for responses (never raw models)
- PHP Enums for status and priority (no magic strings)

### Explicitly out of scope for Milestone 5

Multiple assignees, attachments, checklists, labels, time tracking, dependencies/subtasks, activity logs, remarks/comments, notifications, recurring tasks, project transfer of tasks, restricted transition state machines, Vue Task UI, dashboard/report aggregates.

## Consequences

- Morph map must include `task` → `App\Models\Task` (Phase 5.1)
- Hard-deleting a referenced Project or User is blocked while tasks reference them (**RESTRICT**)
- Soft-deleted tasks remain excluded from default listings
- Assignee eligibility depends on Project ownership/membership rules from Milestone 4
- `DATABASE_DESIGN.md`, `DOMAIN_MODEL.md`, `API_SPECIFICATION.md`, Database ADR, HANDOFF, ROADMAP, and TESTING must be synchronized when this design is accepted and as phases complete
- Do not invent beyond this ADR / Milestone 5 specification during implementation

## References

- [docs/DOMAIN_MODEL.md](../docs/DOMAIN_MODEL.md)
- [docs/MILESTONE_5_TASK_MANAGEMENT.md](../docs/MILESTONE_5_TASK_MANAGEMENT.md)
- [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](../docs/MILESTONE_4_PROJECT_MANAGEMENT.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [decisions/Project-Management.md](Project-Management.md)
- [decisions/Database.md](Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
