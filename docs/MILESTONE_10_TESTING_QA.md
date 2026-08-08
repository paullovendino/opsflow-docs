# Phase 10 — Testing & QA

**Status:** ✅ **Complete** (10.1–10.3 + modal-navigation QA fix).  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-08

> Specification for **Phase 10 Testing & QA** (`opsflow-api` gap-fill + `opsflow-web` automated tests + manual QA).  
> Filename kept as `MILESTONE_10_TESTING_QA.md` for existing links. Roadmap **Milestone 10** is now **Product Enhancements** — see [MILESTONE_10_PRODUCT_ENHANCEMENTS.md](MILESTONE_10_PRODUCT_ENHANCEMENTS.md).  
> ADR: [decisions/Testing-QA.md](../decisions/Testing-QA.md)  
> Prerequisite: Milestone 8 complete · Milestone 9 complete (9.1–9.5 + post-ship UX/performance).  
> Backend APIs (Milestones 2–7) already have substantial PHPUnit Feature coverage — do **not** duplicate.  
> This document describes the **system that exists today**. Do not invent features, permissions, or tooling.

---

## 1. Goal

Validate the current OpsFlow product:

| Layer | What exists today |
|-------|-------------------|
| API | Laravel 13 + Sanctum SPA cookies + PostgreSQL; Users, Lookups, Projects, Members, Tasks, Dashboard, Reports |
| SPA | Vue 3 + TypeScript + Pinia + Vue Router + Axios + Tailwind; Dashboard, Users, Projects, Tasks, Reports |
| UX | List+modal CRUD (Users/Projects/Tasks); Show pages; Reports list/detail pages; skeletons; `AppProgressBar`; `useLookups` SPA-session cache |

Phase 10 is **quality**, not features. Success means regressions are caught, authz boundaries are proven, SPA contracts are tested, type-check/build stay green, and a repeatable manual QA path exists.

Implementation was approved 2026-08-08: Vitest + Vue Test Utils + happy-dom; backend gap-fill; frontend automated tests; manual browser QA; CSRF `419` where practical; remove scaffold `ExampleTest`. Deferred: Playwright/Cypress, GitHub Actions, shared PHPUnit actor trait.

---

## 2. Current testing baseline (inspected)

### 2.1 Backend (`opsflow-api`)

| Item | Actual state |
|------|----------------|
| Framework | **PHPUnit** `^12.5.12` via `php artisan test` — **not Pest** |
| Config | `phpunit.xml` → PostgreSQL `opsflow_testing`, `SESSION_DRIVER=cookie`, `RefreshDatabase` |
| Base class | `tests/TestCase.php` (empty; no shared acting-as / role helpers) |
| Factories | `UserFactory`, `ProjectFactory`, `TaskFactory` |
| Seeders (reusable in tests) | `RolesSeeder`, `DepartmentSeeder`, `JobTitleSeeder` |
| Auth pattern | Stateful SPA simulated with `Origin: http://localhost:5173`; after logout/guest follow-ups, `auth()->forgetGuards()` |
| Last known full suite | **215 tests passed** (Phase 10.1 gap-fill + existing M2–7 Feature/Unit; 2026-08-08) |
| CI | **None** (no `.github` workflows; deferred) |

**Existing Feature suites (do not rewrite):** Auth, inactive login, org foundation, Users CRUD/query/authz, Lookups, Project domain/CRUD/members/query/authz, Task domain/CRUD/assignment/query/authz/status, Dashboard API+authz, Reports list/detail/authz. Paths match [TESTING.md](../TESTING.md) / HANDOFF.

**Scaffold leftovers:** Removed `tests/Feature/ExampleTest.php` and `tests/Unit/ExampleTest.php`.

**Phase 10.1 gap-fill (implemented):** dedicated login **429**; CSRF **419** via `ApiExceptionRenderer` unit test (Laravel disables CSRF in `runningUnitTests()`, so a live cookie-mismatch Feature test is not practical); empty project/employee report list cases; no shared Admin/PM/Employee actor trait.

### 2.2 Frontend (`opsflow-web`)

