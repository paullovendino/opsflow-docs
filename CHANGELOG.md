# Changelog

All notable OpsFlow milestones are recorded here.

Format follows a lightweight Keep a Changelog style.

---

## [Unreleased] — v1.0.0 (Development)

### Backend Foundation (Phase 1) — Completed

- Laravel 13 API application (`opsflow-api`)
- PostgreSQL as the application database
- Laravel Sanctum installed and configured for SPA cookie mode
- API versioning under `/api/v1`
- Environment-driven CORS for the Vue SPA origins
- Agreed backend folder structure (Services, Actions, Enums, Requests, Resources, etc.)
- `BaseApiController` and standard JSON response envelope
- Global API exception rendering (`ApiExceptionRenderer`)
- Morph map foundation (`user`, `role`)
- Roles migration, `Role` model, `RoleName` enum, and `RolesSeeder`
- Health check endpoint: `GET /api/v1/health`
- Foundation documentation and ADRs

### Authentication (Phase 2, API) — Completed

- Sanctum SPA cookie authentication for the first-party Vue SPA
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/me`
- `AuthenticationService` (service-layer auth logic)
- `LoginRequest` Form Request validation
- `UserResource` API Resource responses
- Named `login` RateLimiter (`throttle:login`, 5/min per email + IP)
- Guest-only login (`guest` middleware; API `403` when already authenticated)
- Credential allowlist for `Auth::attempt()` (`email`, `password` only)
- Authentication feature test suite
- Authentication documentation (`AUTHENTICATION.md`, setup/testing/API/ADR updates)

### Changed

- Auth routes standardized under `/api/v1/auth/*`
- Documentation aligned to Laravel 13 / PHP 8.3+
- Tech Stack ADR completed (`decisions/Tech-Stack.md`)

### Deferred

- Vue Pinia authentication (`opsflow-web`)
- User Management / RBAC enforcement
- Registration, password reset, email verification, social login
- Draw.io diagrams, API collections, environment-config hardening
