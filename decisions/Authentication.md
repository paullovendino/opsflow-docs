# Decision: Authentication

## Status

Accepted (Implemented — Phase 2 API)

## Context

OpsFlow has a separate Vue 3 SPA (`opsflow-web`) and a Laravel API (`opsflow-api`). Authentication must be secure for a first-party SPA without implementing token-based mobile auth yet.

## Decision

Use **Laravel Sanctum SPA cookie authentication**.

### Core choices

- Session-based auth via the `web` guard
- Stateful domains configured through `SANCTUM_STATEFUL_DOMAINS`
- CSRF cookie endpoint: `GET /sanctum/csrf-cookie`
- CORS allows credentialed requests from the Vue app
- Service-layer auth logic in `App\Services\Auth\AuthenticationService`
- Form Request validation via `LoginRequest`
- API Resources via `UserResource`
- Standard API response envelope (`success`, `message`, `data`, `errors`, `meta`)

### Endpoints

- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/me`

### Security controls

- Logout and me are protected with `auth:sanctum`
- Successful login returns the authenticated user under `data.user`
- Invalid credentials return HTTP `401`
- Validation failures return HTTP `422`
- Login is rate limited via the named `login` RateLimiter (`throttle:login`, 5 attempts/minute per email + IP)
- Authenticated users cannot access login (`guest` middleware → HTTP `403` for API/JSON)
- `Auth::attempt()` receives only `email` and `password` (credential allowlist)
- Session regenerated after login; invalidated on logout

### RBAC (Phase 3.6 — implemented)

- Roles table + `RoleName` enum + seeder; `User::role()` relation
- Coarse User Management authorization via `UserPolicy` (not advanced RBAC)
- Unauthorized User Management actions return HTTP `403` with the standard API envelope
- See [decisions/Organization-User-Management.md](Organization-User-Management.md)

### Inactive accounts (Phase 3.2 — implemented)

- Users with `status = inactive` must not be able to log in
- Login rejection uses HTTP `403` with a dedicated message (`Account is inactive.`)
- Do not collapse inactive accounts into invalid-credentials `401`

## Local CORS / Stateful Defaults

- Origins: `http://localhost:5173`, `http://127.0.0.1:5173`
- Env-driven for production via `CORS_ALLOWED_ORIGINS`
- Stateful domains include `localhost:5173` and `127.0.0.1:5173`

## Consequences

- Frontend must call `/sanctum/csrf-cookie` before login
- Axios must use `withCredentials: true`
- Production requires matching SPA/API top-level domain (or subdomain) strategy
- Token auth for mobile can be added later without replacing SPA cookie auth
- References: [AUTHENTICATION.md](../AUTHENTICATION.md), [API_SPECIFICATION.md](../API_SPECIFICATION.md), [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md)
- Pinia auth store delivered in Milestone 8.2 ([docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md))
- Login inactive (`403`) and invalid credentials (`401`) are form concerns on the SPA; do not conflate with a global forbidden page

## References

- [AUTHENTICATION.md](../AUTHENTICATION.md)
- [API_SPECIFICATION.md](../API_SPECIFICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
