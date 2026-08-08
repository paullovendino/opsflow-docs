# Milestone 9 — Frontend Modules

**Status:** ✅ Complete — Phases 9.1–9.5 implemented  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-08

> Implementation specification for Milestone 9 (`opsflow-web` feature UIs).  
> ADR: [decisions/Frontend-Modules.md](../decisions/Frontend-Modules.md)  
> Prerequisite: Milestone 8 complete (auth shell, Axios/Sanctum, layouts, shared UI primitives)  
> Backend APIs: Milestones 3–7 complete — **consume only**; no new API/schema in Milestone 9  
> Full companion-docs sync completed after Phase 9.5, including post-ship CRUD/loading/lookup UX

---

## 1. Goal

Ship OpsFlow’s authenticated **product modules** on top of the Milestone 8 foundation:

| Phase | Module | Primary APIs | Status |
|-------|--------|--------------|--------|
| **9.1** | Dashboard UI | `GET /api/v1/dashboard` | ✅ Implemented |
| **9.2** | User Management | `/api/v1/users`, `/api/v1/lookups/*` | ✅ Implemented |
| **9.3** | Project Management | `/api/v1/projects`, members | ✅ Implemented |
| **9.4** | Task Management | `/api/v1/tasks`, assignment, status | ✅ Implemented |
| **9.5** | Reports | `/api/v1/reports/projects`, `/api/v1/reports/employees` | ✅ Implemented |

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
| Create/Edit UX | **Users, Projects, Tasks:** list + **modal** Create/Edit (`AppModal`). Show remains a **page** for deep links (`/users/:id`, `/projects/:id`, `/tasks/:id`). Confirms stay `AppConfirmDialog`. |
| Tasks board | **Table only**. Kanban deferred (roadmap v1.2). |
| Pinia | Keep global `auth` + `ui`. Module list/filter state lives in **page composables**. Lookup cache is **not** Pinia — it lives in `useLookups` module-level refs. |
| Services | One `*Service.ts` per domain under `services/` calling shared `http` |
| CSRF | Existing Axios `withCredentials` + `withXSRFToken`. Mutating calls after login reuse cookie; on HTTP `419`, toast + `fetchCsrfCookie()` once + optional single retry (document in service helper). |
| Toasts | Success toasts on create/update/delete/status/assignment. Inline field errors for `422`. `403` toast (or empty+message); do **not** auto-route to `/403` for API denials. |
| Role-aware nav | Sidebar links enabled; hide/disable by role (see §3) |
| Backend | **No** new endpoints, migrations, or contract changes |
| Testing / Deployment | **Out of Milestone 9** → Phase 10 Testing & QA · Phase 11 Deployment |

### Role-aware navigation (sidebar)

| Nav item | Administrator | Project Manager | Employee |
|----------|---------------|-----------------|----------|
| Dashboard | ✅ | ✅ | ✅ |
| Users | ✅ | ✅ (read) | ❌ (use Profile only) |
| Projects | ✅ | ✅ | ✅ |
| Tasks | ✅ | ✅ | ✅ |
| Reports | ✅ | ✅ | ✅ (API scopes apply) |
| Employee reports | ✅ | ✅ | ❌ |
| My report | — | — | ✅ (self employee detail) |
| Profile | ✅ | ✅ | ✅ |

No “Coming later” placeholders for M9 modules once Milestone 9 is complete.

---

## 3. Target folder structure (extends Milestone 8)

