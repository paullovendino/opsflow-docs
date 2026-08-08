# Milestone 8 — Frontend Foundation

**Status:** ✅ Milestone 8 complete (Phases 8.1–8.3)  
**Product version:** v1.0.0 (Development)  
**Last updated:** 2026-08-04

> Implementation specification for Milestone 8 (`opsflow-web`).  
> ADR: [decisions/Frontend-Foundation.md](../decisions/Frontend-Foundation.md)  
> Auth contract: [AUTHENTICATION.md](../AUTHENTICATION.md) · [decisions/Authentication.md](../decisions/Authentication.md)  
> Tech stack: [decisions/Tech-Stack.md](../decisions/Tech-Stack.md)  
> Prerequisite: Milestone 7 complete (API Milestones 1–7 available); `opsflow-web` create-vue scaffold exists

---

## 1. Goal

Establish the **production SPA foundation** for OpsFlow so the Vue app can authenticate against the Laravel Sanctum API and render a durable app shell — without building feature module pages yet.

Milestone 8 delivers:

- Approved frontend architecture (folders, routing, layouts, TypeScript conventions)
- Axios + Sanctum CSRF credentialed API client
- Pinia auth store with login / logout / `/me` bootstrap / session restore
- Route guards (guest vs authenticated)
- App shell UI foundation (sidebar, topbar, responsive nav, loading / empty / error / toast patterns)
- Removal of create-vue welcome scaffolding as part of implementation

It does **not** implement Dashboard, Users, Projects, Tasks, or Reports pages (those consume APIs in later frontend milestones). It does **not** expand backend schema or API contracts.

---

## 2. Current baseline (`opsflow-web`)

| Area | State |
|------|--------|
| Scaffold | Vue 3 + Vite + TypeScript create-vue template |
| Runtime deps | `vue` only |
| Missing | Pinia, Vue Router, Axios, Tailwind CSS |
| Product code | None (welcome components only) |
| Build | `type-check` / `build-only` currently pass on scaffold |

Implementation **starts from this scaffold** and replaces welcome UI with OpsFlow foundation.

---

## 3. Implementation phasing

| Phase | Scope | Status |
|-------|--------|--------|
| **8.1 Application Foundation** | Folder structure; Vue/Pinia/Router/Tailwind install; env; Axios client; types; replace scaffold entry | ✅ Implemented |
| **8.2 Authentication Foundation** | CSRF → login → session; `/me` bootstrap; auth store; guards; Guest vs Auth layouts; login page | ✅ Implemented |
| **8.3 UI Foundation** | Sidebar; topbar; app shell; responsive nav; loading/empty/error pages; toast; shared UI components | ✅ Implemented |

**Phase order is mandatory.** Do not start a later phase until the prior phase is complete and approved.

---

## 4. Approved stack

| Technology | Choice | Notes |
|------------|--------|-------|
| Framework | Vue 3 | Composition API **only** (`<script setup>`) |
| Language | TypeScript | Strict; match existing `tsconfig` hardening |
| State | Pinia | Auth + UI (toast) stores |
| Routing | Vue Router | History mode; navigation guards |
| HTTP | Axios | `withCredentials: true`; CSRF header |
| Styling | Tailwind CSS | Utility-first; no UI component framework |

**No UI framework** (PrimeVue, Vuetify, Element Plus, Headless UI packages, etc.) unless a future ADR explicitly approves one.

**No extra packages** beyond the approved stack (+ Tailwind Vite plugin) without approval.

---

## 4.1 Approved decisions (locked for implementation)

