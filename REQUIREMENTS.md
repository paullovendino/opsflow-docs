# Functional Requirements

## Authentication

- [x] User login (`POST /api/v1/auth/login`)
- [x] User logout (`POST /api/v1/auth/logout`)
- [x] Protected routes (`auth:sanctum`)
- [x] Current authenticated user (`GET /api/v1/auth/me`)
- [x] Inactive users cannot log in (`403` / `Account is inactive.`)
- [x] Coarse role-based access for User Management (Phase 3.6)
- [ ] Frontend Pinia authentication

---

## Organization Structure

> Phases 3.1–3.6 implemented (Milestone 3 complete)

- [x] Seeded Departments (approved list; soft deletes)
- [x] Seeded Job Titles (approved list; soft deletes)
- [x] Seeded Roles remain available
- [x] Users belong to one Role; optional Department and Job Title
- [x] Lookup collection endpoints under `/api/v1/lookups` for roles, departments, job titles — all authenticated users (Phase 3.4)

Department CRUD, Job Title CRUD, teams, branches, and organization settings are out of scope for Milestone 3.

---

## Dashboard

> Milestone 6 — ✅ Complete (Phases 6.1–6.4)  
> Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · ADR: [decisions/Dashboard.md](decisions/Dashboard.md)

- [x] Project summary (`projects.total`, `projects.by_status`)
- [x] Task summary (`tasks.total`, `tasks.by_status`, `tasks.by_priority`)
- [x] Statistics cards (`overdue`, `assigned_to_me`)
- [x] Recent work items (derived from projects/tasks — not Activity Logs)
- [x] Chart-ready JSON breakdowns (backend does not render charts)
- [x] Visibility: Admin/PM org-wide; Employee owned-or-member
- [x] `GET /api/v1/dashboard` under `auth:sanctum`

---

## User Management

> Phase 3.3 — Implemented (CRUD APIs)

- [x] Create users (`POST /api/v1/users`)
- [x] Update users (`PUT /api/v1/users/{user}`)
- [x] Soft-delete users (`DELETE /api/v1/users/{user}`)
- [x] Activate / deactivate users (`PATCH /api/v1/users/{user}/status`)
- [x] List users (`GET /api/v1/users`) — search, filters, sorting, pagination (Phase 3.5)
- [x] Show user (`GET /api/v1/users/{user}`)
- [x] Assign role (required)
- [x] Assign department (optional)
- [x] Assign job title (optional)
- [x] User list: `search`, filters (`role_id`, `department_id`, `job_title_id`, `status`), sorting, pagination (Phase 3.5)
- [x] Coarse authorization (Phase 3.6):
  - Administrator — full user management
  - Project Manager — read-only directory
  - Employee — view own profile

---

## Project Management

> Milestone 4 — ✅ Phases 4.1–4.5 implemented · Milestone 4 complete  
> Spec: [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md)

- [x] Project domain foundation (`projects`, `project_members`, `ProjectStatus`)
- [x] Project CRUD (`GET/POST /api/v1/projects`, `GET/PUT/DELETE /api/v1/projects/{project}`)
- [x] Project status patch (`PATCH /api/v1/projects/{project}/status`)
- [x] Project members (`GET/POST /api/v1/projects/{project}/members`, `DELETE .../members/{user}`)
  - Owner independent of membership; `joined_at` server-only; duplicate → `409`; active users only
- [x] Project list search / filters / sorting / pagination
- [x] Coarse authorization (`ProjectPolicy`):
  - Administrator — full project management (all projects)
  - Project Manager — full project management (all projects)
  - Employee — list/view owned or member projects only

---

## Task Management

> Milestone 5 — ✅ Complete (Phases 5.1–5.6)  
> Spec: [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md) · ADR: [decisions/Task-Management.md](decisions/Task-Management.md)

- [x] Task domain foundation (`tasks`, `TaskStatus`, `TaskPriority`)
- [x] Task CRUD (`GET/POST /api/v1/tasks`, `GET/PUT/DELETE /api/v1/tasks/{task}`)
- [x] Task assignment (`PATCH /api/v1/tasks/{task}/assignment`)
  - Single nullable assignee; must be active + project owner or member
- [x] Task list search / filters / sorting / pagination
- [x] Coarse authorization (`TaskPolicy`):
  - Administrator — full task management (all tasks)
  - Project Manager — full task management (all tasks)
  - Employee — list/view tasks in accessible projects; status only when assigned to self
- [x] Task status patch (`PATCH /api/v1/tasks/{task}/status`)

---

## Remarks

> Planned — future

- Author remarks on work items (design in [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md))

---

## Activity Logs

> Planned — future (not Milestone 6)

- User actions
- Task updates
- Project updates

---

# Non-Functional Requirements

- Responsive UI
- RESTful API
- Secure Authentication
- Fast Response Time
- Scalable Architecture
- Clean Code
- Mobile Friendly
- Production Ready
- Documentation-first milestone design before implementation