| Item | Actual state |
|------|----------------|
| Test runner | **Vitest** + `@vue/test-utils` + `happy-dom` — colocated `src/**/*.spec.ts` |
| Scripts | `dev`, `build`, `build-only`, `preview`, `type-check`, **`test`**, **`test:watch`** |
| Quality today | `npm run test` (**69** tests / 26 files) + `npm run type-check` + `npm run build` green |
| CI | **None** (deferred) |

SPA contracts automated (mocked services, no live API): auth store bootstrap/login/logout; route guards + role `meta.roles`; `AppSidebar` role nav; `LoginView` inline `422`/`401`; HTTP `401`/`429`/`5xx` interceptors; `useLookups` cache/dedupe; `useUserList` / `useDashboard` loading/empty/error; shared UI; `AppProgressBar` quiet lookups; `viewKey` / `isModalAliasNavigation`. Full CRUD E2E remains **manual QA** (§9).

---

## 3. Phasing (implementation order, when approved)

| Phase | Focus | Status |
|-------|--------|--------|
| **10.1** | Backend gap-fill + regression | ✅ `php artisan test` **215** passed |
| **10.2** | Frontend unit / component tests | ✅ Vitest + VTU + happy-dom; `npm run test` **69** / 26 files |
| **10.3** | Manual QA / E2E checklist | ✅ Passed (auth, Users, Projects, Tasks, Reports + cross-module flow) |
| **10.4** | In-scope defect fixes | ✅ Global modal navigation remount/refetch fix (Users / Projects / Tasks) |

Automated browser E2E (Playwright/Cypress) remains **deferred**.

---

## 4. Authorization / role matrix (from code)

Derived from `UserPolicy`, `ProjectPolicy`, `TaskPolicy`, `DashboardPolicy`, `ReportPolicy`, `AppSidebar`, and router `meta.roles`. Do not invent extra permissions.

Legend: ✅ allowed · ❌ denied · ◐ scoped (owned / member / self)

| Capability | Administrator | Project Manager | Employee |
|------------|---------------|-----------------|----------|
| Login / logout / `/me` | ✅ | ✅ | ✅ |
| Dashboard (data scoped) | ✅ org-wide | ✅ org-wide | ◐ owned-or-member |
| Profile `/profile` | ✅ | ✅ | ✅ |
| Sidebar: Users | ✅ | ✅ | ❌ (Profile only) |
| Sidebar: Projects / Tasks / Reports | ✅ | ✅ | ✅ |
| Sidebar: Employee reports | ✅ | ✅ | ❌ |
| Sidebar: My report | — | — | ✅ (self) |
| Users list / search / filter / paginate | ✅ | ✅ read | ❌ |
| View any user `/users/:id` | ✅ | ✅ | ❌ (self via Profile) |
| User create / edit / delete / status | ✅ | ❌ | ❌ |
| Lookups `/api/v1/lookups/*` | ✅ | ✅ | ✅ authenticated |
| Projects list / filter / paginate | ✅ all | ✅ all | ◐ owned-or-member |
| Project Show `/projects/:id` | ✅ | ✅ | ◐ owned-or-member |
| Project create / edit / delete / status | ✅ | ✅ | ❌ |
| List members | ✅ | ✅ | ◐ if can view project |
| Add / remove members | ✅ | ✅ | ❌ |
| Tasks list / filter / paginate | ✅ all | ✅ all | ◐ accessible project |
| Task Show `/tasks/:id` | ✅ | ✅ | ◐ accessible project |
| Task create / edit / delete / assignment | ✅ | ✅ | ❌ |
| Task status patch | ✅ | ✅ | ◐ assigned to self + accessible project |
| Project reports list / detail | ✅ all | ✅ all | ◐ owned-or-member |
| Employee reports list | ✅ | ✅ | ❌ |
| Employee report detail | ✅ any | ✅ any | ◐ self only |

**SPA `meta.roles`:** `users.index` Admin+PM; `users.create`/`edit` Admin only; `projects.create`/`edit` and `tasks.create`/`edit` Admin+PM; `reports.employees.index` Admin+PM. Wrong role → `/403`. Unauthenticated → `/login?redirect=…`. Authenticated guest `/login` → `/dashboard`.

---

## 5. Backend testing design (10.1)

### Principles

