# Milestone 4 — Project Management

**Status:** ✅ Phases 4.1–4.5 implemented · Milestone 4 complete  
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
| **4.2 Project CRUD** | `ProjectController` / `ProjectService` / Form Requests / Resources / CRUD + status patch | ✅ Implemented |
| **4.3 Project Members** | Member list / add / remove endpoints + service methods | ✅ Implemented |
| **4.4 Project Queries** | Project list search, filters, sorting, pagination `meta` | ✅ Implemented |
| **4.5 Project Authorization** | Coarse role matrix via `ProjectPolicy` | ✅ Implemented |

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

**Status:** ✅ Implemented

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/projects` | List projects (search/filters/sorting/pagination — Phase 4.4) |
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
| List query | `App\Queries\Projects\ProjectQuery` |
| Authorization | `App\Policies\ProjectPolicy` |
| Store validation | `StoreProjectRequest` |
| Update validation | `UpdateProjectRequest` |
| Status validation | `UpdateProjectStatusRequest` |
| Index validation | `IndexProjectsRequest` |
| Resources | `ProjectResource` (+ nested owner `UserResource` when loaded) |
| Status enum | `App\Enums\ProjectStatus` |

### Create / update fields

| Field | Create | Update |
|-------|--------|--------|
| `name` | required | required |
| `description` | optional | optional |
| `status` | **not accepted** — always `planning` | **not accepted** — use status patch |
| `start_date` | optional date | optional date |
| `due_date` | optional date | optional date |
| `created_by` | set server-side from auth user | **not accepted** |

### Behaviors

- Soft delete only
- Status endpoint accepts `ProjectStatus` values only; does not modify other fields
- Eager-load owner for responses; nest via `ProjectResource` + `whenLoaded()`
- Standard response envelope; never return raw models
- List returns search/filter/sort/pagination via Phase 4.4 (`ProjectQuery`)
- Match User Management patterns (`UserController` / `UserService` / status patch)
- Feature tests: `tests/Feature/Project/ProjectManagementApiTest.php`

---

## 7. Phase 4.3 — Project Members

**Status:** ✅ Implemented

### Routes (`auth:sanctum`)

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/v1/projects/{project}/members` | List project members |
| POST | `/api/v1/projects/{project}/members` | Add member (`user_id`) |
| DELETE | `/api/v1/projects/{project}/members/{user}` | Remove member |

### Behaviors

- Add: body `{ "user_id": <id> }`; `joined_at` set server-side to now (client value ignored)
- `user_id` must exist, be active, and not soft-deleted (`422` otherwise)
- Duplicate membership → HTTP `409` (`DuplicateProjectMemberException`)
- Remove: hard-delete pivot row; unknown membership → `404`
- Member list returns `ProjectMemberResource` (user summary + `joined_at`)
- Owner (`created_by`) is **not** auto-added as a member
- Removing membership does **not** change `created_by`
- No invitation emails; no accept/decline flow

### Classes

- `ProjectController::members` / `storeMember` / `destroyMember`
- `ProjectService::listMembers` / `addMember` / `removeMember`
- `StoreProjectMemberRequest`
- `ProjectMemberResource`
- Feature tests: `tests/Feature/Project/ProjectMembersApiTest.php`

---

## 8. Phase 4.4 — Project Queries

**Status:** ✅ Implemented

Target: `GET /api/v1/projects`

| Concern | Behavior |
|---------|----------|
| Search | `search` against `name`, `description` (case-insensitive) |
| Filtering | `status`, `created_by` (composable with search) |
| Sorting | `sort` + `direction`; allowed: `name`, `status`, `start_date`, `due_date`, `created_at`; default `created_at` / `desc` |
| Pagination | `page` / `per_page` (default 15, max 100; values above 100 clamped to 100); items in `data`, page info in `meta` |

**Classes:** `IndexProjectsRequest`, `ProjectQuery`, `ProjectService::list()`, `ProjectController::index()`, `ApiResponse::paginatedResponse()`

Follow `UserQuery` / `IndexUsersRequest` conventions.

Feature tests: `tests/Feature/Project/ProjectListQueryTest.php`

**Note:** Authorization scoping (e.g. Employee sees only owned/member projects) is enforced in Phase 4.5 via policy/`viewAny` + `ProjectQuery` visibility constraints — do not scatter ad-hoc role checks.

---

## 9. Phase 4.5 — Project Authorization

**Status:** ✅ Implemented

Coarse role matrix for Project Management only:

| Role | List | View | Create | Update | Delete | Status | Manage members |
|------|------|------|--------|--------|--------|--------|----------------|
| Administrator | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project Manager | all | all | ✅ | ✅ | ✅ | ✅ | ✅ |
| Employee | owned **or** member | owned **or** member | ❌ | ❌ | ❌ | ❌ | ❌ |

**Classes:** `App\Policies\ProjectPolicy`; register via `Gate::policy` in `AppServiceProvider`; enforce with `$this->authorize()` in `ProjectController`; Employee list scoping in `ProjectQuery`.

Unauthorized → HTTP `403` with standard API envelope.

Feature tests: `tests/Feature/Project/ProjectAuthorizationTest.php`

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

### Completed (4.1–4.5)

- [x] `projects` schema matches approved ERD (soft deletes; `created_by` RESTRICT)
- [x] `project_members` pivot with unique (`project_id`, `user_id`), `joined_at`, timestamps
- [x] Morph map includes `project`
- [x] `ProjectStatus` enum values match approved list
- [x] Project CRUD + status endpoints with standard envelope
- [x] Member list / add / remove endpoints
- [x] Project list search + filters + sorting + pagination (4.4)
- [x] Coarse authorization enforced (4.5)
- [x] Feature tests green for 4.1–4.5
- [x] Docs synchronized for 4.1–4.5 implemented behavior

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