| Decision | Approved choice |
|----------|-----------------|
| Layout wiring | **Nested Vue Router routes** with parent layout components (`GuestLayout` / `AuthLayout`). Do **not** use a runtime `meta.layout` switcher. |
| Tailwind install | **`@tailwindcss/vite`** with Vite 8 (not a separate PostCSS-only pipeline unless the plugin path fails and is re-approved). |
| Env files | Ship **`.env.example`**; local secrets in **`.env`** (gitignored). Do not require `.env.development`. |
| Axios timeout | **15000 ms** |
| Logout placement | **Topbar** user area (name/email + Logout). Sidebar = brand + nav only. |
| `/403` access | Available to **guest and authenticated** users; **no** `requiresAuth` / `guest` meta. |
| Global `403` interceptor | **Do not** auto-navigate to `/403`. Reject the promise with the envelope; callers handle UX. Login form owns inactive / already-authenticated messages. Optional toast only when a caller does not handle the error. |
| Login `403` cases | `Account is inactive.` → show on **LoginView**. `Already authenticated.` → `refreshUser()` via `/me` and redirect to `home` (guest guard should usually prevent this). |
| Login `401` | Show API message / generic invalid credentials on **LoginView** (not a toast-only flow). |
| CSRF | Always `GET /sanctum/csrf-cookie` **immediately before** login (also after logout before a new login, because logout regenerates the CSRF token). |
| AuthUser shape | Normalize login `data.user` and `/me` `data` into one `AuthUser` matching `UserResource` fields needed by the shell: `id`, `first_name`, `middle_name`, `last_name`, `full_name`, `email`, `avatar`, `status`, `last_login_at`, nested `role` (at least `id`, `name`), optional `department` / `job_title`. |
| Nav placeholders | Sidebar **may** show disabled labels for Dashboard / Users / Projects / Tasks / Reports; **no** routes and **no** API calls. |
| `placeholders/` folder | **Omit** unless a later phase needs it; do not invent “coming soon” feature routes in Milestone 8. |

---

## 5. Phase 8.1 — Application Foundation

### 5.1 Target folder structure

```text
opsflow-web/
├── .env.example
├── .env                        # local only (gitignored)
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig*.json
├── env.d.ts
├── public/
└── src/
    ├── main.ts
    ├── App.vue
    ├── assets/                 # global CSS entry for Tailwind
    ├── components/
    │   ├── layout/             # AppSidebar, AppTopbar (wired in 8.3)
    │   └── ui/                 # Shared primitives (wired in 8.3)
    ├── composables/            # useAuth, useToast, …
    ├── layouts/
    │   ├── GuestLayout.vue
    │   └── AuthLayout.vue
    ├── views/
    │   ├── auth/
    │   │   └── LoginView.vue
    │   ├── AppHomeView.vue     # authenticated shell home (not Dashboard feature)
    │   └── errors/
    │       ├── NotFoundView.vue
    │       └── ForbiddenView.vue
    ├── router/
    │   ├── index.ts
    │   └── guards.ts
    ├── stores/
    │   ├── auth.ts
    │   └── ui.ts
    ├── services/
    │   ├── http.ts             # Axios instance
    │   └── authService.ts
    ├── types/
    │   ├── api.ts              # envelope, pagination meta
    │   └── auth.ts             # AuthUser / Role shapes from API
    └── utils/
        ├── cookies.ts          # XSRF cookie read helper (if not relying solely on Axios defaults)
        └── errors.ts           # map API envelope → UI messages
```

**Do not** invent alternate top-level trees. Directories for 8.3 components may start as stubs in 8.1.

### 5.2 Vue architecture

```text
main.ts
  → createApp(App)
  → use(createPinia())
  → use(router)
  → mount('#app')

App.vue
  → <RouterView /> only (+ global ToastHost)

Router
  → nested layout routes (`GuestLayout` | `AuthLayout`)
  → child views

Pinia stores
  → auth (session user)
  → ui (toasts, mobile nav open state)

services/*
  → HTTP / auth API calls (no business UI)

composables/*
  → thin wrappers over stores/services for views
```

**Rules:**

- No Options API
- No raw `fetch` for API calls (Axios only)
- Views stay thin; call stores/composables, not Axios directly (except during early 8.1 wiring if needed — prefer services by end of 8.1)
- Do not put API URLs as magic strings in components — centralize in services

