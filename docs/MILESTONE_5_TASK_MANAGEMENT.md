# Milestone 5 — Task Management

**Status:** ✅ Milestone 5 complete (Phases 5.1–5.6)  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-04

> Implementation specification for Milestone 5.  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)  
> ADR: [decisions/Task-Management.md](../decisions/Task-Management.md)  
> Prerequisite: Milestone 4 complete (Projects, members, queries, `ProjectPolicy`)

---

## 1. Goal

Introduce OpsFlow’s Task Management module on the API:

- Persist Tasks under a Project with status, priority, due date, soft deletes
- Support a single optional assignee (`assigned_to`)
- Expose Task CRUD + dedicated status and assignment endpoints
- Apply list search / filter / sort / pagination
- Enforce coarse role-based authorization via `TaskPolicy`
- Document status workflow via enum + status-only patch (no restricted transition graph in Milestone 5)

---

## 2. Domain summary

```text
User (creator via created_by)
    └── Project
            └── Task
                    ├── optional assignee → User (assigned_to)
                    └── creator → User (created_by)
```

- Each Task **belongs to exactly one Project** (`tasks.project_id`)
- Each Task has **one creator** (`tasks.created_by` → `users.id`)
- Each Task has **at most one assignee** (`tasks.assigned_to` → `users.id`, nullable)
- Assignment targets **Users**, not Departments or Job Titles
- Remarks, Activity Logs, subtasks, and dependencies are **out of scope**

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **5.1 Task Domain Foundation** | `tasks` table; `Task` model; `TaskStatus` / `TaskPriority` enums; relations; morph alias `task`; factory; foundation tests | ✅ Implemented |
| **5.2 Task CRUD** | `TaskController` / `TaskService` / Form Requests / `TaskResource` / CRUD | ✅ Implemented |
| **5.3 Task Assignment** | `PATCH /api/v1/tasks/{task}/assignment`; assignment rules + validation | ✅ Implemented |
| **5.4 Task Queries** | Task list search, filters, sorting, pagination `meta` | ✅ Implemented |
| **5.5 Task Authorization** | Coarse role matrix via `TaskPolicy` + Employee visibility scoping | ✅ Implemented |
| **5.6 Task Status Workflow** | `PATCH /api/v1/tasks/{task}/status`; status not on create/update bodies | ✅ Implemented |

---

## 4. Approved schema decisions

### Tasks

Table: `tasks`

| Column | Notes |
|--------|-------|
| id | PK |
| project_id | required FK → `projects.id` (**RESTRICT** on hard delete) |
| title | required string |
| description | nullable text |
| status | enum-backed string (`TaskStatus`) |
| priority | enum-backed string (`TaskPriority`) |
| due_date | nullable date |
| assigned_to | nullable FK → `users.id` (**RESTRICT** on hard delete) |
| created_by | required FK → `users.id` (**RESTRICT** on hard delete) |
| created_at / updated_at | timestamps |
| deleted_at | **soft deletes** |

**Indexes (approved):** FKs (`project_id`, `assigned_to`, `created_by`) + `status` + `priority`; consider `title` later if search performance requires it.

### Status values (`App\Enums\TaskStatus`)

| Enum case / stored value | Display |
|--------------------------|---------|
| `todo` | To Do |
| `in_progress` | In Progress |
| `in_review` | In Review |
| `blocked` | Blocked |
| `completed` | Completed |
| `cancelled` | Cancelled |

**Defaults / behaviors:**

- On create: `status` always `todo` (client-supplied `status` ignored / not accepted)
- Status changes **only** via `PATCH /api/v1/tasks/{task}/status`
- Milestone 5 does **not** enforce a restricted transition graph — any `TaskStatus` value may be set via the status endpoint (same freedom as `ProjectStatus`)
- Soft delete only (no hard delete API)
- Soft-deleted tasks are excluded from default lists
- Soft-deleted projects keep task rows; default task lists exclude soft-deleted tasks (and typically operate on non-soft-deleted projects via FK / route model binding)

### Priority values (`App\Enums\TaskPriority`)

