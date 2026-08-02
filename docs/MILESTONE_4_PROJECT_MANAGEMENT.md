# Milestone 4 — Project Management

**Status:** Phase 4.1 implemented · Phases 4.2–4.5 pending  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-02

> Implementation specification for Milestone 4.  
> Domain reference: [DOMAIN_MODEL.md](DOMAIN_MODEL.md)  
> ADR: [decisions/Project-Management.md](../decisions/Project-Management.md)

---

## 1. Goal

Introduce OpsFlow’s Project Management module on the API:

- Persist Projects with owner, status, dates, and soft deletes
- Associate Users with Projects via `project_members` (no member roles)
- Expose Project CRUD + status changes
- Support Project Members add / list / remove
- Apply list search / filter / sort / pagination
- Enforce coarse role-based authorization via `ProjectPolicy`

---

## 2. Domain summary

```text
User (owner via created_by)
    └── Project
            ├── has many → Project Members (Users)
            └── (Tasks deferred to Phase 5)
```

- Each Project has **one owner** (`projects.created_by` → `users.id`)
- Projects have **many members** through `project_members`
- Owner and membership are independent: access for Employees is **owner OR member**
- No member roles, invitations, or pivot-level permissions in Phase 4

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **4.1 Project Domain Foundation** | `projects` / `project_members` tables; models; relations; `ProjectStatus` enum; morph alias `project` | ✅ Implemented |
| **4.2 Project CRUD** | `ProjectController` / `ProjectService` / Form Requests / Resources / CRUD + status patch | Pending |
| **4.3 Project Members** | Member list / add / remove endpoints + service methods | Pending |
| **4.4 Project Queries** | Project list search, filters, sorting, pagination `meta` | Pending |
| **4.5 Project Authorization** | Coarse role matrix via `ProjectPolicy` | Pending |

---

## 4. Approved schema decisions

### Projects

| Column | Notes |
|--------|-------|
| id | PK |
| name | required string |
| description | nullable text |
| status | enum-backed string (`ProjectStatus`) |
| start_date | nullable date |
| due_date | nullable date |
| created_by | required FK → `users.id` (**RESTRICT**) |
| created_at / updated_at | timestamps |
| deleted_at | **soft deletes** |

**Status values** (`App\Enums\ProjectStatus`):

| Enum case / stored value | Display |
|--------------------------|---------|
| `planning` | Planning |
| `active` | Active |
| `on_hold` | On Hold |
| `completed` | Completed |
| `archived` | Archived |

**Defaults / behaviors:**

- On create: `created_by` = authenticated user; default `status` = `planning` when omitted
- `created_by` is **not** changeable via update in Phase 4 (no ownership transfer)
- Soft delete only (no hard delete API)
- Soft-deleted projects are excluded from default lists

### Project Members (`project_members`)

| Column | Notes |
|--------|-------|
| id | PK |
| project_id | FK → `projects.id` |
| user_id | FK → `users.id` |
| joined_at | timestamp (set when member is added) |
| created_at / updated_at | timestamps |

**Rules:**

- Unique (`project_id`, `user_id`)
- No member role / permission columns
- No invitation workflow
- No soft deletes on the pivot — remove = hard delete of the pivot row
- FK delete: `project_id` / `user_id` use **RESTRICT** on hard delete while rows reference them (soft-deleted projects/users keep pivot rows intact)
- Owner is **not** auto-inserted into `project_members` on create; membership is explicit

Full physical schema: [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)

---

## 5. Phase 4.1 — Project Domain Foundation

**Status:** ✅ Implemented

Deliverables:

- [x] Migration(s) for `projects` and `project_members`
- [x] Models: `App\Models\Project`, membership via `belongsToMany` / `project_members`
- [x] `App\Enums\ProjectStatus`
- [x] Eloquent relations:
  - `Project::owner()` / `createdBy()` → `belongsTo(User::class, 'created_by')`
  - `Project::members()` → `belongsToMany(User::class, 'project_members')->withPivot(...)->withTimestamps()`
  - `User::ownedProjects()` / `User::projects()`
- [x] Morph map alias: `project` → `App\Models\Project`
- [x] `ProjectFactory`
- [x] Feature tests: `tests/Feature/Project/ProjectDomainFoundationTest.php`

**Out of scope for 4.1:** HTTP CRUD, members APIs, queries, policies. (No project seeders — transactional data only.)

---

