# Decision: Authentication

## Status

Accepted

## Context

OpsFlow has a separate Vue 3 SPA (`opsflow-web`) and a Laravel API (`opsflow-api`). Authentication must be secure for a first-party SPA without implementing token-based mobile auth yet.

## Decision

Use **Laravel Sanctum SPA cookie authentication**.

Details:

- Session-based auth via the `web` guard
- Stateful domains configured through `SANCTUM_STATEFUL_DOMAINS`
- CSRF cookie endpoint: `GET /sanctum/csrf-cookie`
- CORS allows credentialed requests from the Vue app
- Auth API endpoints will live under `/api/v1` (login, logout, user)
- Authentication endpoints are implemented in Phase 2

## Local CORS / Stateful Defaults

- Origins: `http://localhost:5173`, `http://127.0.0.1:5173`
- Env-driven for production via `CORS_ALLOWED_ORIGINS`
- Stateful domains include `localhost:5173` and `127.0.0.1:5173`

## Consequences

- Frontend must call `/sanctum/csrf-cookie` before login
- Axios must use `withCredentials: true`
- Production requires matching SPA/API top-level domain (or subdomain) strategy
- Token auth for mobile can be added later without replacing SPA cookie auth