- Keep PHPUnit Feature tests; match existing envelope assertions (`success`, `message`, `data`, `errors`, `meta`).
- Reuse `User::factory()`, `Project::factory()`, `Task::factory()`, and role/dept/job-title seeders.
- Prefer **gap-fill** over parallel suites that repeat CRUD/authz.
- Optional `CreatesOpsFlowActors` trait only if it reduces duplication — not required to start.

### Add / extend (gaps)

| Priority | Test | Why |
|----------|------|-----|
| P0 | Login throttle → HTTP **429** after 5 attempts per `email\|ip` | ✅ `AuthenticationTest::test_login_is_rate_limited_after_five_attempts` |
| P0 | Re-run **full** existing Feature suite on `opsflow_testing` | ✅ **215** passed (2026-08-08) |
| P1 | CSRF / `419` **if practical** with Sanctum cookie test client | ✅ Practical path: `ApiExceptionRendererTest` unit test. Live Feature CSRF mismatch is not reliable — Laravel skips CSRF while `runningUnitTests()`. |
| P1 | Envelope spot-check only where a status code is missing (`401`/`403`/`404`/`409`/`422`) | Existing suites cover these; no extra parallel suite |
| P2 | Reports **empty dataset** if not already covered | ✅ `ProjectReportApiTest` / `EmployeeReportApiTest` empty list cases |
| P2 | Retire or quarantine Laravel `ExampleTest` placeholders | ✅ Removed Feature + Unit `ExampleTest` |

### Do not re-implement

Full Users/Projects/Tasks/Dashboard/Reports CRUD, query, assignment, status, and Admin/PM/Employee matrices already exist.

### Authentication strategy (API)

Already covered: success, invalid credentials, validation, `/me`, logout, already-authenticated login, inactive account (`UserDomainFoundationTest`). Add **429 throttle**. Do not duplicate inactive-login in two files. Continue `Origin: http://localhost:5173`. Do not switch to bearer tokens.

### Optional thin cross-module API test

Admin creates user → PM/Admin creates project → add member → create/assign task → Employee patches status → dashboard/report counts. Only if it stays small. Not a substitute for existing suites.

---

## 6. Frontend testing design (10.2)

### Installed stack (approved)

| Package | Role |
|---------|------|
| `vitest` | Runner (Vite-native; matches current Vite 8 app) |
| `@vue/test-utils` | Component mount |
| `happy-dom` | DOM |

`@pinia/testing` was not needed — tests use real Pinia + mocked `*Service.ts`.

**Not installed (deferred):** Playwright, Cypress, Storybook, MSW. Prefer **mocked Axios/services** for unit/component tests; use **manual QA** for real Sanctum cookies.

Scripts: `npm run test` / `npm run test:watch`. `type-check` and `build` unchanged.

### Test layout

Colocated `src/**/*.spec.ts`. Vite `test` block in `vite.config.ts` (`environment: happy-dom`, `setupFiles: src/test/setup.ts`). Pure helpers extracted for testability: `utils/modalRoutes.ts`, `utils/httpProgress.ts`, `utils/listLoading.ts`; `resetLookupsCache()` for lookup cache isolation.

### Shared UI components

| Component | Contract to test |
|-----------|------------------|
| `AppButton` | `loading` → `aria-busy`, disabled, slot label |
| `AppInput` / `AppSelect` | label, value emit, error text, disabled |
| `AppModal` | open/close, Escape, does not render when closed |
| `AppConfirmDialog` | confirm/cancel callbacks; loading on confirm |
| `AppTable` | header + row slots; empty slot |
| `AppPagination` | prev/next disabled at bounds; page emit |
| `AppSearch` | input emit (fake timers only if debounce is the contract) |
| `StatusBadge` | status + `kind=priority` at a coarse level |
| Skeletons (`AppSkeleton`, table/card/detail/report) | render `aria-busy`; present when loading flag true |
| `AppProgressBar` | visible when `ui` global loading; hidden when idle |
| `AppEmptyState` | copy + optional CTA; **must not** show with skeleton in parent views |

Do not snapshot entire Tailwind class strings unless a visual regression tool is approved.

### Auth / router

