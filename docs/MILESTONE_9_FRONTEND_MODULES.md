# Milestone 9 — Frontend Modules

**Status:** 🔄 In progress — ✅ Phase 9.1 complete · awaiting Phase 9.2 approval  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-04

> Implementation specification for Milestone 9 (`opsflow-web` feature UIs).  
> ADR: [decisions/Frontend-Modules.md](../decisions/Frontend-Modules.md)  
> Prerequisite: Milestone 8 complete (auth shell, Axios/Sanctum, layouts, shared UI primitives)  
> Backend APIs: Milestones 3–7 complete — **consume only**; no new API/schema in Milestone 9

---

## 1. Goal

Ship OpsFlow’s authenticated **product modules** on top of the Milestone 8 foundation:

| Phase | Module | Primary APIs | Status |
|-------|--------|--------------|--------|
| **9.1** | Dashboard UI | `GET /api/v1/dashboard` | ✅ Implemented |
| **9.2** | User Management | `/api/v1/users`, `/api/v1/lookups/*` | 📋 Designed |
| **9.3** | Project Management | `/api/v1/projects`, members | 📋 Designed |
| **9.4** | Task Management | `/api/v1/tasks`, assignment, status | 📋 Designed |
| **9.5** | Reports | `/api/v1/reports/projects`, `/api/v1/reports/employees` | 📋 Designed |

Replace the Milestone 8 App Home placeholder with a real Dashboard landing experience and enable role-aware sidebar navigation.

**Phase order is mandatory.** Do not start a later phase until the prior phase is complete and approved.

---

## 2. Locked cross-cutting decisions

| Decision | Locked choice |
|----------|---------------|
| Landing route | Authenticated `/` **redirects to** `/dashboard`. `AppHomeView` removed (Phase 9.1). |
| Layout | Continue **nested** `AuthLayout` / `GuestLayout` from Milestone 8 |
| Charts | **No chart library** in Milestone 9. Use Tailwind compositional charts (bars / stacked segments). Chart.js etc. require a future ADR. |
| UI kit | **None** (same as M8) |
| Tables | Shared `AppTable` + mobile **card stack** fallback (`md+` table) |
| Destructive confirms | Shared `AppConfirmDialog` (modal) — not browser `confirm()` |
| Create/Edit UX (Users, Projects, Tasks) | **Dedicated pages** (not create/edit modals). Modals only for confirms / small member-add patterns where noted. |
| Tasks board | **Table only**. Kanban deferred (roadmap v1.2). |
| Pinia | Keep global `auth` + `ui`. Module list/filter state lives in **page composables** (or thin module stores only if URL sync becomes unwieldy — default: composables). |
| Services | One `*Service.ts` per domain under `services/` calling shared `http` |
| CSRF | Existing Axios `withCredentials` + `withXSRFToken`. Mutating calls after login reuse cookie; on HTTP `419`, toast + `fetchCsrfCookie()` once + optional single retry (document in service helper). |
| Toasts | Success toasts on create/update/delete/status/assignment. Inline field errors for `422`. `403` toast (or empty+message); do **not** auto-route to `/403` for API denials. |
| Role-aware nav | Sidebar links enabled; hide/disable by role (see §3) |
| Backend | **No** new endpoints, migrations, or contract changes |
| Testing / Deployment | **Out of Milestone 9** → Phase 10 Testing · Phase 11 Deployment |

### Role-aware navigation (sidebar)

| Nav item | Administrator | Project Manager | Employee |
|----------|---------------|-----------------|----------|
| Dashboard | ✅ | ✅ | ✅ |
| Users | ✅ | ✅ (read) | ❌ (use Profile only) |
| Projects | ✅ | ✅ | ✅ |
| Tasks | ✅ | ✅ | ✅ |
| Reports | ✅ | ✅ | ✅ (API scopes apply) |
| Profile | ✅ | ✅ | ✅ |

---

## 3. Target folder structure (extends Milestone 8)

```text
opsflow-web/src/
├── components/
│   ├── layout/          # AppSidebar, AppTopbar (enable real nav in 9.1)
│   └── ui/              # AppButton, AppInput, AppSpinner, AppEmptyState, AppToastHost
│                        # + AppTable, AppBadge, AppPagination, AppSelect, AppTextarea
│                        # + AppConfirmDialog, AppStatCard, AppBarChart (Tailwind bars)
├── composables/         # useAuth, useToast, usePaginationQuery, useLookupQuery, …
├── layouts/             # GuestLayout, AuthLayout
├── modules/
│   ├── dashboard/
│   │   ├── components/
│   │   └── views/       # DashboardView.vue
│   ├── users/
│   │   ├── components/
│   │   └── views/       # UserListView, UserCreateView, UserEditView, UserShowView, ProfileView
│   ├── projects/
│   │   ├── components/
│   │   └── views/
│   ├── tasks/
│   │   ├── components/
│   │   └── views/
│   └── reports/
│       ├── components/
│       └── views/
├── router/              # register module routes under AuthLayout
├── services/            # http, authService + dashboard/user/project/task/report/lookup services
├── stores/              # auth, ui (no required module stores)
├── types/               # api, auth + domain types
└── utils/
```

