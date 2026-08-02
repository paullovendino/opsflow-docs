# Authentication

## Overview

OpsFlow API authentication is implemented with **Laravel Sanctum SPA cookie authentication**.

The backend authenticates a first-party Vue SPA using session cookies and CSRF protection. Token-based mobile auth is deferred.

Implemented endpoints:

- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/me`

Out of scope for this milestone:

- Registration
- Password reset / forgot password
- Email verification
- Social login
- RBAC enforcement
- User CRUD

---

## Architecture

```text
Client (Vue SPA)
  → GET /sanctum/csrf-cookie
  → POST /api/v1/auth/login
      → guest + throttle:login
      → LoginRequest (validation)
      → AuthController
      → AuthenticationService
      → Auth::attempt (email + password only)
      → session regenerate
      → UserResource in data.user
```

Key classes:

| Class | Responsibility |
|-------|----------------|
| `App\Http\Controllers\Api\V1\AuthController` | Thin HTTP layer |
| `App\Services\Auth\AuthenticationService` | Login, logout, current user |
| `App\Http\Requests\Api\V1\Auth\LoginRequest` | Login validation |
| `App\Http\Resources\Api\V1\UserResource` | User response shape |
| `App\Exceptions\InvalidCredentialsException` | Invalid login → HTTP 401 |
| `App\Exceptions\ApiExceptionRenderer` | Standard API error envelope |

---

## Sanctum SPA Cookie Authentication

- Guard: `web` (session)
- Middleware: `$middleware->statefulApi()`
- Stateful domains: `SANCTUM_STATEFUL_DOMAINS`
- CORS credentials: enabled for configured SPA origins
- Protected routes: `auth:sanctum`

The SPA and API must share a compatible top-level domain strategy in production (same site or approved subdomain cookie setup).

---

## Authentication Flow

1. SPA calls `GET /sanctum/csrf-cookie` (sets `XSRF-TOKEN`).
2. SPA sends `POST /api/v1/auth/login` with credentials, cookies, and `X-XSRF-TOKEN`.
3. Request must include a stateful `Origin` / `Referer` (for example `http://localhost:5173`).
4. On success, Laravel issues an authenticated session cookie.
5. SPA calls protected endpoints (`/auth/me`, later modules) with credentials included.
6. SPA calls `POST /api/v1/auth/logout` to end the session.

Axios requirements:

- `withCredentials: true`
- Send CSRF header derived from the `XSRF-TOKEN` cookie

---

## Session Management

### Login

- `Auth::guard('web')->attempt([...])`
- `$request->session()->regenerate()` (session fixation protection)

### Logout

- `Auth::guard('web')->logout()`
- `$request->session()->invalidate()`
- `$request->session()->regenerateToken()`

### Current User

- Resolved via `$request->user()` under `auth:sanctum`

---

## Login Rate Limiting

Named RateLimiter: `login`

Registered in `AppServiceProvider`:

- Limit: **5 requests per minute**
- Key: `strtolower(email) + '|' + ip`
- Route middleware: `throttle:login`
- Exceeded limit response: HTTP `429`

---

## Guest-only Login

Login is wrapped with Laravel `guest` middleware.

For API/JSON requests, authenticated users receive:

- HTTP `403`
- Message: `Already authenticated.`

Configured via `redirectUsersTo` in `bootstrap/app.php` so API clients get JSON instead of a web redirect.

---

## Standard API Response Format

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

Auth-specific conventions:

| Case | HTTP | Notes |
|------|------|-------|
| Login success | 200 | User under `data.user` |
| `/me` success | 200 | User object in `data` |
| Logout success | 200 | `data` may be null |
| Validation failure | 422 | Field errors in `errors` |
| Invalid credentials | 401 | `Invalid credentials.` |
| Unauthenticated | 401 | `Unauthenticated.` |
| Already authenticated (login) | 403 | `Already authenticated.` |
| Rate limited | 429 | Too many attempts |

---

## Authentication Service Responsibilities

`App\Services\Auth\AuthenticationService`:

- Authenticate credentials (`email`, `password` only)
- Regenerate session after successful login
- Logout and invalidate session
- Return the current authenticated user for `/me`

Controllers remain thin and depend on this service via constructor injection.

---

## Security Decisions

- SPA cookie auth over bearer tokens for the first-party Vue app
- CSRF cookie required before mutating auth requests
- Session regeneration after login
- Session invalidate + CSRF token regenerate on logout
- Credential allowlist into `Auth::attempt()`
- Generic invalid-credentials message (no email enumeration)
- Login rate limiting
- Guest-only login endpoint
- Passwords never returned (`User` hidden attributes + `UserResource`)

See also: [decisions/Authentication.md](decisions/Authentication.md)

---

## Future Authentication Improvements

Deferred to later milestones:

- Pinia auth store and Vue route guards
- Users ERD alignment (`role_id`, profile fields, status)
- RBAC policies / gates using seeded roles
- Failed-login activity logging
- Optional remember-me
- Token auth for mobile clients
- Password reset and email verification (if product requires them)
