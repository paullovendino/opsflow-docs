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

# Milestone suites (Phases 2–4.3)
php artisan test --filter="AuthenticationTest|OrganizationFoundationTest|UserDomainFoundationTest|UserManagementApiTest|UserListQueryTest|LookupApiTest|UserAuthorizationTest|ProjectDomainFoundationTest|ProjectManagementApiTest|ProjectMembersApiTest"
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

## Lookups (Phase 3.4)

Path: `tests/Feature/Lookup/LookupApiTest.php`

| Area | Expectations |
|------|----------------|
| Auth | Authenticated `200`; guest `401` |
| Roles / Departments / Job Titles | Collections via `/api/v1/lookups/*` |
| Soft deletes | Soft-deleted departments/job titles excluded |
| Ordering | Alphabetical by `name` |

---

## User List Query (Phase 3.5)

Path: `tests/Feature/User/UserListQueryTest.php`

| Area | Expectations |
|------|----------------|
| Defaults | `per_page` 15; sort `created_at` desc; pagination `meta` present |
| Search | Matches `first_name`, `middle_name`, `last_name`, `email` |
| Filters | `role_id`, `department_id`, `job_title_id`, `status` (composable with search) |
| Sorting | Allowed columns + `asc`/`desc` |
| Pagination | `page` / `per_page`; `per_page` > 100 clamped to 100 |
| Validation | Invalid `sort` / `status` / `direction` → `422` |
| Guest | `401` |

## User Authorization (Phase 3.6)

Path: `tests/Feature/User/UserAuthorizationTest.php`

| Area | Expectations |
|------|----------------|
| Administrator | Full list/view/create/update/delete/status |
| Project Manager | List + view only; mutations `403` |
| Employee | View own profile `200`; list/other/mutate `403` |
| Envelope | Unauthorized uses `success: false`, HTTP `403` |

---

## Project Domain Foundation (Phase 4.1)

Path: `tests/Feature/Project/ProjectDomainFoundationTest.php`

| Area | Expectations |
|------|----------------|
| Schema | `projects` / `project_members` columns present; pivot has no `deleted_at` |
| Owner | `owner` / `createdBy` / `ownedProjects` |
| Members | `members` / `projects` pivot with `joined_at`; owner not auto-added |
| Unique | Duplicate (`project_id`, `user_id`) → `QueryException` |
| Enum | `ProjectStatus` cast + default `planning` |
| Soft deletes | Soft-deleted projects excluded; member rows retained |
| FK RESTRICT | forceDelete owner/member/project-with-members → `QueryException` |
| Factory | Generates owner + default Planning status |
| Morph map | `project` alias registered |

## Project Management APIs (Phase 4.2)

Path: `tests/Feature/Project/ProjectManagementApiTest.php`

| Area | Expectations |
|------|----------------|
| Create | `201`, owner = auth user, status `planning`, envelope + owner nested |
| Owner assignment | Client `created_by` ignored |
| Validation | Missing `name` → `422` |
| List / show | `200`, owner nested, no members |
| Update | Mutable fields only; status/owner unchanged |
| Soft delete | `200`, `assertSoftDeleted` |
| Status patch | Only `status` changes; invalid enum → `422` |
| Guest | All project endpoints `401` |
| Resource shape | id/name/description/status/dates/owner/timestamps |

## Project Members APIs (Phase 4.3)

Path: `tests/Feature/Project/ProjectMembersApiTest.php`

| Area | Expectations |
|------|----------------|
| List | `200`, user summary + `joined_at`; owner not auto-listed |
| Add | `201`, server `joined_at`; client `joined_at` ignored |
| Duplicate | HTTP `409` envelope |
| Inactive / soft-deleted user | `422` on `user_id` |
| Remove | `200`, pivot gone; owner unchanged |
| Unknown member | `404` |
| Guest | Member endpoints `401` |

## Project Management (Milestone 4 — complete)

Spec: [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md)

| Phase | Path | Coverage |
|-------|------|----------|
| 4.4 | `tests/Feature/Project/ProjectListQueryTest.php` | ✅ search, filters, sort, pagination, clamp, validation |
| 4.5 | `tests/Feature/Project/ProjectAuthorizationTest.php` | ✅ Admin / PM / Employee matrix (owned-or-member for Employee) |

## Task Management (Milestone 5 — ✅ Complete)

Spec: [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md)

| Phase | Path | Coverage |
|-------|------|----------|
| 5.1 | `tests/Feature/Task/TaskDomainFoundationTest.php` | ✅ schema, relations, enums, soft delete, FK RESTRICT, factory |
| 5.2 | `tests/Feature/Task/TaskManagementApiTest.php` | ✅ CRUD, validation, defaults, `created_by`, assignment on create, guest `401`, resource shape |
| 5.3 | `tests/Feature/Task/TaskAssignmentApiTest.php` | ✅ assign/clear; active+owner/member only; `422` cases |
| 5.4 | `tests/Feature/Task/TaskListQueryTest.php` | ✅ search, filters, sort, pagination, clamp, validation |
| 5.5 | `tests/Feature/Task/TaskAuthorizationTest.php` | ✅ Admin / PM / Employee matrix (status ability when assigned) |
| 5.6 | `tests/Feature/Task/TaskStatusApiTest.php` | ✅ status patch; create/update status-free; all enum values; Employee assigned-to-self authz |

---

## Dashboard (Milestone 6 — ✅ Complete)

Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · ADR: [decisions/Dashboard.md](decisions/Dashboard.md)

| Phase | Path | Coverage |
|-------|------|----------|
| 6.1–6.3 | `tests/Feature/Dashboard/DashboardApiTest.php` | ✅ envelope; statistics; overdue; assigned_to_me; recent merge/limit/clamp; guest `401`; validation |
| 6.4 | `tests/Feature/Dashboard/DashboardAuthorizationTest.php` | ✅ Admin / PM / Employee visibility matrix; policy view |

---

## Reports (Milestone 7 — ✅ Complete)

Spec: [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) · ADR: [decisions/Reports.md](decisions/Reports.md)

| Phase | Path | Coverage |
|-------|------|----------|
| 7.1–7.2 | `tests/Feature/Report/ProjectReportApiTest.php` | ✅ list/detail; filters; date range; overdue/unassigned; pagination; guest `401` |
| 7.3 | `tests/Feature/Report/EmployeeReportApiTest.php` | ✅ list/detail; filters; `by_project`; pagination; validation |
| 7.4 | `tests/Feature/Report/ReportAuthorizationTest.php` | ✅ Admin / PM / Employee matrix |

---

## Remaining coverage (not yet)

- Explicit `429` rate-limit assertion
- CSRF rejection cases
- Frontend Pinia auth tests
- Milestone 8+ broader testing suites
- Activity Logs suites