```text
opsflow-web/src/
├── components/
│   ├── layout/          # AppSidebar, AppTopbar
│   └── ui/              # AppButton, AppInput, AppSpinner, AppEmptyState, AppToastHost
│                        # + AppTable, AppBadge, AppPagination, AppSelect, AppTextarea
│                        # + AppConfirmDialog, AppModal, AppDropdownMenu
│                        # + AppPageHeader, AppFormSection, AppFormActions, AppSearch, AppFilterBar
│                        # + StatusBadge (status + priority kinds)
│                        # + AppSkeleton, AppTableSkeleton, AppCardSkeleton, AppDetailSkeleton
│                        # + AppReportSkeleton, AppProgressBar
├── composables/         # useAuth, useToast, useDashboard, useUserList, useLookups, useProjectList, useTaskList, …
├── layouts/             # GuestLayout, AuthLayout
├── modules/
│   ├── dashboard/
│   │   ├── components/
│   │   └── views/       # DashboardView.vue
│   ├── users/
│   │   ├── components/  # UserForm, UserFormDialog, UserDetailDialog, UserDetailPanel, UserActionsMenu, …
│   │   └── views/       # UserListView, UserShowView, ProfileView
│   ├── projects/        # Phase 9.3 ✅
│   ├── tasks/           # Phase 9.4 ✅
│   └── reports/         # Phase 9.5 ✅
├── router/
├── services/            # http, authService, dashboardService, userService, lookupService, projectService, taskService, reportService
├── stores/              # auth, ui
├── types/
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
| `AppConfirmDialog` | Title, body, confirm/cancel; loading on confirm; focus trap |
| `AppModal` | General modal shell (CRUD forms/details); focus trap + Escape + scroll lock |
| `AppDropdownMenu` | Teleported fixed menu (no overflow clipping); keyboard nav |
| `AppPageHeader` | Page title + description + actions slot |
| `AppFormSection` / `AppFormActions` | Form grouping + footer actions |
| `AppSearch` / `AppFilterBar` | List search + filter container |
| `AppStatCard` | Label + value (+ optional hint) for dashboard/report cards |
| `AppBarChart` | Horizontal bars from `{ label, value }[]` (Tailwind only; alias: dashboard status bars) |
| `StatusBadge` | Map API enums to colors (status + **priority** via `kind=priority`) |
| `AppSkeleton` | Primitive pulse placeholder |
| `AppTableSkeleton` | Table + mobile card placeholders |
| `AppCardSkeleton` | Stat-card placeholders |
| `AppDetailSkeleton` | Show/detail + compact modal placeholders |
| `AppReportSkeleton` | Report detail placeholders (cards + bars + table) |
| `AppProgressBar` | Top bar for route navigation + page-level HTTP |

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
| Initial load | Module skeleton (`UserListSkeleton`, `ProjectListSkeleton`, `TaskListSkeleton`, `DashboardSkeleton`, `AppReportSkeleton` / `AppDetailSkeleton`) — **not** a full-page block for every API call |
| Soft refresh | Keep existing rows/detail visible; `opacity-60` + `aria-busy` while refetching |
| Modal/detail load | Compact `AppDetailSkeleton` inside `AppModal` until the record arrives |
| Button submit | Inline `AppButton` loading |
| Auth bootstrap | Full-screen `AppSpinner` in `App.vue` until `/me` bootstrap completes |
| Global progress | `AppProgressBar` for route changes and page-level HTTP; **`/api/v1/lookups/*` excluded**; Create/Edit modal alias navigations do not start route progress |
| Empty | `AppEmptyState` — **distinct from loading** (empty only when not loading and no error) |
| Error (load fail) | Inline error panel + retry; toast for unexpected 5xx |
| `422` | Field errors on forms |
| Success | Toast (`ui` store) |
| Responsive | Filters stack vertically on mobile; tables → card list below `md` |

### CRUD modal pattern (as shipped)

The SPA prefers **modals/dialogs** for lightweight CRUD so the underlying list stays mounted:

| Interaction | Implementation |
|-------------|----------------|
| User Create/Edit | `UserFormDialog` (`AppModal`) on `UserListView` |
| User View (from list) | `UserDetailDialog`; `/users/:id` and `/profile` remain **pages** |
| Project Create/Edit | `ProjectFormDialog` on `ProjectListView`; Show page can also open the same edit modal in place |
| Project View (from list) | `ProjectDetailDialog`; `/projects/:id` remains the workspace **page** |
| Task Create/Edit | `TaskFormDialog` on `TaskListView` |
| Task View (from list) | `TaskDetailDialog`; `/tasks/:id` remains a **page** |
| Task assignment | `TaskAssignmentDialog` / detail panel |
| Status / delete / member remove | `AppConfirmDialog` or small `AppModal` (status patch) |

Reports remain **dedicated list + detail pages** (not modal CRUD). Do not rewrite every screen as a modal if the implementation uses a page.

### Stable modal route handling

Create/Edit alias routes render the **same list component** as index:

- `/users/create`, `/users/:id/edit` → `UserListView`
- `/projects/create`, `/projects/:id/edit` → `ProjectListView`
- `/tasks/create`, `/tasks/:id/edit` → `TaskListView`

`AuthLayout` uses a stable `viewKey` (`users.index` / `projects.index` / `tasks.index`) so opening a Create/Edit alias does **not** remount the list. Router guards skip `AppProgressBar` route progress for those family navigations (`isModalAliasNavigation`).

### Lookup caching (`useLookups`)

**Root cause:** each `useLookups()` instance previously owned independent refs and fetched roles / departments / job titles on mount. Multiple mounted components (list + form dialog) therefore hit `/api/v1/lookups/*` repeatedly. Modal route remounts made this worse.

**Current implementation:**

- Shared **module-level** refs in `composables/useLookups.ts` (SPA-session, in-memory)
- `ensureLookups()` + in-flight promise **deduplication**
- Cache survives component unmount/remount; resets on full browser reload
- Explicit refresh via `load()` / `ensureLookups({ force: true })` where used
- **Not** Pinia, **not** `localStorage`, **not** Redis/server-side cache

Users list **search / filters / pagination / sorting remain server-side**. Only reference lookup data is cached. Create/Edit forms reuse cached Roles / Departments / Job Titles; if lookups are not yet available, selects show a loading placeholder.

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

**Status:** ✅ Implemented (including post-ship UX polish: modals, teleported actions menu, stable Clear control, lookup cache)

### Scope

- User list with search, filters, sort, pagination  
- Create / Edit / View via **modals** on the list  
- Status activate/deactivate with confirm  
- Soft delete with confirm (existing `DELETE /api/v1/users/{id}`)  
- Lookups for role / department / job title selects  
- Profile page + `/users/:id` deep-link show page  

### Architecture

```text
UserListView
  ├─ useUserList() → userService.list
  ├─ useLookups() → lookupService
  ├─ UserFormDialog (AppModal + UserForm) → create / update
  ├─ UserDetailDialog (AppModal + UserDetailPanel) → view
  └─ AppConfirmDialog → status / delete

UserShowView / ProfileView → userService.show (deep-link pages)
```

### Routes

| Path | Name | Component / behavior | Authz notes |
|------|------|----------------------|-------------|
| `/users` | `users.index` | `UserListView` | Admin, PM |
| `/users/create` | `users.create` | `UserListView` + Create modal | Admin |
| `/users/:id/edit` | `users.edit` | `UserListView` + Edit modal | Admin |
| `/users/:id` | `users.show` | `UserShowView` page | Admin/PM; Employee own id (API) |
| `/profile` | `profile` | `ProfileView` → show self | Authenticated |

Route order: `users/:id/edit` is registered **before** `users/:id`.

### Components / views

- Views: `UserListView`, `UserShowView`, `ProfileView` (no dedicated Create/Edit page components)
- Module: `UserForm`, `UserFormDialog`, `UserDetailDialog`, `UserDetailPanel`, `UserActionsMenu`, `UserListSkeleton`
- Shared: `AppTable`, `AppPagination`, `AppSearch`, `AppFilterBar`, `AppConfirmDialog`, `AppModal`, `AppDropdownMenu`, `AppPageHeader`, `AppFormSection`, `AppFormActions`, `AppSelect`, `AppTextarea`, `AppBadge`, `StatusBadge`, `AppEmptyState`, `AppButton`

### Locked UX (as shipped)

| Topic | Choice |
|-------|--------|
| Create/Edit | **Modals** (`AppModal` + `UserFormDialog`) on the Users list; create/edit routes open list + dialog |
| View | List opens **detail modal**; `/users/:id` and `/profile` remain pages for deep links |
| Table columns | Name, email, role, department, job title, status, last login, actions |
| Search | Debounced `search` query param |
| Filters | `role_id`, `department_id`, `job_title_id`, `status` |
| Clear filters | Always visible; **disabled** when idle (no layout shift) |
| Actions menu | Teleported `AppDropdownMenu` (fixed position; flips when needed; Escape / outside click / arrow keys) |
| Pagination | Server `meta`; `AppPagination` |
| Status | Badge + Admin actions via confirm → `PATCH .../status` |
| Delete | Admin confirm → `DELETE` soft delete |
| Employee | No Users nav; Profile only |
| Responsive | Filters stack; table → cards below `md` |

### Services / types / composables

- `userService.ts`, `lookupService.ts`  
- `types/user.ts`, `types/lookup.ts`  
- `useUserList.ts`, `useLookups.ts` (shared SPA-session lookup cache + in-flight dedupe)  
- **No** dedicated Pinia users store; users list remains server-side (search/filters/pagination/sort)  

### UX states

| State | Behavior |
|-------|----------|
| Initial load | `UserListSkeleton` |
| Refresh with rows | Soft opacity + `aria-busy` |
| Empty | “No users yet” / filter empty with Clear CTA |
| Form `422` | Inline field errors in `UserForm` |
| Success | Toast; list reload; create/edit may open detail modal |
| Load error | Inline retry (no raw Axios errors) |
| Confirm | `AppConfirmDialog` with focus trap / body scroll lock |

### Out of scope (9.2)

- Role/Department/Job Title CRUD screens  
- Invitation emails / password reset  
- Avatar upload  

### Acceptance criteria (9.2)

- [x] List query parity with API  
- [x] Create/Edit/Show/Profile work with authz  
- [x] Status + delete confirms  
- [x] Lookups populate selects  
- [x] Modal CRUD + teleported actions menu + stable Clear control  
- [x] `npm run type-check` + `npm run build` green  

---

## 7. Phase 9.3 — Project Management

**Status:** ✅ Implemented (Create/Edit later revised from dedicated pages to **list modals**, matching Users/Tasks)

### Scope

- Project list (search/filters/sort/pagination)  
- Create / Edit / View via **modals** on the list  
- `/projects/:id` Show **page** (workspace: info + members + tasks)  
- Status patch  
- Members on Show: list, add, remove  
- Show integrates `ProjectTasksPanel` (filled in Phase 9.4)

### Architecture

```text
ProjectListView
  ├─ useProjectList() → projectService.list
  ├─ ProjectFormDialog (AppModal + ProjectForm) → create / update
  ├─ ProjectDetailDialog (AppModal) → view from list
  └─ AppConfirmDialog / AppModal → delete / status

ProjectShowView → projectService.show (workspace page)
  ├─ ProjectFormDialog → in-place edit
  ├─ ProjectMembersPanel → list / add / remove confirm
  └─ ProjectTasksPanel → project tasks (Phase 9.4)
```

### Routes

| Path | Name | Component / behavior |
|------|------|----------------------|
| `/projects` | `projects.index` | `ProjectListView` |
| `/projects/create` | `projects.create` | `ProjectListView` + Create modal |
| `/projects/:id/edit` | `projects.edit` | `ProjectListView` + Edit modal |
| `/projects/:id` | `projects.show` | `ProjectShowView` workspace page |

Route order: `projects/:id/edit` is registered **before** `projects/:id`.

### Locked UX (as shipped)

| Topic | Choice |
|-------|--------|
| Create/Edit | **Modals** (`AppModal` + `ProjectFormDialog`) on the Projects list; create/edit routes keep the list mounted (stable `viewKey`) |
| View | List opens **detail dialog**; `/projects/:id` remains the Show **page** |
| Status badge | ProjectStatus palette; status patch via small `AppModal` |
| Members | Section on **Show** page |
| Add member | **Inline panel** on Show (user select + Add) — not a separate route |
| Remove member | `AppConfirmDialog` → `DELETE .../members/{user}` |
| Duplicate member | Surface API `409` message via toast/inline |
| Tasks on Show | `ProjectTasksPanel` (Phase 9.4) |
| Employee | List/view accessible only; no create/edit/members mutate |

### Components

- Views: `ProjectListView`, `ProjectShowView` (no dedicated Create/Edit page components)
- Module: `ProjectForm`, `ProjectFormDialog`, `ProjectDetailDialog`, `ProjectActionsMenu`, `ProjectMembersPanel`, `ProjectTasksPanel`, `ProjectListSkeleton`

### Services / composables

- `projectService.ts` (CRUD, status, members)
- `useProjectList.ts`

### Out of scope (9.3)

- Project templates, files, remarks  

### Acceptance criteria (9.3)

- [x] Full CRUD + status + members UX against API  
- [x] Authz-aligned actions hidden/disabled in UI (API remains source of truth)  
- [x] Modal Create/Edit/View on list; Show workspace page  

---

## 8. Phase 9.4 — Task Management

**Status:** ✅ Implemented

### Scope

- Task list (search/filters/pagination)  
- Create / Edit / View via **modals** on the list  
- `/tasks/:id` page for deep links  
- Assignment patch · status patch · soft delete  
- Integrated into Project Show via `ProjectTasksPanel`  

### Architecture

```text
TaskListView
  ├─ useTaskList() → taskService.list
  ├─ TaskFormDialog (AppModal + TaskForm) → create / update
  ├─ TaskDetailDialog (AppModal + TaskDetailPanel) → view
  ├─ TaskAssignmentDialog → assignment patch
  └─ TaskActionsMenu → status / assignment / delete

TaskShowView → taskService.show (deep-link page)
ProjectShowView → ProjectTasksPanel
```

### Locked UX (as shipped)

| Topic | Choice |
|-------|--------|
| Layout | **Table** (not Kanban); no chart library |
| Create/Edit | **Modals** (`AppModal` + `TaskFormDialog`) on the Tasks list; create/edit routes open list + dialog (Users pattern) |
| View | List opens **detail modal**; `/tasks/:id` remains a page for deep links |
| Assignment | `TaskAssignmentDialog` / detail panel → `PATCH .../assignment` |
| Status | → `PATCH .../status` (any `TaskStatus` per API) |
| Priority | `StatusBadge` with `kind=priority` |
| Filters | status, priority, project (plus search); pagination |
| Authz | Admin/PM mutate; Employee list/view + status only when assigned to self |

### Routes

| Path | Name | Component / behavior |
|------|------|----------------------|
| `/tasks` | `tasks.index` | `TaskListView` |
| `/tasks/create` | `tasks.create` | `TaskListView` + Create modal |
| `/tasks/:id/edit` | `tasks.edit` | `TaskListView` + Edit modal |
| `/tasks/:id` | `tasks.show` | Deep-link show page |

Route order: `tasks/:id/edit` is registered **before** `tasks/:id`.

### Components

- Views: `TaskListView`, `TaskShowView`; no dedicated Create/Edit page components  
- Module: `TaskForm`, `TaskFormDialog`, `TaskDetailDialog`, `TaskDetailPanel`, `TaskAssignmentDialog`, `TaskActionsMenu`, `TaskListSkeleton`  
- Project integration: `ProjectTasksPanel`  

### Services / composables

- `taskService.ts`  
- `useTaskList.ts`  

### Employee UX

- List/view scoped by API  
- Status change only when assigned to self (hide/disable otherwise; honor API `403`)  
- No create/edit/delete/assignment mutate  

### Out of scope (9.4)

- Kanban, calendar, time tracking, bulk edit  

### Acceptance criteria (9.4)

- [x] Table CRUD + assignment + status  
- [x] Priority/status badges  
- [x] Role-appropriate controls  
- [x] Modal Create/Edit/View (Users pattern)  

---

## 9. Phase 9.5 — Reports

**Status:** ✅ Implemented

### Scope

- Project reports list + detail  
- Employee reports list + detail (Admin/PM list; Employee self detail via My report)  
- Optional `from_date` / `to_date` filters  
- Cards + Tailwind bars (reuse Dashboard components)  

### Routes

| Path | Name |
|------|------|
| `/reports` | redirect → project reports |
| `/reports/projects` | `reports.projects.index` |
| `/reports/projects/:id` | `reports.projects.show` |
| `/reports/employees` | `reports.employees.index` |
| `/reports/employees/:id` | `reports.employees.show` |

### Locked UX (as shipped)

| Topic | Choice |
|-------|--------|
| Date filters | `ReportDateFilters` — inclusive `from_date` / `to_date` |
| List | Project + Employee summary lists (`AppTable` + mobile cards) |
| Detail | Stat cards + Tailwind bars |
| Charts | Reuse `DashboardStatCard` / `DashboardStatusBar` only (**no chart library**) |
| Loading | Initial: `AppTableSkeleton` / `AppReportSkeleton`; soft refresh keeps prior data (`opacity-60`) |
| Empty / error | Distinct empty state; inline retry on load failure |
| Employee nav | Employee reports list Admin/PM only; Employee self detail via **My report** |
| Exports | **None** |

### Components

- Views: `ProjectReportsListView`, `ProjectReportShowView`, `EmployeeReportsListView`, `EmployeeReportShowView`
- `ReportDateFilters`; reuse Dashboard stat/bar components; `AppTableSkeleton` / `AppReportSkeleton`

### Services

- `reportService.ts`

### Out of scope (9.5)

- PDF/CSV export, scheduled reports, custom builders, Activity Log reports, chart libraries  

### Acceptance criteria (9.5)

- [x] Four report screens wired to APIs (+ `/reports` redirect)  
- [x] Date filters + role visibility  
- [x] No exports  

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

- Automated frontend test suite (Phase 10 — Testing & QA)  
- Deployment (Phase 11)  
- Chart libraries / UI frameworks  
- Kanban / calendar / chat  
- Activity Logs / Remarks UI  
- Notifications / attachments  
- New backend endpoints or schema  
- Multi-org / organization settings  

---

## 12. Acceptance Criteria (Milestone 9 complete)

- [x] `docs/MILESTONE_9_FRONTEND_MODULES.md` exists  
- [x] `decisions/Frontend-Modules.md` exists  
- [x] Companion docs synchronized (full sync after 9.5 + post-ship UX/performance)  
- [x] Phase 9.1 implementation complete  
- [x] Phase 9.2 implementation complete  
- [x] Phase 9.3 implementation complete (Projects Create/Edit locked as **modals**; Show page)  
- [x] Phase 9.4 implementation complete (Tasks Create/Edit/View locked as **modals**)  
- [x] Phase 9.5 implementation complete  
- [x] Sidebar has no “Coming later” for M9 modules  
- [x] Skeleton loading, `AppProgressBar`, lookup cache / dedupe, stable modal `viewKey` documented as shipped |

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