### 5.3 Layout architecture (contracts for 8.2–8.3)

| Layout | Used when | Structure |
|--------|-----------|-----------|
| `GuestLayout` | Unauthenticated routes (`/login`) | Centered content; no sidebar |
| `AuthLayout` | Authenticated routes | `AppShell` = Sidebar + Topbar + main `<RouterView />` |

**Approved wiring:** nested routes — parent route renders the layout; children render in the layout’s `<RouterView />`.

### 5.4 Route architecture (foundation)

| Path | Name | Meta | View | Notes |
|------|------|------|------|-------|
| `/login` | `login` | `guest: true` | `LoginView` | Guest only |
| `/` | `home` | `requiresAuth: true` | `AppHomeView` | Authenticated shell home — **not** Dashboard API page |
| `/403` | `forbidden` | *(none)* | `ForbiddenView` | Soft deny page; guest + auth |
| `/:pathMatch(.*)*` | `not-found` | — | `NotFoundView` | 404 |

**Explicitly no routes in Milestone 8 for:** `/dashboard`, `/users`, `/projects`, `/tasks`, `/reports` (feature pages deferred).

Sidebar may show **disabled** future nav labels (Dashboard, Users, Projects, Tasks, Reports) for shell realism, but they must **not** navigate to real feature pages or call those APIs.

### 5.5 Environment variables

| Variable | Required | Purpose | Example |
|----------|----------|---------|---------|
| `VITE_API_BASE_URL` | Yes | Laravel origin (no trailing slash) | `http://localhost:8000` |
| `VITE_APP_NAME` | Yes | Branding | `OpsFlow` |

Document in `.env.example`. Type via `ImportMetaEnv` in `env.d.ts`. Local overrides live in `.env` (gitignored).

**Local auth strategy (approved):** Keep SPA on `http://localhost:5173` and API on `http://localhost:8000` using existing CORS + `SANCTUM_STATEFUL_DOMAINS` (see Authentication ADR). Do **not** require a Vite proxy for Milestone 8. Optional proxy may be added later if cookie debugging warrants it — out of default scope.

### 5.6 API client strategy

- Single Axios instance in `services/http.ts`
- `baseURL` = `VITE_API_BASE_URL`
- All versioned API calls use paths under `/api/v1/...`
- Sanctum CSRF uses `/sanctum/csrf-cookie` on the **same API origin** (not under `/api/v1`)
- Parse the standard API envelope:

```ts
type ApiEnvelope<T> = {
  success: boolean
  message: string
  data: T
  errors: Record<string, string[]> | null
  meta: Record<string, unknown> | null
}
```

- Login success user is under `data.user` (existing API contract)
- `/me` returns user under `data` as Resource (existing intentional asymmetry — `authService` **must** normalize both into `AuthUser`)

### 5.7 Axios configuration (approved)

| Setting | Value |
|---------|--------|
| `withCredentials` | `true` |
| CSRF | `withXSRFToken: true`, `xsrfCookieName: 'XSRF-TOKEN'`, `xsrfHeaderName: 'X-XSRF-TOKEN'` (Laravel defaults). Required for cross-origin SPA→API (`localhost:5173` → `:8000`); Axios 1.x does not send the XSRF header on cross-origin requests unless `withXSRFToken` is true. Cookie value is URL-encoded; Axios decodes when using these options. |
| Accept | `application/json` |
| Timeout | **15000** ms |

**Response interceptors (minimum):**

| Status | Behavior |
|--------|----------|
| `401` | If request is **not** the login attempt: clear auth store; redirect to `login` when not already on a guest route (avoid loops). Login `401` (invalid credentials) is handled by `LoginView` / `auth.login` — do **not** treat it as a forced navigation away from `/login`. |
| `403` | **No auto-navigation to `/403`.** Reject with envelope; LoginView / callers own messaging. |
| `422` | Reject with field `errors` for form binding |
| `429` | Toast rate-limit message (LoginView may also show inline) |
| Network / 5xx | Toast generic error |

