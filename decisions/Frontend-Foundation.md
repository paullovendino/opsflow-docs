# Decision: Frontend Foundation

## Status

Accepted — Milestone 8 complete (Phases 8.1–8.3)

## Context

OpsFlow completed API Milestones 1–7. `opsflow-web` existed only as a create-vue scaffold (Vue + Vite + TypeScript) with **no** Pinia, Vue Router, Axios, Tailwind, auth, or app shell.

The roadmap previously listed Phase 8 as a vague “Testing” bucket while Vue/Pinia auth remained an unresolved Phase 2 follow-up. Frontend testing cannot precede a real SPA foundation. Milestone 8 therefore establishes the **Frontend Foundation** before feature UIs and before a dedicated Testing phase.

Companion specification: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md)

## Decision

### Milestone 8 is frontend-only

- Work occurs in `opsflow-web`
- **No** new API endpoints, contract changes, migrations, or morph aliases
- Consume existing Sanctum SPA auth: CSRF cookie, `POST /api/v1/auth/login`, `POST /api/v1/auth/logout`, `GET /api/v1/auth/me`

### Approved stack (unchanged)

Vue 3 (Composition API) · TypeScript · Pinia · Vue Router · Axios · Tailwind CSS

**No UI framework** unless a future ADR approves one. No extra packages without approval (Tailwind Vite/PostCSS tooling excepted).

### Phasing

| Phase | Scope | Status |
|-------|--------|--------|
| 8.1 | Application foundation — folders, tooling, env, Axios client, types, router skeleton, remove create-vue welcome | ✅ Implemented |
| 8.2 | Authentication foundation — CSRF, login/logout, `/me` bootstrap, auth store, guards, Guest vs Auth layouts | ✅ Implemented |
| 8.3 | UI foundation — sidebar, topbar, shell, responsive nav, loading/empty/error, toast, shared UI components | ✅ Implemented |

### Auth model

- Session cookies + CSRF (not bearer tokens in the SPA)
- Pinia holds **user state only**; persistence is the HTTP session cookie
- App bootstrap calls `/me`; `401` ⇒ guest
- Route meta: `requiresAuth` / `guest`
- Handle API `401` / `403` (incl. inactive) / `422` / `429` per existing envelopes

### Routing & pages in scope

| In scope | Out of scope |
|----------|--------------|
| `/login`, `/` (App Home shell), `/403`, 404 catch-all | Dashboard, Users, Projects, Tasks, Reports **pages** |
| GuestLayout, AuthLayout via **nested routes** | Feature module CRUD UIs |
| Disabled nav placeholders optional | Calling dashboard/users/projects/tasks/reports APIs for product UI |

`AppHomeView` proves the authenticated shell. It is **not** the Dashboard feature page.

### Locked implementation choices

| Topic | Choice |
|-------|--------|
| Layout wiring | Nested routes (not `meta.layout` switcher) |
| Tailwind | `@tailwindcss/vite` |
| Env | `.env.example` + local `.env` |
| Axios timeout | 15000 ms |
| Axios CSRF (cross-origin) | `withXSRFToken: true` + Laravel XSRF cookie/header names |
| Logout UI | Topbar (sidebar has no logout) |
| Global `403` interceptor | No auto-navigate to `/403`; LoginView owns login `403`s |
| AuthUser | Normalize login `data.user` and `/me` `data` to `UserResource`-aligned shape |

### Local environment

- SPA: `http://localhost:5173`
- API: `VITE_API_BASE_URL` (e.g. `http://localhost:8000`)
- Rely on existing CORS + `SANCTUM_STATEFUL_DOMAINS`
- Vite proxy **not** required for Milestone 8

### Toast / UI primitives

- First-party toast via Pinia `ui` store + host component
- Small shared Tailwind components (`AppButton`, `AppInput`, `AppSpinner`, `AppEmptyState`, shell parts)
- No toast/UI kit dependency
- Login credential/inactive/validation errors prefer **inline** form messages

### Roadmap renumbering

| Phase | Milestone |
|-------|-----------|
| **8** | **Frontend Foundation** (this ADR) — ✅ complete |
| **9** | **Frontend Modules** — see [Frontend-Modules.md](Frontend-Modules.md) |
| **10** | Testing |
| **11** | Deployment |

### Explicitly out of scope

Feature module pages, frontend test frameworks as a deliverable, deployment, registration/password reset, Activity Logs/Remarks UI, design-system packages, dark-mode product theme as a goal.

## Consequences

- Companion docs mark Milestone 8 complete; next is **Phase 9 — Testing** (after approval)
- Phase 2 “Pinia auth pending” is delivered by Milestone 8.2
- Database ADR: Milestone 8 remains **schema-neutral**
- Do not invent Dashboard/Users/Projects/Tasks/Reports UI until those frontend milestones are designed and approved

## References

- [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](../docs/MILESTONE_8_FRONTEND_FOUNDATION.md)
- [decisions/Authentication.md](Authentication.md)
- [decisions/Tech-Stack.md](Tech-Stack.md)
- [decisions/Database.md](Database.md)
- [AUTHENTICATION.md](../AUTHENTICATION.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [UI_PAGES.md](../UI_PAGES.md)
- [ROADMAP.md](../ROADMAP.md)
- [HANDOFF.md](../HANDOFF.md)
- [REQUIREMENTS.md](../REQUIREMENTS.md)
- [CODING_STANDARDS.md](../CODING_STANDARDS.md)
