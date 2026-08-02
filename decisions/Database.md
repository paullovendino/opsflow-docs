# Decision: Database

## Status

Accepted

## Context

OpsFlow needs a production-ready relational database with support for JSONB, strong relational integrity, and clean polymorphic typing for future features.

## Decision

Use **PostgreSQL** as the only application database.

### Roles

- Roles are stored in a dedicated `roles` table
- Role `name` values are unique lowercase snake_case
- Seeded roles:
  - `administrator`
  - `project_manager`
  - `employee`
- Role names are represented in code with the `App\Enums\RoleName` enum

### Schema Scope by Phase

- Phase 1 foundation includes `roles` (+ Sanctum `personal_access_tokens`)
- Users ERD columns (`role_id`, `first_name`, `last_name`, etc.) are applied in later phases
- Do not invent schema beyond the approved ERD without updating this decision

### Morph Map

- Use `Relation::enforceMorphMap`
- Register aliases only for existing models
- Current aliases: `user`, `role`
- Future aliases added when models are introduced

## Consequences

- Local and production environments must provide PostgreSQL and `pdo_pgsql`
- SQLite is not the application database target
- Enum-backed role names keep seeders and domain logic consistent