Do not invent new backend error shapes.

### 5.8 TypeScript conventions

- Prefer `interface` for object shapes; `type` for unions/aliases
- Export API/domain types from `src/types/`
- No `any` unless unavoidable and localized
- Use `import type` for type-only imports
- Keep `@/` path alias
- Enum-like role/status strings must match API string values (`administrator`, `project_manager`, `employee`, `active`, `inactive`, …) — mirror backend enums as string union types (no duplicate business logic)

### 5.9 Phase 8.1 deliverables checklist

- [x] Install Pinia, Vue Router, Axios, Tailwind CSS via **`@tailwindcss/vite`**
- [x] Create folder structure above (no `placeholders/` directory)
- [x] `.env.example` + typed env
- [x] `http.ts` Axios instance (`withCredentials`, XSRF names, **15000** ms timeout)
- [x] `types/api.ts` + `types/auth.ts` (`AuthUser`)
- [x] Router skeleton with **nested** Guest/Auth layouts + placeholder views
- [x] Replace create-vue welcome UI from `App.vue` / remove unused scaffold components
- [x] `index.html` title → OpsFlow
- [x] App still type-checks and builds

**Out of scope for 8.1:** Working login UX polish, full shell chrome, feature pages.

---

## 6. Phase 8.2 — Authentication Foundation

### 6.1 Sanctum CSRF flow

1. Immediately before login: `GET {API}/sanctum/csrf-cookie`
2. Axios sends credentials + `X-XSRF-TOKEN`
3. `POST /api/v1/auth/login` with `{ email, password }`
4. On success, normalize `data.user` into auth store as `AuthUser`
5. Navigate to `home` (`/`)

Always re-fetch the CSRF cookie before a new login after logout (logout regenerates the CSRF token).

### 6.2 Login

| Concern | Rule |
|---------|------|
| Route | `/login` |
| Layout | `GuestLayout` |
| Fields | Email, password |
| Validation | Client: required + basic email; server: `422` field errors inline |
| Inactive account | API `403` + `Account is inactive.` — **inline on LoginView** (not `/403` page) |
| Invalid credentials | API `401` — inline on LoginView |
| Already authenticated | Prefer guest guard; if API returns `403` `Already authenticated.` → bootstrap/`/me` then redirect `home` |
| Rate limit | API `429` — inline and/or toast |
| Already on `/login` while authed | Guard redirects to `/` |

### 6.3 Logout

- `POST /api/v1/auth/logout` (`auth:sanctum`) — requires CSRF cookie/header like other mutating calls
- Clear Pinia auth state
- Redirect to `/login`
- Ignore benign failures after local clear if session already gone (still force guest UX)
- Affordance: **Topbar** Logout control (see §4.1)

### 6.4 `/me` bootstrap & session persistence

- Persistence is **cookie/session based** (no localStorage auth tokens)
- On app start (before first authenticated navigation resolves):

```text
bootstrapAuth()
  → attempt GET /api/v1/auth/me once (even when landing on /login)
  → 200: normalize `data` → AuthUser; mark bootstrapped
  → 401: clear user; mark bootstrapped
```

- Router `beforeEach` waits until `auth.isBootstrapped` (or awaits bootstrap promise) to avoid flicker / wrong redirects
- Do **not** store passwords or Sanctum tokens in Pinia/localStorage

### 6.5 Auth store (Pinia) — approved shape

```text
state:
  user: AuthUser | null
  isBootstrapped: boolean
  isLoading: boolean

getters:
  isAuthenticated: boolean  // user !== null
  fullName / email / roleName helpers as needed

actions:
  bootstrap()
  login(email, password)    // csrf → login → setUser from data.user
  logout()
  setUser(user | null)
  clear()
```

