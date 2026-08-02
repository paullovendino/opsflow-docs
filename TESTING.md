# Testing

## Overview

OpsFlow API tests use PHPUnit via Laravel's testing harness.

---

## Test Database Requirements

This environment does not use SQLite (`pdo_sqlite` may be unavailable).

PHPUnit is configured for PostgreSQL:

| Setting | Value |
|---------|-------|
| Connection | `pgsql` |
| Database | `opsflow_testing` |
| Host | `127.0.0.1` |
| Port | `5432` |
| Session driver | `cookie` |

Create the database once:

```sql
CREATE DATABASE opsflow_testing;
```

Ensure the DB user in `phpunit.xml` can migrate/drop tables. Prefer moving secrets to a local override in a future hardening pass.

Tests use `RefreshDatabase`.

---

## How to Run Tests

From `opsflow-api`:

```bash
# All tests
php artisan test

# Milestone suites (Phases 2–3.3)
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest"
```

---

## Authentication Feature Tests

Path: `tests/Feature/Auth/AuthenticationTest.php`

| Test | Expectation |
|------|-------------|
| Successful login | `200`, `data.user`, authenticated session |
| Invalid credentials | `401`, `Invalid credentials.` |
| Validation failure | `422`, email/password errors |
| Authenticated `/me` | `200`, user in `data` |
| Unauthenticated `/me` | `401`, `Unauthenticated.` |
| Authenticated user cannot login again | `403`, `Already authenticated.` |
| Successful logout | `200`, then `/me` is `401` |

Notes:

- Stateful SPA behavior is simulated with `Origin: http://localhost:5173`
- After logout (and guest checks), tests call `$this->app['auth']->forgetGuards()` so the next request reloads auth from the session (Laravel 13 HTTP test client quirk)

---

## Organization Foundation (Phase 3.1)

Path: `tests/Feature/Organization/OrganizationFoundationTest.php`

Coverage: departments/job_titles migrations, seeders, unique constraints, soft deletes.

---

## User Domain Foundation (Phase 3.2)

Path: `tests/Feature/User/UserDomainFoundationTest.php`

Coverage: users ERD migration / legacy name split, relationships, `UserStatus`, `full_name`, soft deletes, inactive login `403`, expanded auth `UserResource`.

---

## User Management APIs (Phase 3.3)

Path: `tests/Feature/User/UserManagementApiTest.php`

| Area | Expectations |
|------|----------------|
| Create user | `201`, envelope, nested resources |
| Update user | `200`, fields updated |
| Show / list | `200`, relations loaded |
| Soft delete | `200`, `assertSoftDeleted` |
| Status patch | Only `status` changes (`active` / `inactive`) |
| Validation | `422` for missing fields |
| Unique email | `422` on duplicate |
| Password hashing | `Hash::check` succeeds; plain text not stored |
| Resource shape | `UserResource` + Role/Department/JobTitle resources |
| Guest access | `401` |

---

## Remaining Milestone 3 coverage (not yet)

- Lookup API tests (Phase 3.4)
- Filters / search / pagination (Phase 3.5)
- Coarse authorization matrix (Phase 3.6)
- Explicit `429` rate-limit assertion
- CSRF rejection cases
- Frontend Pinia auth tests