## 6. Phase 4.2 — Project CRUD

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/projects` | List projects (query behavior finalized in 4.4; authz in 4.5) |
| GET | `/api/v1/projects/{project}` | Show project |
| POST | `/api/v1/projects` | Create project |
| PUT | `/api/v1/projects/{project}` | Update project |
| DELETE | `/api/v1/projects/{project}` | Soft delete |
| PATCH | `/api/v1/projects/{project}/status` | Update `status` only |

### Layering

| Concern | Class |
|---------|-------|
| Controller | `App\Http\Controllers\Api\V1\ProjectController` (thin) |
| Service | `App\Services\Projects\ProjectService` |
| List query | `App\Queries\Projects\ProjectQuery` (wired in 4.4) |
| Authorization | `App\Policies\ProjectPolicy` (wired in 4.5) |
| Store validation | `StoreProjectRequest` |
| Update validation | `UpdateProjectRequest` |
| Status validation | `UpdateProjectStatusRequest` |
| Index validation | `IndexProjectsRequest` (wired in 4.4) |
| Resources | `ProjectResource` (+ nested owner `UserResource` summary when loaded) |
| Status enum | `App\Enums\ProjectStatus` |

### Create / update fields

| Field | Create | Update |
|-------|--------|--------|
| `name` | required | required |
| `description` | optional | optional |
| `status` | optional (default `planning`) | optional (or use status patch) |
| `start_date` | optional date | optional date |
| `due_date` | optional date | optional date |
| `created_by` | set server-side from auth user | **not accepted** |

### Behaviors

- Soft delete only
- Status endpoint accepts `ProjectStatus` values only; does not modify other fields
- Eager-load owner (and members when useful) for responses; nest via API Resources + `whenLoaded()`
- Standard response envelope; never return raw models
- Match User Management patterns (`UserController` / `UserService` / status patch)

---

## 7. Phase 4.3 — Project Members

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/projects/{project}/members` | List project members |
| POST | `/api/v1/projects/{project}/members` | Add member (`user_id`) |
| DELETE | `/api/v1/projects/{project}/members/{user}` | Remove member |

### Behaviors

- Add: body `{ "user_id": <id> }`; set `joined_at` to now; reject duplicates (`422` or conflict message via validation)
- `user_id` must exist and refer to a non-soft-deleted user
- Remove: delete pivot row; idempotent-or-404 per Laravel route-model conventions (document chosen behavior in tests)
- Member list returns nested user summary resources (not full password-bearing payloads)
- Owner (`created_by`) is **not** required to be in `project_members`
- Removing membership does **not** change `created_by`
- No invitation emails; no accept/decline flow

### Classes

- Prefer methods on `ProjectService` (or a focused collaborator if needed) called from `ProjectController` / dedicated thin actions — keep controllers thin
- Form Requests: `StoreProjectMemberRequest` (and index validation only if query params are added later)
- Response: member collection shaped for API consistency (user fields + `joined_at`)

---

## 8. Phase 4.4 — Project Queries

Target: `GET /api/v1/projects`

| Concern | Behavior |
|---------|----------|
| Search | `search` against `name`, `description` (case-insensitive) |
| Filtering | `status`, `created_by` (composable with search) |
| Sorting | `sort` + `direction`; allowed: `name`, `status`, `start_date`, `due_date`, `created_at`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100; values above 100 clamped to 100); items in `data`, page info in `meta` |

**Classes:** `IndexProjectsRequest`, `ProjectQuery`, `ProjectService::list()`, `ProjectController::index()`, `ApiResponse::paginatedResponse()`

Follow `UserQuery` / `IndexUsersRequest` conventions.

**Note:** Authorization scoping (e.g. Employee sees only owned/member projects) is enforced in Phase 4.5 via policy/`viewAny` + query constraints — do not scatter ad-hoc role checks.

---

## 9. Phase 4.5 — Project Authorization

Coarse role matrix for Project Management only:

| Role | List | View | Create | Update | Delete | Status | Manage members |
|------|------|------|--------|--------|--------|--------|----------------|
| Administrator | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Employee | owned **or** member | owned **or** member | ❌ | ❌ | ❌ | ❌ | ❌ |

**Classes:** `App\Policies\ProjectPolicy`; register via `Gate::policy` in `AppServiceProvider`; enforce with `$this->authorize()` in `ProjectController`.

Unauthorized → HTTP `403` with standard API envelope.

**Out of scope:** Permission tables, project-level custom roles, member-level permissions, task/remark policies, frontend.

---

## 10. Out of Scope (Future Work)

- Tasks module (Phase 5)
- Remarks / Activity Logs
- Dashboard / Reports
- Project templates, archiving workflows beyond `archived` status, ownership transfer
- Member roles / invitations / pivot permissions
- Advanced RBAC matrices
- Vue Project Management UI
- Multi-organization / `organizations` table

---

## 11. Acceptance Criteria

### Pending (4.1–4.5)

- [x] `projects` schema matches approved ERD (soft deletes; `created_by` RESTRICT)
- [x] `project_members` pivot with unique (`project_id`, `user_id`), `joined_at`, timestamps
- [x] Morph map includes `project`
- [x] `ProjectStatus` enum values match approved list
- [ ] Project CRUD + status endpoints with standard envelope
- [ ] Member list / add / remove endpoints
- [ ] Project list search + filters + sorting + pagination (4.4)
- [ ] Coarse authorization enforced (4.5)
- [x] Feature tests green for 4.1
- [ ] Feature tests green for 4.2–4.5
- [x] Docs synchronized for 4.1 implemented behavior

---

## 12. References

- [DOMAIN_MODEL.md](DOMAIN_MODEL.md)
- [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [TESTING.md](../TESTING.md)
- [decisions/Project-Management.md](../decisions/Project-Management.md)
- [decisions/Database.md](../decisions/Database.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