**Routing:** Views are imported from `modules/*/views`. Do not invent a second layout system.

---

## 4. Shared reusable UI (Milestone 9 additions)

| Component | Purpose |
|-----------|---------|
| `AppTable` | Desktop table shell (header + slots + empty row) |
| `AppBadge` | Neutral badge; status/priority wrappers use it |
| `AppPagination` | Prev/next + page meta (`current_page`, `last_page`, `total`) |
| `AppSelect` | Native or styled select + label + error |
| `AppTextarea` | Multiline + label + error |
| `AppConfirmDialog` | Title, body, confirm/cancel; loading on confirm |
| `AppStatCard` | Label + value (+ optional hint) for dashboard/report cards |
| `AppBarChart` | Horizontal bars from `{ label, value }[]` (Tailwind only) |
| `StatusBadge` / `PriorityBadge` | Map API enums to colors (locked palettes below) |

### Locked badge colors (Tailwind)

**User / project status-ish:**

| Value | Style guidance |
|-------|----------------|
| `active` | green |
| `inactive` | slate |
| `planning` | sky |
| `on_hold` | amber |
| `completed` | emerald |
| `cancelled` | rose |

**Task status:** `todo` slate · `in_progress` blue · `in_review` violet · `blocked` amber · `completed` emerald · `cancelled` rose  

**Task priority:** `low` slate · `medium` sky · `high` orange · `urgent` red  

Exact class strings chosen at implementation; semantics above are locked.

### Cross-cutting UX

| State | Pattern |
|-------|---------|
| Loading | Page-level `AppSpinner` or table skeleton; button loading on submit |
| Empty | `AppEmptyState` with short copy + optional CTA |
| Error (load fail) | Inline error panel + retry; toast for unexpected 5xx |
| `422` | Field errors on forms |
| Success | Toast (`ui` store) |
| Responsive | Filters stack vertically on mobile; tables → card list below `md` |

---

## 5. Phase 9.1 — Dashboard UI

**Status:** ✅ Implemented

### Scope

- Replace App Home with Dashboard as default authenticated landing
- Consume `GET /api/v1/dashboard` (optional `recent_limit`, default 10, max 25)
- Statistics cards + Tailwind status bars + recent work list
- Enable Dashboard sidebar link (other module links remain disabled until later phases)

### Architecture

```text
DashboardView → useDashboard() → dashboardService.getSummary()
             → DashboardStatCard / DashboardStatusBar / DashboardRecentWork
```

### Routes

| Path | Name | Meta |
|------|------|------|
| `/dashboard` | `dashboard` | `requiresAuth`, `title: Dashboard` |
| `/` | — | redirect → `{ name: 'dashboard' }` |

### Components / views

- `modules/dashboard/views/DashboardView.vue`
- `DashboardSection`, `DashboardStatCard`, `DashboardStatusBar`, `DashboardRecentWork`, `DashboardSkeleton`
- Shared: `AppBadge`, `StatusBadge`, `AppEmptyState`, `AppButton`

### Composables / stores / services

- `composables/useDashboard.ts` — fetch, loading, error, retry
- `services/dashboardService.ts` — `getSummary(params?)`
- Types: `types/dashboard.ts` matching API resource
- **No** dedicated Pinia dashboard store

### API consumption

- `GET /api/v1/dashboard?recent_limit=`
- Display: `projects.total`, `projects.by_status`, `tasks.total`, `tasks.by_status`, `tasks.by_priority`, `overdue`, `assigned_to_me`, `recent[]`

### UX states

| State | Behavior |
|-------|----------|
| Loading | `DashboardSkeleton` in main content |
| Empty recent | Empty state “No recent work yet” |
| Zero stats | Cards show `0` (not empty page) |
| Error | Inline retry (friendly message; no raw Axios errors) |
| Toast | Prefer inline for load failure |

### Layout (locked)

1. Top: page title + welcome subtitle  
2. Overview: **4** stat cards — Projects total · Tasks total · Overdue · Assigned to me  
3. Status: Tailwind bars — **Tasks by status** · **Tasks by priority** · **Projects by status**  
4. Bottom: Recent work list (icon, title, type, updated date, status badge). Deep links deferred until later modules.

### Responsive

