# Decision: Tech Stack

## Status

Accepted

## Context

OpsFlow is a production-quality portfolio SaaS for project and operations management. The stack must support a separated SPA + API architecture, strong typing where practical, PostgreSQL as the system of record, and secure first-party authentication.

## Decision

### Backend

| Technology | Choice | Notes |
|------------|--------|-------|
| Framework | Laravel 13 | Keep installed major; do not downgrade to Laravel 12 |
| Language | PHP 8.3+ | Matches runtime/composer constraints |
| Database | PostgreSQL | Only application database (see `decisions/Database.md`) |
| Auth | Laravel Sanctum | SPA cookie authentication (see `decisions/Authentication.md`) |
| API style | REST | Versioned under `/api/v1` |
| Architecture | Service layer | Thin controllers; Form Requests; API Resources; PHP Enums |

### Frontend

| Technology | Choice | Notes |
|------------|--------|-------|
| Framework | Vue 3 | Composition API only |
| Language | TypeScript | Required |
| State | Pinia | Global auth/app state |
| Routing | Vue Router | Navigation guards for protected pages |
| Styling | Tailwind CSS | Utility-first UI |
| HTTP | Axios | `withCredentials: true` for Sanctum SPA auth |

### Tooling

| Tool | Purpose |
|------|---------|
| Cursor / VS Code | Development |
| Git / GitHub | Source control |
| Postman / Bruno / Insomnia | API exploration |
| pgAdmin | PostgreSQL administration |
| PHPUnit | Backend feature/unit tests |
| Vercel | Planned frontend hosting |

### Repositories

| Repository | Responsibility |
|------------|----------------|
| `opsflow-api` | Laravel REST API |
| `opsflow-web` | Vue 3 SPA (separate repo; pending) |
| `opsflow-docs` | Documentation and ADRs |

## Consequences

- Backend and frontend ship as separate deployable units
- CORS and Sanctum stateful domains must stay environment-driven
- New packages require approval (`CODING_STANDARDS.md`)
- Stack changes require updating this ADR and related docs

## References

- [PROJECT_OVERVIEW.md](../PROJECT_OVERVIEW.md)
- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [decisions/Database.md](Database.md)
- [decisions/Authentication.md](Authentication.md)
