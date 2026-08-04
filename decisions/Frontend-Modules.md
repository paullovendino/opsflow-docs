# Decision: Frontend Modules

## Status

Accepted — Milestone 9 in progress (✅ Phase 9.1 · remaining phases await approval)

## Context

Milestone 8 delivered the OpsFlow SPA foundation (auth, shell, Axios/Sanctum, shared primitives). Backend Milestones 3–7 already expose Users, Projects, Tasks, Dashboard, and Reports APIs. The product still lacks remaining feature UIs after Dashboard.

Roadmap Phase 9 was previously “Testing.” Feature UIs must land before a meaningful frontend test pass. Milestone 9 is therefore **Frontend Modules**; Testing and Deployment shift to Phases 10–11.

Companion specification: [docs/MILESTONE_9_FRONTEND_MODULES.md](../docs/MILESTONE_9_FRONTEND_MODULES.md)

## Decision

### Milestone 9 is frontend-only

- Work in `opsflow-web`
- **No** new API endpoints, contract changes, migrations, or morph aliases
- Consume existing `/api/v1` resources under Sanctum session auth

### Phasing (mandatory order)

| Phase | Module | Status |
|-------|--------|--------|
| 9.1 | Dashboard UI | ✅ Implemented |
| 9.2 | User Management | 📋 Designed — next |
| 9.3 | Project Management | 📋 Designed |
| 9.4 | Task Management | 📋 Designed |
| 9.5 | Reports | 📋 Designed |

### Architecture

- Extend Milestone 8 layouts/router/http/auth/ui toast patterns
- Feature code under `src/modules/{dashboard,users,projects,tasks,reports}/`
- Domain HTTP in `services/*Service.ts`; types in `types/`
- Prefer **composables** for list/query state; keep Pinia limited to `auth` + `ui`
- Shared table/badge/pagination/confirm/stat/bar-chart components in `components/ui`
- **No** UI framework; **no** chart library (Tailwind bars only)

### Locked product UX

| Area | Choice |
|------|--------|
| Landing | `/` → `/dashboard`; App Home placeholder removed in 9.1 |
| Dashboard | Stat cards + Tailwind status bars + recent work (✅) |
| Users | Table + filters + pagination; Create/Edit **pages**; status badges; Profile route |
| Projects | Table + CRUD pages; members on Show; status badges |
| Tasks | **Table only** (no Kanban); assignment + status controls; priority colors |
| Reports | List/detail; date filters; cards + Tailwind charts + tables |
| Confirms | `AppConfirmDialog` for delete/status/member remove |
| Nav | Enable sidebar links per phase; role-aware visibility |

### Roadmap renumbering

| Phase | Milestone |
|-------|-----------|
| **9** | **Frontend Modules** (this ADR) |
| **10** | Testing (formerly Phase 9) |
| **11** | Deployment (formerly Phase 10) |

### Explicitly out of scope

Kanban, chart libraries, exports, Activity Logs/Remarks UI, automated test suite as M9 deliverable, deployment, backend changes.

## Consequences

- Companion docs point next work at **Phase 9.2** (after approval)
- Phase 9.1 delivered Dashboard landing; remaining modules stay designed until approved
- Do **not** implement later phases until user explicitly approves
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
