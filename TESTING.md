# Testing

## Overview

OpsFlow API tests use PHPUnit via Laravel's testing harness.

Authentication coverage lives in:

`opsflow-api/tests/Feature/Auth/AuthenticationTest.php`

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

# Authentication suite only
php artisan test --filter=AuthenticationTest
```

---

## Authentication Feature Tests

Current cases:

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

## Expected Coverage (Authentication Milestone)

Must remain green:

- Login success / failure / validation
- Guest rejection on login
- `/me` authenticated and unauthenticated
- Logout ends session for subsequent `/me`

Recommended later (not required to close this milestone):

- Explicit `429` rate-limit assertion
- CSRF rejection cases
- Frontend Pinia auth tests

---

## Organization & User Management (Milestone 3 — Planned)

When implementation begins, add feature coverage for:

| Area | Expectations |
|------|----------------|
| Users CRUD | Create/update/show/list/soft-delete with envelope |
| Status patch | Activate / deactivate |
| Filters | `search`, `role_id`, `department_id`, `job_title_id`, `status` + pagination `meta` |
| Lookups | Roles / Departments / Job Titles list + show for any authenticated user |
| Authorization | Administrator write; Project Manager read directory; Employee own profile |
| Inactive login | Credentials valid but `status = inactive` → HTTP `403` (`Account is inactive.`) |
| Auth regression | Login/`/me` still green after `UserResource` expansion |
| `last_login_at` | Updated on successful login |

Seeders for roles, departments, and job titles must be available in the test database (`RefreshDatabase` + seed as needed).

Spec: [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md)