- Cards: 1 col mobile → 2 → 4  
- Status bars stack on mobile / tablet  
- Recent list full width  
- Sidebar overlay behavior unchanged  

### Out of scope (9.1)

- Date filters on dashboard (Reports owns date range)  
- Activity Logs  
- Chart libraries  
- Users/Projects/Tasks/Reports pages  

### Acceptance criteria (9.1)

- [x] `/` redirects to `/dashboard`  
- [x] Dashboard loads for Admin/PM/Employee with API-scoped data  
- [x] Cards + Tailwind status bars + recent list  
- [x] Loading / empty recent / error+retry  
- [x] Sidebar Dashboard link active  
- [x] No new API/schema  
- [x] `npm run type-check` + `npm run build` green  

---

## 6. Phase 9.2 — User Management

### Scope

- User list with search, filters, sort, pagination  
- Create / Edit / Show pages  
- Status activate/deactivate with confirm  
- Lookups for role / department / job title selects  
- Profile page for current user  

### Architecture

```text
UserListView → useUserList() → userService.list / lookupService
UserCreate/Edit → forms → userService.create/update
UserShow / Profile → userService.show
```

### Routes

| Path | Name | Notes |
|------|------|-------|
| `/users` | `users.index` | Admin, PM |
| `/users/create` | `users.create` | Admin |
| `/users/:id` | `users.show` | Admin, PM; Employee own id only (API enforces) |
| `/users/:id/edit` | `users.edit` | Admin |
| `/profile` | `profile` | Authenticated → show self (reuse show UI) |

### Components / views

- `UserListView`, `UserCreateView`, `UserEditView`, `UserShowView`, `ProfileView`
- `UserFilters`, `UserForm`, `UserStatusBadge`

### Locked UX

| Topic | Choice |
|-------|--------|
| Create/Edit | **Pages** with `UserForm` |
| Table columns | Name, email, role, department, job title, status, actions |
| Search | Debounced `search` query param |
| Filters | `role_id`, `department_id`, `job_title_id`, `status` |
| Pagination | Server `meta`; `AppPagination` |
| Status | Badge + Admin actions via confirm → `PATCH .../status` |
| Delete | Admin confirm → `DELETE` soft delete |
| Employee | No Users nav; Profile only |

### Services / types

- `userService.ts`, `lookupService.ts`  
- `types/user.ts`, `types/lookup.ts`  

### UX states

Loading table/form; empty “No users found”; `422` inline; success toasts; `403` toast + stay/redirect list.

### Responsive

Filters stack; table → cards with primary actions.

### Out of scope (9.2)

- Role/Department/Job Title CRUD screens  
- Invitation emails / password reset  
- Avatar upload  

### Acceptance criteria (9.2)

- [ ] List query parity with API  
- [ ] Create/Edit/Show/Profile work with authz  
- [ ] Status + delete confirms  
- [ ] Lookups populate selects  

---

## 7. Phase 9.3 — Project Management

### Scope

- Project list (search/filters/sort/pagination)  
- Create / Edit / Show pages  
- Status patch  
- Members on Show: list, add, remove  

### Routes

| Path | Name |
|------|------|
| `/projects` | `projects.index` |
| `/projects/create` | `projects.create` |
| `/projects/:id` | `projects.show` |
| `/projects/:id/edit` | `projects.edit` |

### Locked UX

| Topic | Choice |
|-------|--------|
| Create/Edit | **Pages** |
| Status badge | ProjectStatus palette |
| Members | Section on **Show** page |
| Add member | **Inline panel** on Show (user select + Add) — not a separate route; small confirm optional |
| Remove member | `AppConfirmDialog` → `DELETE .../members/{user}` |
| Duplicate member | Surface API `409` message via toast/inline |
| Employee | List/view accessible only; no create/edit/members mutate |

### Components

- `ProjectListView`, `ProjectCreateView`, `ProjectEditView`, `ProjectShowView`
- `ProjectFilters`, `ProjectForm`, `ProjectMembersPanel`, `ProjectStatusBadge`

### Services

- `projectService.ts` (CRUD, status, members)

### Out of scope (9.3)

- Project templates, files, remarks  

### Acceptance criteria (9.3)

- [ ] Full CRUD + status + members UX against API  
- [ ] Authz-aligned actions hidden/disabled in UI (API remains source of truth)  

---

## 8. Phase 9.4 — Task Management

### Scope

- Task list (search/filters/sort/pagination)  
- Create / Edit / Show  
- Assignment patch  
- Status patch (workflow UI without transition graph)  

### Locked UX

