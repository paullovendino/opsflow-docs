# Cursor Development Rules

## General

- Follow Laravel 13 best practices.
- Follow Vue 3 Composition API.
- Use TypeScript.
- Write clean, readable code.
- Avoid duplicated logic.
- Prefer dependency injection.
- Do not generate unnecessary comments.
- Keep controllers thin.
- Put business logic in services when appropriate.
- Use Form Requests for validation.
- Use API Resources for responses.
- Use PHP Enums instead of magic strings.
- Use Eloquent relationships instead of manual joins when appropriate.
- Follow PSR-12 coding standards.
- Do not introduce packages unless requested.
- If a requirement is unclear, ask instead of assuming.
- Follow approved decisions in `opsflow-docs/decisions/`.
- Prefer `opsflow-docs/docs/DOMAIN_MODEL.md` for business concepts.
- Prefer milestone specs under `opsflow-docs/docs/` before implementing a phase.
- Do not implement future modules early (Remarks/Activity Logs/Reports/advanced RBAC) unless the milestone says so.
- Milestones **3–6** backend are **complete**. Do **not** implement Phase 7 (Reports) without explicit implementation approval.
- Match existing patterns: `UserController` → Form Request → `UserService` / `UserQuery` → Resources; authorize via `UserPolicy`; lookups use `LookupController` → `LookupService` → Resources. For Projects/Tasks, mirror with Controller / Service / Query / Policy per milestone specs. For Dashboard: `DashboardController` → `ShowDashboardRequest` → `DashboardService` → `DashboardResource` + `viewDashboard` Gate per `docs/MILESTONE_6_DASHBOARD.md`.

## API

- All endpoints must be under `/api/v1`.
- Return the approved JSON response envelope from `CODING_STANDARDS.md`.
- Use proper HTTP status codes.
- Never return raw Eloquent models.
- Follow `AUTHENTICATION.md` and `decisions/Authentication.md` for auth endpoints.

## Database

- PostgreSQL only.
- Use `Relation::enforceMorphMap` for polymorphic aliases.
- Register morph aliases only for models that already exist.
- Milestone 6 adds **no** new tables — do not invent `activity_logs` for the dashboard.

## Frontend

- Use Pinia for global state.
- Use Axios for API requests.
- Use Vue Router navigation guards.
- Use Composition API only.
- Vue Dashboard UI is **out of scope** for Milestone 6 API work.