| Enum case / stored value | Display |
|--------------------------|---------|
| `low` | Low |
| `medium` | Medium |
| `high` | High |
| `urgent` | Urgent |

**Defaults / behaviors:**

- On create: when `priority` omitted → `medium`
- `priority` may be set on create and update (unlike `status`)

### Assignment rules

- **Single assignee only** (`assigned_to`); multiple assignees deferred
- `assigned_to` may be `null` (unassigned)
- When set, assignee **must**:
  1. Exist and not be soft-deleted
  2. Have `status = active`
  3. Be the **project owner** (`projects.created_by`) **or** a **project member** (`project_members`) for the task’s `project_id`
- Otherwise → HTTP `422` with standard validation envelope
- Client may set `assigned_to` on **create** (optional)
- Client may **not** set `assigned_to` on general **update** — use assignment endpoint
- Clearing assignee: assignment endpoint with `"assigned_to": null`
- Changing `project_id` after create is **not** allowed in Milestone 5

### Creator / ownership fields

- `created_by` = authenticated user on create; **not** transferable; **not** accepted on update
- `project_id` required on create; **not** accepted on update

Full physical schema companion (to be synced when this design is approved): [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)

---

## 5. Phase 5.1 — Task Domain Foundation

**Status:** ✅ Implemented

Deliverables:

- [x] Migration for `tasks`
- [x] Model: `App\Models\Task`
- [x] Enums: `App\Enums\TaskStatus`, `App\Enums\TaskPriority`
- [x] Eloquent relations:
  - `Task::project()` → `belongsTo(Project::class)`
  - `Task::assignee()` → `belongsTo(User::class, 'assigned_to')`
  - `Task::creator()` / `createdBy()` → `belongsTo(User::class, 'created_by')`
  - `Project::tasks()` → `hasMany(Task::class)`
  - `User::createdTasks()` → `hasMany(Task::class, 'created_by')`
  - `User::assignedTasks()` → `hasMany(Task::class, 'assigned_to')`
- [x] Morph map alias: `task` → `App\Models\Task`
- [x] `TaskFactory`
- [x] Feature tests: `tests/Feature/Task/TaskDomainFoundationTest.php`

**Out of scope for 5.1:** HTTP CRUD, status/assignment APIs, queries, policies. (No task seeders — transactional data only.)

---

## 6. Phase 5.2 — Task CRUD