`AuthUser` fields: see §4.1 (align with `UserResource`).

### 6.6 Route guards

| Meta | Rule |
|------|------|
| `requiresAuth: true` | If not authenticated after bootstrap → redirect `login` (optional `redirect` query) |
| `guest: true` | If authenticated → redirect `home` |
| *(none)* on `/403` | No auth gate |

Implement in `router/guards.ts`; keep `router/index.ts` thin.

### 6.7 Guest vs Auth layouts

- Guest: login only in Milestone 8
- Auth: wraps `AppHomeView` (shell home) via nested routes
- Switching layouts must not remount Pinia

### 6.8 Phase 8.2 deliverables checklist

- [x] `authService.ts` (csrf, login, logout, me)
- [x] `stores/auth.ts`
- [x] `LoginView` wired end-to-end against local API
- [x] Bootstrap on startup
- [x] Guards for guest/auth routes
- [x] GuestLayout + AuthLayout shells (chrome can be minimal until 8.3)
- [x] Manual verification: login, refresh restores session via cookie + `/me`, logout

**Out of scope for 8.2:** Dashboard/Users/Projects/Tasks/Reports UI; registration; password reset.

---

## 7. Phase 8.3 — UI Foundation

### 7.1 App shell

`AuthLayout` composition:

```text
┌────────────┬──────────────────────────┐
│ Sidebar    │ Topbar                   │
│ (brand +   ├──────────────────────────┤
│  nav)      │ Main content (RouterView)│
│            │                          │
└────────────┴──────────────────────────┘
```

### 7.2 Sidebar

- Brand: **OpsFlow** (product name visible)
- Active item: **Home**
- Optional disabled placeholders: Dashboard, Users, Projects, Tasks, Reports (no routes / no API)
- **No** logout control in sidebar (logout lives in topbar — §4.1)
- Collapse / drawer on small screens (see responsive)

### 7.3 Topbar

- Page title (from route `meta.title` or view)
- Authenticated user display (name / email)
- **Logout** control
- Mobile menu toggle controlling sidebar drawer

### 7.4 Responsive navigation

| Breakpoint guidance | Behavior |
|---------------------|----------|
| Desktop (`md+`) | Persistent sidebar |
| Mobile | Sidebar off-canvas; toggled from topbar; closes on navigate |

Use Tailwind responsive utilities. No separate mobile app. Mobile drawer open state may live in `stores/ui.ts`.

### 7.5 Loading states

| Context | Pattern |
|---------|---------|
| Auth bootstrap | Full-app or layout-level spinner until `isBootstrapped` |
| Login submit | Button loading / disabled |
| Future lists | Shared `AppSpinner` / skeleton-friendly slots (components only — no feature lists yet) |

### 7.6 Empty states

- Shared `AppEmptyState` (title, description, optional action slot)
- Used on `AppHomeView` with a short “foundation ready” / placeholder message (not Dashboard widgets)

### 7.7 Error pages

| Page | Route | Purpose |
|------|-------|---------|
| Not Found | catch-all | Unknown routes |
| Forbidden | `/403` | Explicit soft-deny page (not the default for login `403`s) |

Prefer toast + stay on page for API 5xx in Milestone 8. Login auth errors stay on `LoginView`.

### 7.8 Toast strategy

- **No third-party toast library** in Milestone 8
- `stores/ui.ts` queue: `{ id, type: 'success' | 'error' | 'info', message }`
- `ToastHost` global component in `App.vue`
- `useToast()` composable
- Auto-dismiss after a short timeout; allow manual dismiss
- Login validation / inactive / invalid-credentials prefer **inline** form errors; toasts are secondary

### 7.9 Global / shared UI components (minimum set)

| Component | Role |
|-----------|------|
| `AppButton` | Primary / secondary / danger; loading prop |
| `AppInput` | Text input + label + error message |
| `AppSpinner` | Inline / block loading |
| `AppEmptyState` | Empty placeholder |
| `AppToast` / `ToastHost` | Notifications |
| `AppSidebar` | Shell nav |
| `AppTopbar` | Shell header |

