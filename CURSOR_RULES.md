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
- Do not implement future modules early (Projects/Tasks/Remarks/advanced RBAC) unless the milestone says so.
- Current Milestone 3 backend is **complete**. Phase **4.1** is complete — do **not** start Phase 4.2 without explicit implementation approval.
- Match existing patterns: `UserController` → Form Request → `UserService` / `UserQuery` → Resources; authorize via `UserPolicy`; lookups use `LookupController` → `LookupService` → Resources (no Form Requests for collection GETs). For Projects, mirror with `ProjectController` / `ProjectService` / `ProjectQuery` / `ProjectPolicy` per `docs/MILESTONE_4_PROJECT_MANAGEMENT.md`.

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

## Frontend

- Use Pinia for global state.
- Use Axios for API requests.
- Use Vue Router navigation guards.
- Use Composition API only.