| Case | Expectation |
|------|-------------|
| Guest → protected route | redirect `login` + `redirect` query |
| Authenticated → `/login` | redirect `dashboard` |
| Employee → `/users` | `/403` |
| PM → `/users/create` | `/403` |
| Admin → `/users/create` | allowed (list+modal) |
| Employee → `/reports/employees` | `/403` |
| Bootstrap spinner | `App.vue` shows spinner until `isBootstrapped` |
| Logout | clears auth; subsequent nav is guest |
| Login `422` / invalid creds | **inline** errors on `LoginView` (not auto `/403`) |
| API `401` interceptor | clears auth; redirects to login when not on guest route |

Mock `authService` / `http`; do not require a live Laravel server for these tests.

### Module views (mocked services)

- **Dashboard:** skeleton on first load; cards/bars/recent on success; empty recent ≠ zero stats; error + retry.
- **Users:** skeleton vs empty vs rows; search/filter/page call `userService.list`; Create/Edit `UserFormDialog`; `422` field errors; confirm status/delete; PM no mutate; Employee never reaches list (guard). Lookups: see §7.
- **Projects:** same list+modal pattern; Show members add/remove confirm; Employee hides mutate; status modal.
- **Tasks:** table not Kanban; Create/Edit/View modals; `TaskAssignmentDialog`; status; `ProjectTasksPanel`; Employee status only when assigned to self.
- **Reports:** date filters update query; project/employee list+detail; report/table skeletons; empty ≠ loading; Employee My report vs Admin/PM employee list.

### What not to over-test

Exact pixel/responsive CSS (manual §9); Axios internals beyond 401/429/5xx toast contracts; private Vue internals; chart libraries (none installed).

---

## 7. Performance / UX testing strategy

Test **user-facing contracts**, not timing flakiness.

| Contract | How |
|----------|-----|
| Lookup in-flight dedupe | Unit-test `ensureLookups()` with mocked `lookupService`; concurrent calls → **one** `listRoles` / `listDepartments` / `listJobTitles` |
| Lookup SPA-session cache | Second `ensureLookups()` after success → **zero** extra HTTP; `force: true` refetches |
| Create User does not refetch lookups if cached | After cache warm, opening Create/Edit does not call lookup endpoints again |
| Modal alias does not remount list | Test `isModalAliasNavigation` + `viewKey` mapping as pure logic; optional: list `onMounted` fetch count stays `1` on `/users` → `/users/create` |
| Skeletons on **initial** load | `isLoading && items.length === 0` → skeleton; not empty state |
| Soft refresh | `isLoading && items.length > 0` → prior rows remain; opacity/`aria-busy`; **not** empty |
| Progress bar quiet lookups | `/api/v1/lookups/` does not increment HTTP progress; other `/api/v1/*` does |
| Modal alias skips route progress | `isModalAliasNavigation` true → `setRouteLoading` not started |
| Button loading | `AppButton` `loading` during submit |
| Auth bootstrap | full-screen spinner until `/me` settles |

Do **not** assert request waterfalls in milliseconds or Pinia/`localStorage` lookup persistence (cache is in-memory composable only).

---

## 8. CRUD testing strategy

| Layer | Approach |
|-------|----------|
| API | Rely on existing Feature suites; add only gaps (§5) |
| SPA | Component/view tests with **mocked** `*Service.ts`; assert dialogs open, payloads passed to service, toasts/errors, list reload after success |
| Modal vs page | Users/Projects/Tasks Create/Edit/View from list = **modals**; Show = **pages**; Reports = **pages** |
| Validation | Client-side required fields where implemented + API `422` mapped to field errors |
| Confirms | `AppConfirmDialog` for delete/status/member remove — cancel does not call mutate |

---

## 9. Manual QA / E2E checklist (10.3) — ✅ passed

Run against **local** `opsflow-api` + `opsflow-web` (Sanctum stateful domain). Use seeded roles + Admin / PM / Employee users.

**Result (2026-08-08):** manual browser QA **passed**. Authentication, Users, Projects, Tasks, and Reports verified. Cross-module flow verified: Admin → Project → Member → Task → Employee status → Reports.

### Auth

- [x] Login success → `/dashboard`
- [x] Invalid credentials → inline error, stay on `/login`
- [x] Inactive user → `403` message, not logged in
- [x] Logout → guest; `/dashboard` redirects to login
- [x] Refresh while logged in → session restored via `/me` (bootstrap spinner then shell)
- [x] Hard refresh on `/projects/:id` stays authenticated (cookie)

