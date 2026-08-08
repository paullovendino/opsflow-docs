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
- Do not implement future modules early (Remarks/Activity Logs/advanced RBAC) unless the milestone says so.
- Milestones **3–9** and **Phase 10 Testing & QA** are **complete**. **Next:** Milestone 10 Product Enhancements (planned — do **not** implement until a design package is approved). Do **not** implement Milestone 11 Deployment early. Follow `docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md` / `docs/MILESTONE_10_TESTING_QA.md` / `decisions/Testing-QA.md`.
- Match existing patterns. Frontend modules: follow `docs/MILESTONE_9_FRONTEND_MODULES.md` / `decisions/Frontend-Modules.md`. Foundation: `docs/MILESTONE_8_FRONTEND_FOUNDATION.md`.

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
- Milestones 6–7 add **no** new tables — do not invent `activity_logs` or report tables for these milestones.

## Frontend

- Use Pinia for global state.
- Use Axios for API requests (`withCredentials: true`, `withXSRFToken: true`).
- Use Vue Router navigation guards.
- Use Composition API only.
- Use Tailwind CSS; **no UI framework** unless an ADR approves one.
- Follow [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) for the auth shell (✅ Milestone 8).
- Follow [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md) for feature modules (✅ Milestone 9 complete — Dashboard, Users, Projects, Tasks, Reports).
- Users, Projects, and Tasks Create/Edit/View use `AppModal` on the list; do not reintroduce dedicated Create/Edit pages for those modules unless an ADR revises that. Show pages (`/users/:id`, `/projects/:id`, `/tasks/:id`) remain. Reports are dedicated pages.
- Task assignees must be the **project owner or a project member** (existing rule — do not reinterpret).
- `useLookups` cache is composable module-level **in-memory SPA-session** + in-flight dedupe — do not invent Pinia / localStorage / Redis lookup caching. Do not introduce Vuex or another state-management library.
- Stable family `viewKey`: Create/Edit modal aliases share the list view key; do not remount/refetch the list on modal navigation; do not start route progress on modal aliases.
- No chart library / UI kit / Kanban unless an ADR changes that.
