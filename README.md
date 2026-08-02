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
| Phase 3.1 — Organization Foundation | ✅ Implemented |
| Phase 3.2 — User Domain Foundation | ✅ Implemented |
| Phase 3.3 — User Management APIs | ✅ Implemented |
| Phase 3.4 — Lookup APIs | Not started |
| Phase 3.5 — Search, Filtering & Pagination | Not started |
| Phase 3.6 — Authorization (RBAC) | Not started |

## Documentation Index

| Document | Description |
|----------|-------------|
| [README.md](README.md) | Docs index + status |
| [HANDOFF.md](HANDOFF.md) | **Project handoff for new AI/dev sessions** |
| [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) | **Business domain model (primary)** |
| [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Milestone 3 implementation specification |
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
- Lookups (Phase 3.4): `/api/v1/roles`, `/api/v1/departments`, `/api/v1/job-titles`
