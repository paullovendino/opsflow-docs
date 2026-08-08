# Decision: Frontend Modules

## Status

Accepted — ✅ Complete (Phases 9.1–9.5 implemented)

## Context

Milestone 8 delivered the OpsFlow SPA foundation (auth, shell, Axios/Sanctum, shared primitives). Backend Milestones 3–7 already expose Users, Projects, Tasks, Dashboard, and Reports APIs. Milestone 9 delivered all feature module UIs on top of that foundation.

Roadmap Phase 9 was previously “Testing.” Feature UIs landed before a meaningful frontend test pass. Milestone 9 is therefore **Frontend Modules**; Testing & QA and Deployment remain Phases 10–11.

Companion specification: [docs/MILESTONE_9_FRONTEND_MODULES.md](../docs/MILESTONE_9_FRONTEND_MODULES.md)

## Decision

### Milestone 9 is frontend-only

- Work in `opsflow-web`
- **No** new API endpoints, contract changes, migrations, or morph aliases
- Consume existing `/api/v1` resources under Sanctum session auth

### Phasing (mandatory order — all complete)

| Phase | Module | Status |
|-------|--------|--------|
| 9.1 | Dashboard UI | ✅ Implemented |
| 9.2 | User Management | ✅ Implemented |
| 9.3 | Project Management | ✅ Implemented |
| 9.4 | Task Management | ✅ Implemented |
| 9.5 | Reports | ✅ Implemented |

### Architecture

- Extend Milestone 8 layouts/router/http/auth/ui toast patterns
- Feature code under `src/modules/{dashboard,users,projects,tasks,reports}/`
- Domain HTTP in `services/*Service.ts`; types in `types/`
- Prefer **composables** for list/query state; keep Pinia limited to `auth` + `ui`
- Shared UI in `components/ui` (tables, badges, pagination, modals, dropdowns, filters, confirms)
- **No** UI framework; **no** chart library (Tailwind bars only)

### Locked product UX

| Area | Choice |
|------|--------|
| Landing | `/` → `/dashboard`; App Home placeholder removed in 9.1 |
| Dashboard | Stat cards + Tailwind status bars + recent work (✅) |
| Users | Table + filters + pagination; Create/Edit/View **modals** on list; show/profile **pages** for deep links; teleported actions menu; Clear always visible/disabled; status/delete confirms; `useLookups` SPA-session cache (✅) |
| Projects | Table + Create/Edit/View **modals** on list; `/projects/:id` Show **page** (members + `ProjectTasksPanel`); status badges (✅) |
| Tasks | **Table only** (no Kanban); Create/Edit/View **modals** on list (Users pattern); `/tasks/:id` deep-link page; assignment + status controls; priority via `StatusBadge` kind=priority (✅) |
| Reports | Dedicated list/detail **pages**; date filters; cards + Tailwind bars (reuse Dashboard stats/bars); no chart library; no exports (✅) |
| Confirms | `AppConfirmDialog` for delete/status/member remove |
| Loading | Shared skeletons + `AppProgressBar`; lookup HTTP excluded from global progress; soft refresh ≠ empty |
| Modal routes | Stable `viewKey` + `isModalAliasNavigation` for Users/Projects/Tasks Create/Edit aliases |
| Nav | Sidebar: Dashboard, Users (role), Projects, Tasks, Reports, Employee reports (Admin/PM) / My report (Employee), Profile |

### Roadmap renumbering

| Phase | Milestone |
|-------|-----------|
| **9** | **Frontend Modules** (this ADR) — ✅ complete |
| **10** | Testing & QA (formerly Phase 9) — next |
| **11** | Deployment (formerly Phase 10) |

### Explicitly out of scope

Kanban, chart libraries, exports, Activity Logs/Remarks UI, automated test suite as M9 deliverable, deployment, backend changes.

## Consequences

- Companion docs point next work at **Phase 10 — Testing & QA** (after approval); then **Phase 11 — Deployment**
- All M9 phases delivered (including post-ship CRUD/loading/lookup UX); do **not** implement Phase 10 until user explicitly approves
- Users Create/Edit UX was revised from dedicated pages to **list modals** during 9.2 polish
- Tasks Create/Edit UX was revised from dedicated pages to **list modals** during 9.4 (explicit user request) — current locked Tasks pattern
- Projects Create/Edit UX was later revised from dedicated pages to **list modals** (same pattern as Users/Tasks); `/projects/:id` remains the Show workspace **page**
- Lookup cache is composable module-level in-memory SPA-session + in-flight dedupe — **not** Pinia / localStorage / Redis
- Backend APIs were not changed for post-ship frontend UX/performance work; `opsflow-web` type-check and production build pass
- Frontend Foundation ADR remains valid; this ADR extends it for modules

## References

- [docs/MILESTONE_9_FRONTEND_MODULES.md](../docs/MILESTONE_9_FRONTEND_MODULES.md)
- [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md)
- [decisions/Frontend-Foundation.md](Frontend-Foundation.md)
- [UI_PAGES.md](../UI_PAGES.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- Milestone API specs 3–7 under `docs/`