| Topic | Choice |
|-------|--------|
| Layout | **Table** (not Kanban) |
| Create/Edit | **Pages** |
| Assignment | Show page + list row action: select user / clear → `PATCH .../assignment` |
| Status | Select or button group of `TaskStatus` values → `PATCH .../status` (any status allowed per API) |
| Priority | Badge colors locked above |
| Filters | status, priority, project_id, assigned_to, created_by, search |

### Routes

| Path | Name |
|------|------|
| `/tasks` | `tasks.index` |
| `/tasks/create` | `tasks.create` |
| `/tasks/:id` | `tasks.show` |
| `/tasks/:id/edit` | `tasks.edit` |

### Components

- `TaskListView`, `TaskCreateView`, `TaskEditView`, `TaskShowView`
- `TaskFilters`, `TaskForm`, `TaskStatusControl`, `TaskAssignmentControl`, badges

### Services

- `taskService.ts`

### Employee UX

- List/view scoped by API  
- Status change only when assigned to self (hide/disable otherwise; honor API `403`)  
- No create/edit/delete/assignment mutate  

### Out of scope (9.4)

- Kanban, calendar, time tracking, bulk edit  

### Acceptance criteria (9.4)

- [ ] Table CRUD + assignment + status  
- [ ] Priority/status badges  
- [ ] Role-appropriate controls  

---

## 9. Phase 9.5 — Reports

### Scope

- Project reports list + detail  
- Employee reports list + detail (Admin/PM list; Employee self detail)  
- Optional `from_date` / `to_date` filters  
- Cards + Tailwind charts + tables  

### Routes

| Path | Name |
|------|------|
| `/reports/projects` | `reports.projects.index` |
| `/reports/projects/:id` | `reports.projects.show` |
| `/reports/employees` | `reports.employees.index` |
| `/reports/employees/:id` | `reports.employees.show` |

Optional index `/reports` → redirect to project reports.

### Locked UX

| Topic | Choice |
|-------|--------|
| Date filters | Inclusive `Y-m-d` inputs; validate client `to >= from`; API `422` inline |
| List | Table of summaries + pagination/search/status as API allows |
| Detail | Stat cards + bar charts + (employee) `by_project` table |
| Charts | Tailwind bars from `by_status` / `by_priority` |
| Employee nav | Hide employee report **list** for Employee role; allow self detail via profile link or reports entry that opens self |

### Components

- Project/Employee list + show views  
- `ReportDateFilters`, `ReportStatCards`, `ReportCharts`, tables  

### Services

- `reportService.ts`

### Out of scope (9.5)

- PDF/CSV export, scheduled reports, custom builders, Activity Log reports  

### Acceptance criteria (9.5)

- [ ] Four report screens wired to APIs  
- [ ] Date filters + role visibility  
- [ ] No exports  

---

## 10. Global toast / error matrix (modules)

| Event | Toast | Inline |
|-------|-------|--------|
| Create/update/delete success | ✅ success | — |
| Assignment/status success | ✅ success | — |
| Validation `422` | optional summary | ✅ fields |
| Forbidden `403` | ✅ error | and/or empty |
| Conflict `409` | ✅ error | — |
| Rate limit `429` | ✅ (existing interceptor) | — |
| `419` CSRF | ✅ + refresh CSRF | — |
| Load failure | optional | ✅ panel + retry |

---

## 11. Out of Scope (Milestone 9 overall)

- Automated frontend test suite (Phase 10)  
- Deployment (Phase 11)  
- Chart libraries / UI frameworks  
- Kanban / calendar / chat  
- Activity Logs / Remarks UI  
- Notifications / attachments  
- New backend endpoints or schema  
- Multi-org / organization settings  

---

## 12. Acceptance Criteria (design package + Phase 9.1)

- [x] `docs/MILESTONE_9_FRONTEND_MODULES.md` exists  
- [x] `decisions/Frontend-Modules.md` exists  
- [x] Companion docs synchronized  
- [x] Phase 9.1 implementation complete  
- [ ] Phase 9.2 implementation approved before coding  

---

## 13. Related documents

| Document | Use |
|----------|-----|
| [decisions/Frontend-Modules.md](../decisions/Frontend-Modules.md) | ADR |
| [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](MILESTONE_8_FRONTEND_FOUNDATION.md) | Shell prerequisite |
| [docs/MILESTONE_6_DASHBOARD.md](MILESTONE_6_DASHBOARD.md) | Dashboard API |
| [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Users API |
| [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) | Projects API |
| [docs/MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md) | Tasks API |
| [docs/MILESTONE_7_REPORTS.md](MILESTONE_7_REPORTS.md) | Reports API |
| [UI_PAGES.md](../UI_PAGES.md) | Page inventory |
| [ROADMAP.md](../ROADMAP.md) | Roadmap |
| [HANDOFF.md](../HANDOFF.md) | Handoff |