### Navigation / roles

- [x] Admin: Users, Projects, Tasks, Reports, Employee reports, Profile
- [x] PM: Users (read), Projects/Tasks mutate, Employee reports; no user create
- [x] Employee: no Users nav; My report; Projects/Tasks scoped; `/users` and `/reports/employees` → 403
- [x] Deep link `/users/create` as Employee/PM → 403; as Admin → list + create modal

### Dashboard / Users / Projects / Tasks / Reports

- [x] Dashboard: skeleton then data; empty recent vs zero stats; error + retry
- [x] Users: search/filters/pagination (server); create/edit/view modals; show/profile pages; status/delete confirms; Create does not remount list or flash full-page skeleton; lookups load once; progress bar does not look like a full reload on lookup-only traffic
- [x] Projects: list+modals; `/projects/:id` workspace; status modal; member add / duplicate `409` / remove confirm; Employee list/view only
- [x] Tasks: table (not Kanban); create/edit/view modals; `/tasks/:id`; assignment; status; `ProjectTasksPanel`; Employee status only when assigned to self
- [x] Reports: `/reports` → projects; date filters; empty + error+retry; Tailwind bars (no chart lib); Admin/PM employee list+detail; Employee My report (self); other employee `403`

### Responsive / loading

- [x] Mobile: sidebar overlay; tables → cards below `md`; modals/confirms usable
- [x] Initial load = skeleton; refresh = soft opacity, not empty
- [x] Empty copy only when not loading and no error; friendly retry (no raw Axios dumps)

### Cross-module workflow (supported today)

1. Admin creates **Employee** user (active)
2. Admin/PM creates **Project**
3. Add employee as **member**
4. Create **Task** on that project
5. **Assign** employee
6. Employee logs in → sees project/task → changes **status**
7. Admin/PM: Dashboard + Project report + Employee report reflect work
8. Logout / login as other roles; confirm authz UI

**Verified.** Assignee must be the project owner or a **project member** before they are eligible (existing Task Assignment rule; unchanged).

---

## 10. Test data strategy

| Source | Use |
|--------|-----|
| `RolesSeeder` / `DepartmentSeeder` / `JobTitleSeeder` | Authz + lookup tests (already the pattern) |
| `UserFactory` (+ `inactive()` state) | Users; set `role_id` explicitly for Admin/PM/Employee |
| `ProjectFactory` / `TaskFactory` | Projects/tasks; set `created_by` / `assigned_to` / `project_id` |
| `RefreshDatabase` + PostgreSQL `opsflow_testing` | Isolation; **do not** use SQLite |
| SPA manual QA | Local `db:seed` + hand-made users or tinker/factories; not production data |
| Frontend unit tests | **No real DB**; mock services |

Do **not** add Redis, localStorage fixtures, or a second test DB engine. Optional later: factory states `administrator()` / `projectManager()` if duplication hurts — not a blocker. `phpunit.xml` still embeds DB credentials (known debt) — do not expand that unless tests require it.

---

## 11. Quality gates

**No CI exists today.** Gates are a **local / PR checklist** unless a future ADR adds GitHub Actions (out of Phase 10 default).

| Gate | Command | Required for Phase 10 complete |
|------|---------|--------------------------------|
| API regression | `cd opsflow-api && php artisan test` | ✅ **215** passed (2026-08-08) |
| SPA types | `cd opsflow-web && npm run type-check` | ✅ green |
| SPA production build | `cd opsflow-web && npm run build` | ✅ green |
| SPA unit/component | `npm run test` | ✅ **69** tests / 26 files |
| Manual QA | Checklist §9 | ✅ **Passed** |

Phase 10 is **complete**. Playwright/Cypress and GitHub Actions remain deferred. No CI is claimed.

---

## 12. Defect policy

