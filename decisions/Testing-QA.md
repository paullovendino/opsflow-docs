# Decision: Testing & QA

## Status

Accepted — ✅ **Complete** (Phase 10.1–10.3 + in-scope QA fix). Playwright/Cypress and GitHub Actions remain **deferred**.

## Context

Milestones 8–9 delivered the Vue SPA (auth shell + Dashboard, Users, Projects, Tasks, Reports, including post-ship modal/loading/lookup UX). Backend Milestones 2–7 already ship substantial PHPUnit Feature coverage.

Roadmap **Phase 10** was **Testing & QA** (quality-only). **Milestone 10** is now **Product Enhancements** (next, not implemented). **Milestone 11** is **Deployment** (future).

Companion specification: [docs/MILESTONE_10_TESTING_QA.md](../docs/MILESTONE_10_TESTING_QA.md)

User approved implementation 2026-08-08:

| Decision | Outcome |
|----------|---------|
| Vitest + Vue Test Utils + happy-dom | ✅ Approved and installed |
| Backend gap-fill tests | ✅ Approved |
| Frontend automated tests | ✅ Approved |
| Manual browser QA | ✅ Executed and passed |
| CSRF `419` where practical | ✅ `ApiExceptionRenderer` unit test |
| Remove scaffold `ExampleTest` | ✅ Removed |
| Playwright/Cypress | ❌ Deferred |
| GitHub Actions | ❌ Deferred |
| Shared PHPUnit actor trait | ❌ Deferred |

## Decision

### Phase 10 was quality-only

- Validate the system that exists today (`opsflow-api` + `opsflow-web`)
- **No** new business features as Phase 10 deliverables
- **No** deployment / production infrastructure (Milestone 11)
- In-scope defects: authz/security, envelope regressions, broken happy paths, loading/empty bugs, lookup/remount regressions

### Backend: PHPUnit gap-fill, not Pest

- Keep **PHPUnit** Feature tests (`php artisan test`)
- Database: PostgreSQL `opsflow_testing`, `RefreshDatabase`, cookie session, `Origin: http://localhost:5173`
- Gap-fill: login **429**; CSRF **419** via `ApiExceptionRenderer` unit test (Laravel skips CSRF in `runningUnitTests()`); empty-report list cases; scaffold `ExampleTest` removed
- Full suite: **215** passed
- No shared Admin/PM/Employee actor trait

### Frontend: Vitest + Vue Test Utils + happy-dom

- Installed: `vitest`, `@vue/test-utils`, `happy-dom`
- Scripts: `npm run test` / `npm run test:watch`
- Last run: **69** tests / 26 files; `npm run type-check` + `npm run build` green
- Mock domain `*Service.ts`; no live API for unit/component tests
- Helpers: `modalRoutes.ts`, `httpProgress.ts`, `listLoading.ts`; `resetLookupsCache()`
- **E2E:** manual browser checklist (Sanctum cookies). Playwright/Cypress deferred. **No CI.**

### Manual QA

Cross-module browser workflow (Admin creates user → project → member → task → assign → Employee status → dashboard/reports) executed successfully. Authentication, Users, Projects, Tasks, and Reports verified.

### Phase 10.3 QA fix — global modal navigation

Root cause: `AuthLayout` `RouterView` keying used **path** for list index (`/users`) and **name** for Create/Edit (`users.index`), so opening a modal remounted the list, refetched, flashed loading/progress, and dropped query state.

Centralized fix (Users / Projects / Tasks):

- Stable family `viewKey` (`authLayoutViewKey`): index + create + edit share `users.index` / `projects.index` / `tasks.index`
- `shouldTrackRouteProgress`: no route progress for modal-alias navigation; real page navigation still tracked
- `openModalAlias` / `listIndexLocation`: preserve search / filters / pagination query
- Modal-scoped detail GETs may use `quietProgress`; lookups remain quiet + SPA-session cache / in-flight dedupe
- List stays mounted; modal-local loading remains

### Quality gates (local; no CI)

| Gate | Status |
|------|--------|
| `php artisan test` | ✅ **215** passed |
| `npm run type-check` | ✅ |
| `npm run build` | ✅ |
| `npm run test` | ✅ **69** / 26 files |
| Manual QA checklist | ✅ Passed |

## Consequences

- Phase 10 Testing & QA is **complete**
- Next: **Milestone 10 — Product Enhancements** (design then implementation; not started)
- Then: **Milestone 11 — Deployment**
- Do **not** invent GitHub Actions or Playwright unless separately approved

## References

- [docs/MILESTONE_10_TESTING_QA.md](../docs/MILESTONE_10_TESTING_QA.md)
- [docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md](../docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md)
- [TESTING.md](../TESTING.md)
- [HANDOFF.md](../HANDOFF.md)
- [ROADMAP.md](../ROADMAP.md)
