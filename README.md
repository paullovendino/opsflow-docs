# OpsFlow Documentation

Central documentation for the OpsFlow SaaS platform.

## Repositories

| Repository | Purpose |
|------------|---------|
| `opsflow-api` | Laravel 13 REST API |
| `opsflow-web` | Vue 3 frontend (separate) |
| `opsflow-docs` | This documentation set |

## Current Status

| Milestone | Status |
|-----------|--------|
| Phase 1 — Project Setup (API foundation) | ✅ Completed |
| Phase 2 — Authentication (API) | ✅ Completed |
| Phase 2 — Pinia auth (Vue) | Pending |
| **Milestone 3 — Organization & User Management** | ✅ **Complete** (3.1–3.6) |
| Phase 3.1 — Organization Foundation | ✅ Implemented |
| Phase 3.2 — User Domain Foundation | ✅ Implemented |
| Phase 3.3 — User Management APIs | ✅ Implemented |
| Phase 3.4 — Lookup APIs | ✅ Implemented |
| Phase 3.5 — Search, Filtering & Pagination (incl. Sorting) | ✅ Implemented |
| Phase 3.6 — Authorization (RBAC) | ✅ Implemented |
| Phase 4 — Project Management | In progress (4.1–4.3 done) |
| Phase 4.1 — Project Domain Foundation | ✅ Implemented |
| Phase 4.2 — Project CRUD | ✅ Implemented |
| Phase 4.3 — Project Members | ✅ Implemented |
| Phase 4.4 — Project Queries | ⏳ Pending (next) |
| Phase 4.5 — Project Authorization | ⏳ Pending |

## Documentation Index

| Document | Description |
|----------|-------------|
| [README.md](README.md) | Docs index + status |
| [HANDOFF.md](HANDOFF.md) | **Project handoff for new AI/dev sessions** |
| [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) | **Business domain model (primary)** |
| [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Milestone 3 implementation specification |
| [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) | Milestone 4 implementation specification |
| [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) | Product overview and tech stack |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Functional and non-functional requirements |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System and backend architecture |
| [AUTHENTICATION.md](AUTHENTICATION.md) | Authentication module guide |
| [API_SPECIFICATION.md](API_SPECIFICATION.md) | REST API endpoints |
| [DATABASE_DESIGN.md](DATABASE_DESIGN.md) | ERD and table design |
| [CODING_STANDARDS.md](CODING_STANDARDS.md) | Coding conventions |
| [CURSOR_RULES.md](CURSOR_RULES.md) | Cursor agent rules |
| [SETUP.md](SETUP.md) | Local setup and installation |
| [TESTING.md](TESTING.md) | Testing guide |
| [ROADMAP.md](ROADMAP.md) | Phase roadmap (canonical) |
| [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md) | Roadmap pointer / summary |
| [CHANGELOG.md](CHANGELOG.md) | Release / milestone changelog |
| [UI_PAGES.md](UI_PAGES.md) | Frontend page inventory |
| [decisions/](decisions/) | Architecture Decision Records |

## Quick Links

- Domain model: [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)
- Auth: `/api/v1/auth/login`, `/api/v1/auth/logout`, `/api/v1/auth/me`
- Users (implemented): `/api/v1/users`, `/api/v1/users/{user}`, `/api/v1/users/{user}/status`
- Health: `GET /api/v1/health`
- CSRF cookie: `GET /sanctum/csrf-cookie`
- Lookups (Phase 3.4): `/api/v1/lookups/roles`, `/api/v1/lookups/departments`, `/api/v1/lookups/job-titles`
- Users list (Phase 3.5): search / filters / sorting / pagination on `GET /api/v1/users`
- Authorization (Phase 3.6): coarse User Management policies (`UserPolicy`)
- Next: Phase 4.4 — Project Queries — see [HANDOFF.md](HANDOFF.md) · [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md)
