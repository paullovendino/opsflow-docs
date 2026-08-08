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
| Phase 2 — Pinia auth (Vue) | ✅ Milestone 8.2 |
| **Milestone 3 — Organization & User Management** | ✅ **Complete** (3.1–3.6) |
| Phase 3.1 — Organization Foundation | ✅ Implemented |
| Phase 3.2 — User Domain Foundation | ✅ Implemented |
| Phase 3.3 — User Management APIs | ✅ Implemented |
| Phase 3.4 — Lookup APIs | ✅ Implemented |
| Phase 3.5 — Search, Filtering & Pagination (incl. Sorting) | ✅ Implemented |
| Phase 3.6 — Authorization (RBAC) | ✅ Implemented |
| Phase 4 — Project Management | ✅ **Complete** (4.1–4.5) |
| Phase 4.1 — Project Domain Foundation | ✅ Implemented |
| Phase 4.2 — Project CRUD | ✅ Implemented |
| Phase 4.3 — Project Members | ✅ Implemented |
| Phase 4.4 — Project Queries | ✅ Implemented |
| Phase 4.5 — Project Authorization | ✅ Implemented |
| **Milestone 5 — Task Management** | ✅ **Complete** (5.1–5.6) |
| Phase 5.1 — Task Domain Foundation | ✅ Implemented |
| Phase 5.2 — Task CRUD | ✅ Implemented |
| Phase 5.3 — Task Assignment | ✅ Implemented |
| Phase 5.4 — Task Queries | ✅ Implemented |
| Phase 5.5 — Task Authorization | ✅ Implemented |
| Phase 5.6 — Task Status Workflow | ✅ Implemented |
| **Milestone 6 — Dashboard** | ✅ **Complete** (6.1–6.4) |
| Phase 6.1 — Dashboard API Foundation | ✅ Implemented |
| Phase 6.2 — Project & Task Statistics | ✅ Implemented |
| Phase 6.3 — Recent Work Items | ✅ Implemented |
| Phase 6.4 — Dashboard Authorization | ✅ Implemented |
| **Milestone 7 — Reports** | ✅ **Complete** (7.1–7.4) |
| Phase 7.1 — Reports API Foundation | ✅ Implemented |
| Phase 7.2 — Project Reports | ✅ Implemented |
| Phase 7.3 — Employee Reports | ✅ Implemented |
| Phase 7.4 — Reports Authorization | ✅ Implemented |
| **Milestone 8 — Frontend Foundation** | ✅ **Complete** (8.1–8.3) |
| Phase 8.1 — Application Foundation | ✅ Implemented |
| Phase 8.2 — Authentication Foundation | ✅ Implemented |
| Phase 8.3 — UI Foundation | ✅ Implemented |
| **Milestone 9 — Frontend Modules** | ✅ **Complete** (9.1–9.5 + post-ship UX/performance) |
| Phase 9.1 — Dashboard UI | ✅ Implemented |
| Phase 9.2 — User Management | ✅ Implemented (modal CRUD + lookup cache) |
| Phase 9.3 — Project Management | ✅ Implemented (list+modal Create/Edit; Show page) |
| Phase 9.4 — Task Management | ✅ Implemented (modal CRUD; table not Kanban) |
| Phase 9.5 — Reports | ✅ Implemented (Tailwind bars; no exports) |
| **Phase 10 — Testing & QA** | ✅ **Complete** (10.1–10.3 + modal QA fix) |
| **Milestone 10 — Product Enhancements** | 📋 **Next** (not implemented) |
| **Milestone 11 — Deployment** | Future |

## Documentation Index

| Document | Description |
|----------|-------------|
| [README.md](README.md) | Docs index + status |
| [HANDOFF.md](HANDOFF.md) | **Project handoff for new AI/dev sessions** |
| [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md) | **Business domain model (primary)** |
| [docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](docs/MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Milestone 3 implementation specification |
| [docs/MILESTONE_4_PROJECT_MANAGEMENT.md](docs/MILESTONE_4_PROJECT_MANAGEMENT.md) | Milestone 4 implementation specification |
| [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md) | Milestone 5 implementation specification |
| [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md) | Milestone 6 implementation specification |
| [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md) | Milestone 7 implementation specification |
| [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md) | Milestone 8 implementation specification |
| [docs/MILESTONE_9_FRONTEND_MODULES.md](docs/MILESTONE_9_FRONTEND_MODULES.md) | Milestone 9 implementation specification |
| [docs/MILESTONE_10_TESTING_QA.md](docs/MILESTONE_10_TESTING_QA.md) | Phase 10 Testing & QA — ✅ complete |
| [docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md](docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md) | Milestone 10 Product Enhancements — 📋 planned |
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
- Projects (Milestone 4): `/api/v1/projects`, members, queries, `ProjectPolicy`
- Tasks (Milestone 5 complete): `/api/v1/tasks` CRUD, assignment, queries, `TaskPolicy`, status — see [docs/MILESTONE_5_TASK_MANAGEMENT.md](docs/MILESTONE_5_TASK_MANAGEMENT.md)
- Dashboard (Milestone 6 complete): `GET /api/v1/dashboard` — see [docs/MILESTONE_6_DASHBOARD.md](docs/MILESTONE_6_DASHBOARD.md)
- Reports (Milestone 7 complete): `/api/v1/reports/projects`, `/api/v1/reports/employees` — see [docs/MILESTONE_7_REPORTS.md](docs/MILESTONE_7_REPORTS.md)
- Next: Milestone 10 — Product Enhancements (planned) — then Milestone 11 — Deployment — see [docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md](docs/MILESTONE_10_PRODUCT_ENHANCEMENTS.md) · [HANDOFF.md](HANDOFF.md) · [TESTING.md](TESTING.md)
