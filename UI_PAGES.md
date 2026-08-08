# UI Pages

## Public

> Milestone 8 — ✅ Login implemented

- Login (`/login` — Milestone 8.2)

---

## App shell (Milestone 8)

> ✅ Complete · Spec: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md)

- Auth layout: sidebar + topbar; **Logout in topbar**
- Guest layout: centered login
- Forbidden (`/403`) · Not Found (catch-all)
- Landing: authenticated `/` → `/dashboard` (Phase 9.1)
- Sidebar (M9 complete): Dashboard, Users (role), Projects, Tasks, Reports, Employee reports (Admin/PM) / My report (Employee), Profile — no “Coming later” for M9 modules
- Global `AppProgressBar` (route + page-level HTTP; `/api/v1/lookups/*` excluded; Create/Edit modal aliases do not start route progress)
- Auth bootstrap: full-screen `AppSpinner` until `/me` completes

---

## Dashboard

> API: Milestone 6 ✅ · Vue: Phase 9.1 ✅  
> Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

- Dashboard Overview (`/dashboard` — `GET /api/v1/dashboard`)
- Stat cards · Tailwind status bars · recent work
- Skeleton / empty recent / error+retry (`DashboardSkeleton`; soft refresh keeps prior data)
- Landing: `/` → `/dashboard`

---

## Organization & User Management

> API: Milestone 3 ✅ · Vue: Phase 9.2 ✅  
> Spec: [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Routes

| Path | Behavior |
|------|----------|
| `/users` | User directory (Admin, PM) |
| `/users/create` | Same list + **Create** modal (`UserFormDialog`) |
| `/users/:id/edit` | Same list + **Edit** modal (`UserFormDialog`) |
| `/users/:id` | **Show page** (deep link; Employee may view self) |
| `/profile` | Current user profile page (reuses show UI) |

### List UX

- Search (debounced), filters (`role_id`, `department_id`, `job_title_id`, `status`), pagination
- Desktop `AppTable`; mobile card stack below `md`
- Columns: name, email, role badge, department, job title, status badge, last login, actions
- **Clear** filter control always visible; disabled when no filters active (no layout shift)
- Actions via teleported `AppDropdownMenu` (View / Edit / Activate|Deactivate / Delete) — not clipped by table overflow
- View opens **detail modal**; Edit opens **form modal**; status/delete use `AppConfirmDialog`
- Lookups (roles / departments / job titles) use shared `useLookups` SPA-session cache + in-flight dedupe; list search/filters/pagination remain server-side
- Loading: `UserListSkeleton` on initial load; soft opacity refresh; empty ≠ loading

### Shared building blocks

`AppModal`, `AppConfirmDialog`, `AppDropdownMenu`, `AppTable`, `AppPagination`, `AppSearch`, `AppFilterBar`, `AppSelect`, `UserForm`, `UserDetailPanel`, `UserListSkeleton`

---

## Project Management

> API: Milestone 4 ✅ · Vue: Phase 9.3 ✅  
> Spec: [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Routes

| Path | Behavior |
|------|----------|
| `/projects` | Project directory (table) |
| `/projects/create` | Same list + **Create** modal (`ProjectFormDialog`) |
| `/projects/:id/edit` | Same list + **Edit** modal (`ProjectFormDialog`) |
| `/projects/:id` | **Show page** (workspace: info + members + `ProjectTasksPanel`) |

### List UX

- Search, status filter, pagination; desktop `AppTable`; mobile card stack below `md`
- Create/Edit/View via **modals** on the list (`AppModal`) — same pattern as Users/Tasks
- Status patch via small `AppModal`; soft delete via `AppConfirmDialog`
- Authz: Admin/PM mutate; Employee list/view only

### Show workspace

- Project information; in-place edit via `ProjectFormDialog`
- Members: list / add (inline select) / remove confirm (`AppConfirmDialog`); API `409` for duplicates
- Tasks: `ProjectTasksPanel` (Phase 9.4)

### Shared building blocks

`AppModal`, `AppConfirmDialog`, `ProjectForm`, `ProjectFormDialog`, `ProjectDetailDialog`, `ProjectActionsMenu`, `ProjectMembersPanel`, `ProjectTasksPanel`, `ProjectListSkeleton`

---

## Task Management

> API: Milestone 5 ✅ · Vue: Phase 9.4 ✅ — **table only** (Kanban later)  
> Spec: [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

### Routes

| Path | Behavior |
|------|----------|
| `/tasks` | Task directory (table) |
| `/tasks/create` | Same list + **Create** modal (`TaskFormDialog`) |
| `/tasks/:id/edit` | Same list + **Edit** modal (`TaskFormDialog`) |
| `/tasks/:id` | **Show page** (deep link) |

### List UX

- Search, filters (status / priority / project), pagination
- Create/Edit/View via **modals** on the list (`AppModal`) — same locked pattern as Users
- Status patch; assignment via `TaskAssignmentDialog` / detail panel; soft delete confirm; priority badges (`StatusBadge` kind=priority)
- Authz: Admin/PM mutate; Employee list/view + status only when assigned to self
- Also embedded on Project Show via `ProjectTasksPanel`
- Loading: `TaskListSkeleton` on initial load; soft opacity refresh; empty ≠ loading

### Components

`TaskForm`, `TaskFormDialog`, `TaskDetailDialog`, `TaskDetailPanel`, `TaskAssignmentDialog`, `TaskActionsMenu`, `TaskListSkeleton`

---

## Reports

> API: Milestone 7 ✅ · Vue: Phase 9.5 ✅  
> Spec: [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

| Path | Behavior |
|------|----------|
| `/reports` | Redirect → project reports |
| `/reports/projects` | Project reports list |
| `/reports/projects/:id` | Project report detail |
| `/reports/employees` | Employee reports list (Admin/PM) |
| `/reports/employees/:id` | Employee report detail (Employee: self via My report) |

- `ReportDateFilters` (`from_date` / `to_date`)
- Reuse `DashboardStatCard` / `DashboardStatusBar` (**Tailwind bars only** — no chart library)
- Loading: `AppTableSkeleton` / `AppReportSkeleton`; soft refresh keeps prior data; empty ≠ loading; error+retry
- Role-aware nav: Employee list Admin/PM only; Employee self via My report
- No exports; no chart libraries; dedicated list/detail **pages** (not modal CRUD)

---

## Activity Logs

> Planned — future

- Activity List

---

## Cross-cutting UI (Milestone 9 post-ship)

- Shared skeletons: `AppSkeleton`, `AppTableSkeleton`, `AppCardSkeleton`, `AppDetailSkeleton`, `AppReportSkeleton`
- Stable modal `viewKey` on Users / Projects / Tasks Create/Edit aliases (`AuthLayout`) so the underlying list is not remounted
- Loading remains distinct from Empty State; friendly inline retry on load failure

---

## Error

> Milestone 8 — ✅

- Forbidden (`/403`)
- Not Found (catch-all)
