# UI Pages

## Public

> Milestone 8 — Login in scope (design package)

- Login (`/login` — Milestone 8.2)

---

## App shell (Milestone 8)

> Spec: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) · ADR: [decisions/Frontend-Foundation.md](decisions/Frontend-Foundation.md)

- App Home (`/` — authenticated shell placeholder; **not** Dashboard feature UI)
- Forbidden (`/403`)
- Not Found (catch-all)
- Auth layout: sidebar + topbar (Milestone 8.3); **Logout in topbar**
- Guest layout: centered login

---

## Dashboard

> Milestone 6 API complete; Vue UI **after** Milestone 8  
> Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md)

- Dashboard Overview (API: `GET /api/v1/dashboard` — statistics + recent work items; Vue pending — **out of Milestone 8**)

---

## Organization & User Management

> Frontend pages planned **after** Milestone 8. Backend User CRUD APIs implemented (Phase 3.3).

- User List (API: search/filters/sorting/pagination + authz — Milestone 3 complete; UI pending)
- Create User
- Edit User
- User Details
- My Profile

### Reference data (read-only UI)

> Lookup APIs implemented in Phase 3.4 (`/api/v1/lookups/*`); Vue screens later

- Roles lookup (no CRUD screens in Milestone 3)
- Departments lookup (no CRUD screens in Milestone 3)
- Job Titles lookup (no CRUD screens in Milestone 3)

---

## Project Management

> Vue UI after Milestone 8

- Project List
- Create Project
- Edit Project
- Project Details
- Project Members (Milestone 4 API; Vue UI later)

---

## Task Management

> Milestone 5 API complete; Vue UI after Milestone 8

- Task List
- Create Task
- Edit Task
- Task Details
- Task Assignment / Status (API: Milestone 5)

---

## Reports

> Milestone 7 API complete; Vue UI after Milestone 8  
> Spec: [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md)

- Project Reports (API: `/api/v1/reports/projects`)
- Employee Reports (API: `/api/v1/reports/employees`)

---

## Activity Logs

> Planned — future (not Milestone 6, 7, or 8; dashboard recent ≠ activity logs; reports ≠ activity logs)

- Activity List

---

## Profile

- My Profile (after Milestone 8)

---

## Settings

- Account Settings
- Organization Settings (future — out of scope for Milestone 3)