| Finding | During Phase 10? |
|---------|------------------|
| Authz / security hole (API or guard mismatch) | **Fix** |
| Envelope / status-code regression | **Fix** |
| Broken happy-path CRUD / login | **Fix** |
| Loading/empty swapped (product bug) | **Fix** |
| Lookup duplicate requests / list remount regression | **Fix** (contract already shipped) |
| Cosmetic copy / spacing | Defer unless blocking QA |
| New features, Kanban, exports, chart libs, Remarks, Activity Logs | **Out of scope** |
| Deployment / prod infra | Milestone 11 |
| Large unrelated refactors | **No** |

---

## 13. Out of scope (Phase 10)

- New business features or API/schema
- New UI modules
- Deployment / production hosting / real CI unless separately approved
- Pest migration
- Playwright/Cypress unless approved
- Chart libraries, Kanban, exports, Activity Logs / Remarks
- Pinia/localStorage/Redis lookup caching (not how the app works)
- Replacing Sanctum SPA cookies

---

## 14. Acceptance criteria (Phase 10 complete)

- [x] This spec + [decisions/Testing-QA.md](../decisions/Testing-QA.md) exist
- [x] Companion docs mark Phase 10 Testing & QA **complete**; next is Milestone 10 Product Enhancements
- [x] Backend gap tests implemented and full `php artisan test` green (**215**)
- [x] Frontend test runner installed; coverage for §6–§7 contracts (`npm run test` **69** / 26 files)
- [x] Manual QA checklist executed and passed
- [x] Type-check + production build green
- [x] In-scope 10.3 defect fixed: global modal navigation (stable family `viewKey`; query preserved; no route progress on modal aliases)

### 14.1 Global modal navigation QA fix (complete)

**Root cause:** `AuthLayout` keyed `RouterView` with **path** on list index (`/users`) and **name** on Create/Edit (`users.index`). Opening a modal changed the key → list remount → `onMounted` refetch → skeleton / route+HTTP progress → lost search/filter/page query. Same pattern on Projects and Tasks.

**Final behavior (Users / Projects / Tasks):**

- Stable modal family `viewKey` (`authLayoutViewKey`): index + create + edit share `*.index`
- List remains mounted; no unnecessary list refetch when opening Create/Edit/View modal
- Existing search / filters / pagination query is preserved (`openModalAlias` / `listIndexLocation`)
- Modal navigation does **not** trigger page-level route progress (`shouldTrackRouteProgress`)
- Modal-specific API requests stay local (detail GET may use `quietProgress`); lookups remain quiet
- `useLookups` in-memory SPA-session cache + in-flight dedupe unchanged (not Pinia / localStorage / Redis)
- Real page navigation (Show, Reports, Dashboard, Profile) still uses route + HTTP progress

**Not changed:** authorization, assignment (assignee must be owner or project member), API contracts, schema, Reports as dedicated pages.

---

## 15. Decisions (locked 2026-08-08)

| Decision | Outcome |
|----------|---------|
| Vitest + Vue Test Utils + happy-dom | ✅ Approved and installed |
| Automated E2E (Playwright/Cypress) | ❌ Deferred — manual checklist remains first-class |
| GitHub Actions CI | ❌ Deferred |
| CSRF `419` | ✅ Practical path: `ApiExceptionRenderer` unit test (not a live Feature CSRF mismatch) |
| Shared PHPUnit actor trait | ❌ Deferred unless duplication warrants it |
| Remove Laravel `ExampleTest` | ✅ Removed |

---

## 16. Related documents

| Document | Use |
|----------|-----|
| [decisions/Testing-QA.md](../decisions/Testing-QA.md) | ADR |
| [TESTING.md](../TESTING.md) | How to run existing API tests |
| [HANDOFF.md](../HANDOFF.md) | Session start |
| [ROADMAP.md](../ROADMAP.md) | Phase order (Phase 10 complete → Milestone 10 Product Enhancements → Milestone 11 Deployment) |
| [docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md](MILESTONE_10_PRODUCT_ENHANCEMENTS.md) | Next milestone (planned only) |
| [docs/MILESTONE_9_FRONTEND_MODULES.md](MILESTONE_9_FRONTEND_MODULES.md) | SPA contracts |
| [UI_PAGES.md](../UI_PAGES.md) | Routes / UX |
| [API_SPECIFICATION.md](../API_SPECIFICATION.md) | HTTP contracts |
| [AUTHENTICATION.md](../AUTHENTICATION.md) | Sanctum SPA auth |