**Status:** ✅ Implemented

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/tasks` | List tasks (search/filters/sort/pagination — Phase 5.4) |
| GET | `/api/v1/tasks/{task}` | Show task |
| POST | `/api/v1/tasks` | Create task |
| PUT | `/api/v1/tasks/{task}` | Update task |
| DELETE | `/api/v1/tasks/{task}` | Soft delete |

Status and assignment mutations are **not** part of PUT — see Phases 5.6 and 5.3.

### Layering

| Concern | Class |
|---------|-------|
| Controller | `App\Http\Controllers\Api\V1\TaskController` (thin) |
| Service | `App\Services\Tasks\TaskService` |
| List query | `App\Queries\Tasks\TaskQuery` (✅ Phase 5.4) |
| Authorization | `App\Policies\TaskPolicy` (✅ Phase 5.5) |
| Store validation | `StoreTaskRequest` |
| Update validation | `UpdateTaskRequest` |
| Assignment validation | `UpdateTaskAssignmentRequest` (✅ Phase 5.3) |
| Index validation | `IndexTasksRequest` (✅ Phase 5.4) |
| Status validation | `UpdateTaskStatusRequest` (✅ Phase 5.6) |
| Resources | `TaskResource` (+ nested `project` summary, `assignee`, `creator` via `whenLoaded()`) |
| Enums | `TaskStatus`, `TaskPriority` |

### Create / update fields

| Field | Create | Update |
|-------|--------|--------|
| `project_id` | required; must exist, not soft-deleted | **not accepted** |
| `title` | required | required |
| `description` | optional | optional |
| `priority` | optional; default `medium` | optional; when present must be `TaskPriority` |
| `due_date` | optional date | optional date |
| `assigned_to` | optional; must satisfy assignment rules when present | **not accepted** — use assignment endpoint |
| `status` | **not accepted** — always `todo` | **not accepted** — use status patch |
| `created_by` | set server-side from auth user | **not accepted** |

### Validation rules (Form Requests)

**`StoreTaskRequest`:**

| Field | Rules |
|-------|-------|
| `project_id` | required, integer, `exists:projects,id` (non-soft-deleted) |
| `title` | required, string, max:255 |
| `description` | sometimes, nullable, string |
| `priority` | sometimes, nullable, `Rule::enum(TaskPriority::class)` |
| `due_date` | sometimes, nullable, date |
| `assigned_to` | sometimes, nullable, integer, `exists:users,id`; custom/service check: active + project owner-or-member |

**`UpdateTaskRequest`:**

| Field | Rules |
|-------|-------|
| `title` | required, string, max:255 |
| `description` | sometimes, nullable, string |
| `priority` | required, `Rule::enum(TaskPriority::class)` |
| `due_date` | sometimes, nullable, date |

Reject unknown fields that would change `status`, `assigned_to`, `project_id`, or `created_by` (do not silently apply them).

### Behaviors

- Soft delete only
- Eager-load `project`, `assignee`, `creator` for show/create/update responses as needed; nest via `TaskResource` + `whenLoaded()`
- Standard response envelope; never return raw models
- Match Project / User Management patterns (`ProjectController` / `ProjectService` / status-style patches)
- Feature tests: `tests/Feature/Task/TaskManagementApiTest.php` ✅

### Deliverables

- [x] `TaskController` (thin)
- [x] `TaskService`
- [x] `StoreTaskRequest` / `UpdateTaskRequest`
- [x] `TaskResource`
- [x] CRUD routes under `auth:sanctum`
- [x] Feature tests green

### Task resource shape (approved)

```json
{
  "id": 1,
  "title": "Draft API contract",
  "description": "Document task endpoints",
  "status": "todo",
  "priority": "medium",
  "due_date": "2026-08-15",
  "project": {
    "id": 1,
    "name": "OpsFlow Launch"
  },
  "assignee": {
    "id": 3,
    "first_name": "Sam",
    "middle_name": null,
    "last_name": "Lee",
    "full_name": "Sam Lee",
    "email": "sam@example.com"
  },
  "creator": {
    "id": 1,
    "first_name": "Jane",
    "middle_name": null,
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane@example.com"
  },
  "created_at": "2026-08-03T00:00:00.000000Z",
  "updated_at": "2026-08-03T00:00:00.000000Z"
}
```

`assignee` may be `null` when unassigned. Nested objects appear when relations are loaded.

---

## 7. Phase 5.3 — Task Assignment

**Status:** ✅ Implemented

### Route (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| PATCH | `/api/v1/tasks/{task}/assignment` | Set or clear assignee |

### Request body

```json
{
  "assigned_to": 3
}
```

Clear assignee:

```json
{
  "assigned_to": null
}
```

### Validation (`UpdateTaskAssignmentRequest`)

| Field | Rules |
|-------|--------|
| `assigned_to` | present (nullable), integer when non-null, `exists:users,id`; when non-null must be active, not soft-deleted, and project owner **or** member |

### Behaviors

- Replaces previous assignee (single assignee model)
- Does not change `created_by`, `status`, or `project_id`
- Invalid assignee → `422`
- Soft-deleted / inactive user → `422`
- Non-member and non-owner of the task’s project → `422`

**Classes:** `TaskController::updateAssignment`, `TaskService::changeAssignment`, `UpdateTaskAssignmentRequest`

### Deliverables

- [x] `PATCH /api/v1/tasks/{task}/assignment`
- [x] `UpdateTaskAssignmentRequest`
- [x] `TaskController::updateAssignment`
- [x] `TaskService::changeAssignment`
- [x] Feature tests: `tests/Feature/Task/TaskAssignmentApiTest.php`

---

## 8. Phase 5.4 — Task Queries

**Status:** ✅ Implemented

Target: `GET /api/v1/tasks`

| Concern | Behavior |
|---------|----------|
| Search | `search` against `title`, `description` (case-insensitive; PostgreSQL `ilike`) |
| Filtering | `status`, `priority`, `project_id`, `assigned_to`, `created_by` (composable with search) |
| Sorting | `sort` + `direction`; allowed: `title`, `status`, `priority`, `due_date`, `created_at`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100; values above 100 clamped to 100); items in `data`, page info in `meta` |

### Query parameter validation (`IndexTasksRequest`)

| Param | Rules / notes |
|-------|----------------|
| `search` | Optional string; max 255 |
| `status` | Optional; `Rule::enum(TaskStatus::class)` |
| `priority` | Optional; `Rule::enum(TaskPriority::class)` |
| `project_id` | Optional integer; `exists:projects,id` |
| `assigned_to` | Optional integer; `exists:users,id` |
| `created_by` | Optional integer; `exists:users,id` |
| `sort` | Optional; one of allowed sorts (default `created_at`) |
| `direction` | Optional; `asc` \| `desc` (default `desc`) |
| `page` | Optional integer ≥ 1 (default `1`) |
| `per_page` | Optional integer 1–100 (default `15`); values above 100 clamped to 100 |

Invalid query params → HTTP `422` (standard API envelope).

**Classes:** `IndexTasksRequest`, `TaskQuery`, `TaskService::list()`, `TaskController::index()`, `ApiResponse::paginatedResponse()`

Follow `UserQuery` / `ProjectQuery` / `IndexUsersRequest` / `IndexProjectsRequest` conventions (including stable `orderBy('id')` tie-break).

### Deliverables

- [x] `TaskQuery`
- [x] `IndexTasksRequest`
- [x] Wire `TaskService::list()` / `TaskController::index()`
- [x] Feature tests: `tests/Feature/Task/TaskListQueryTest.php`

**Note:** Authorization scoping for Employees is enforced in Phase 5.5 via policy/`viewAny` + `TaskQuery` visibility constraints — do not scatter ad-hoc role checks.

---

## 9. Phase 5.5 — Task Authorization

**Status:** ✅ Implemented

Coarse role matrix for Task Management only:

| Role | List | View | Create | Update | Delete | Status | Assignment |
|------|------|------|--------|--------|--------|--------|------------|
| Administrator | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Employee | tasks in accessible projects | tasks in accessible projects | ❌ | ❌ | ❌ | assigned to self only | ❌ |

**Accessible projects (Employee):** projects the user **owns** (`created_by`) **or** is a **member** of (`project_members`) — same definition as `ProjectPolicy` visibility.

**Employee status ability:** `updateStatus` allowed only when `tasks.assigned_to` equals the actor’s id (and the task’s project remains accessible). Unassigned tasks: Employee may view (if project-accessible) but **cannot** change status. Enforced on `PATCH .../status` (Phase 5.6).

**Classes:** `App\Policies\TaskPolicy`; register via `Gate::policy` in `AppServiceProvider`; enforce with `$this->authorize()` in `TaskController`; Employee list scoping in `TaskQuery` (tasks whose `project_id` is in accessible projects).

Unauthorized → HTTP `403` with standard API envelope.

### Deliverables

- [x] `TaskPolicy`
- [x] Register via `Gate::policy`
- [x] `$this->authorize()` in `TaskController`
- [x] Employee visibility in `TaskQuery`
- [x] Feature tests: `tests/Feature/Task/TaskAuthorizationTest.php`

**Out of scope:** Permission tables, task-level custom roles, per-field ACLs beyond this matrix, remark/task comment policies, frontend.

---

## 10. Phase 5.6 — Task Status Workflow

**Status:** ✅ Implemented

### Route (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| PATCH | `/api/v1/tasks/{task}/status` | Update `status` only |

### Request body

```json
{
  "status": "in_progress"
}
```

### Validation (`UpdateTaskStatusRequest`)

| Field | Rules |
|-------|--------|
| `status` | required, `Rule::enum(TaskStatus::class)` |

### Workflow rules (Milestone 5)

| Rule | Decision |
|------|----------|
| Default on create | `todo` |
| Create/update bodies | must **not** accept `status` |
| Allowed target values | any `TaskStatus` value |
| Transition graph | **not enforced** in Milestone 5 (same pattern as Project status) |
| Side effects | status patch does not modify title, priority, assignee, dates, or project |

**Classes:** `TaskController::updateStatus`, `TaskService::changeStatus`, `UpdateTaskStatusRequest`

### Deliverables

- [x] `PATCH /api/v1/tasks/{task}/status`
- [x] `UpdateTaskStatusRequest`
- [x] `TaskController::updateStatus` (authorize via `TaskPolicy::updateStatus`)
- [x] `TaskService::changeStatus`
- [x] Feature tests: `tests/Feature/Task/TaskStatusApiTest.php`

---

## 11. Out of Scope (Future Work)

Explicitly deferred (do **not** implement in Milestone 5):

- Multiple assignees
- Attachments
- Checklists
- Labels / tags
- Time tracking
- Task dependencies / blockers graph beyond `blocked` status value
- Subtasks
- Activity logs
- Comments / Remarks
- Notifications
- Recurring tasks
- Moving a task to another project (`project_id` change)
- Restricted status transition state machine (graph enforcement)
- Vue Task Management UI
- Dashboard / Reports aggregates
- Nested-only routes under `/projects/{project}/tasks` as the primary API (top-level `/api/v1/tasks` is approved)

---

## 12. Acceptance Criteria

### Acceptance (5.1–5.6)

- [x] `tasks` schema matches this approved ERD (soft deletes; FKs **RESTRICT**)
- [x] Morph map includes `task`
- [x] `TaskStatus` and `TaskPriority` enum values match approved lists
- [x] Task CRUD endpoints with standard envelope
- [x] Assignment patch endpoint; assignment rules enforced (`422` on invalid assignee)
- [x] Task list search + filters + sorting + pagination (5.4)
- [x] Coarse authorization enforced (5.5)
- [x] Status patch endpoint; status not accepted on create/update (5.6)
- [x] Feature tests green for 5.1
- [x] Feature tests green for 5.2
- [x] Feature tests green for 5.3
- [x] Feature tests green for 5.4
- [x] Feature tests green for 5.5
- [x] Feature tests green for 5.6
- [x] Companion docs synchronized for Phase 5.1
- [x] Companion docs synchronized for Phase 5.2
- [x] Companion docs synchronized for Phase 5.3
- [x] Companion docs synchronized for Phase 5.4
- [x] Companion docs synchronized for Phase 5.5
- [x] Companion docs synchronized for Phase 5.6

---

## 13. Testing expectations

| Phase | Suggested path | Coverage |
|-------|----------------|----------|
| 5.1 | `tests/Feature/Task/TaskDomainFoundationTest.php` | schema, relations, enums, soft delete, FK RESTRICT, factory |
| 5.2 | `tests/Feature/Task/TaskManagementApiTest.php` | CRUD, validation, defaults (`todo` / `medium`), `created_by`, guest `401`, resource shape |
| 5.3 | `tests/Feature/Task/TaskAssignmentApiTest.php` | assign / clear; active+member/owner only; inactive/soft-deleted/`422`; non-member/`422` |
| 5.4 | `tests/Feature/Task/TaskListQueryTest.php` | search, filters, sort, pagination, clamp, validation, defaults, guest `401` |
| 5.5 | `tests/Feature/Task/TaskAuthorizationTest.php` | Admin/PM full; Employee project-scoped list/view; Employee status only when assigned; denied create/update/delete/assignment |
| 5.6 | `tests/Feature/Task/TaskStatusApiTest.php` | status patch only; create/update reject status; all enum values; Employee assigned-to-self authz |

Existing Milestone 3–4 suites must continue passing.

---

## 14. References

- [DOMAIN_MODEL.md](DOMAIN_MODEL.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [TESTING.md](../TESTING.md)
- [decisions/Task-Management.md](../decisions/Task-Management.md)
- [decisions/Project-Management.md](../decisions/Project-Management.md)
- [decisions/Database.md](../decisions/Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md)