Keep components presentational; no feature-domain props beyond shell needs.

### 7.10 Phase 8.3 deliverables checklist

- [x] Sidebar + Topbar + Auth shell polish
- [x] Responsive drawer behavior
- [x] Loading / empty / error views wired
- [x] Toast store + host
- [x] Shared UI primitives above
- [x] Create-vue CSS/theme leftovers removed in favor of Tailwind-based shell
- [x] Type-check + production build pass

---

## 8. Backend / schema impact

| Concern | Milestone 8 |
|---------|-------------|
| New API endpoints | **None** |
| API contract changes | **None** (consume existing auth endpoints) |
| Database / migrations | **None** |
| Morph map | **No change** |

Frontend-only milestone. Database ADR remains schema-neutral for M8.

---

## 9. Out of Scope (Future Work)

Explicitly deferred (do **not** implement in Milestone 8):

- Dashboard page / `GET /api/v1/dashboard` UI → Milestone **9.1** ✅
- Users / Lookups admin UI → Milestone **9.2** ✅
- Projects / Members UI → Milestone **9.3** ✅
- Tasks UI → Milestone **9.4** ✅
- Reports UI → Milestone **9.5** ✅
- Charts, tables for domain modules, advanced filters (chart libs remain deferred)
- Registration, password reset, email verification, remember-me
- Token auth / mobile clients
- Frontend automated test suite (Vitest/Cypress/Playwright) — later **Phase 10 — Testing & QA**
- Deployment / production hosting — later **Phase 11 — Deployment**
- UI component libraries / design systems packages
- Dark-mode product theme (Tailwind may support later; not a M8 goal)
- Activity Logs / Remarks UI
- i18n

---

## 10. Testing expectations (documentation only)

Milestone 8 does **not** require a frontend test suite.

**Manual acceptance checks (implementation):**

1. Guest visits `/` → redirected to `/login`
2. Login success → `/` shell with user identity
3. Refresh → session restored via cookie + `/me`
4. Logout → `/login`; protected routes blocked
5. Inactive user → visible `403` inactive message
6. Invalid credentials → `401` message
7. Unknown path → Not Found view
8. Mobile width → sidebar drawer works
9. `npm run type-check` and `npm run build` succeed

Broader automated testing is **Phase 9**.

---

## 11. Acceptance Criteria

### Design package

- [x] `docs/MILESTONE_8_FRONTEND_FOUNDATION.md` exists
- [x] `decisions/Frontend-Foundation.md` ADR exists
- [x] Companion docs synchronized
- [x] Implementation approved by user before coding

### Implementation (after approval)

- [x] Phases 8.1–8.3 complete per checklists
- [x] Sanctum cookie auth wired against local `opsflow-api` contracts
- [x] No feature module pages shipped
- [x] No new backend schema/API invented
- [x] Docs flipped to ✅ Implemented as phases complete
- [x] `npm run type-check` and `npm run build` succeed

---

## 12. Related documents

| Document | Use |
|----------|-----|
| [decisions/Frontend-Foundation.md](../decisions/Frontend-Foundation.md) | ADR |
| [decisions/Authentication.md](../decisions/Authentication.md) | Sanctum SPA decisions |
| [decisions/Tech-Stack.md](../decisions/Tech-Stack.md) | Approved stack |
| [AUTHENTICATION.md](../AUTHENTICATION.md) | Auth module guide |
| [UI_PAGES.md](../UI_PAGES.md) | Page inventory |
| [ARCHITECTURE.md](../ARCHITECTURE.md) | System architecture |
| [CODING_STANDARDS.md](../CODING_STANDARDS.md) | Conventions |
| [ROADMAP.md](../ROADMAP.md) | Phase roadmap |
| [HANDOFF.md](../HANDOFF.md) | Session handoff |
