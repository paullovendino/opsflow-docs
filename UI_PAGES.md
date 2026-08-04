# UI Pages

## Public

> Milestone 8 — ✅ Login implemented

- Login (`/login` — Milestone 8.2)

---

## App shell (Milestone 8)

> ✅ Complete · Spec: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md)

- Auth layout: sidebar + topbar; **Logout in topbar**
- Guest layout: centered login
- Forbidden (`/403`) · Not Found (catch-all)
- Landing: authenticated `/` → `/dashboard` (Phase 9.1)

---

## Dashboard

> API: Milestone 6 ✅ · Vue: Phase 9.1 ✅  
> Spec: [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) · [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md)

- Dashboard Overview (`/dashboard` — `GET /api/v1/dashboard`)
- Stat cards · Tailwind status bars · recent work
- Skeleton / empty recent / error+retry
- Landing: `/` → `/dashboard`

---

## Organization & User Management

> API: Milestone 3 ✅ · Vue: Milestone 9.2 (designed)

- User List (`/users`)
- Create User (`/users/create`) — **page**
- Edit User (`/users/:id/edit`) — **page**
- User Details (`/users/:id`)
- My Profile (`/profile`)
- Lookups via selects (no CRUD screens)

---

## Project Management

> API: Milestone 4 ✅ · Vue: Milestone 9.3 (designed)

- Project List / Create / Edit / Details
- Project Members (on Show)

---

## Task Management

> API: Milestone 5 ✅ · Vue: Milestone 9.4 (designed) — **table only** (Kanban later)

- Task List / Create / Edit / Details
- Assignment / Status controls

---

## Reports

> API: Milestone 7 ✅ · Vue: Milestone 9.5 (designed)

- Project Reports list + detail
- Employee Reports list + detail
- Date filters (`from_date` / `to_date`)

---

## Activity Logs

> Planned — future

- Activity List

---

## Settings

- Account Settings
- Organization Settings (future)
