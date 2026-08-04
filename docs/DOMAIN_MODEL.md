# OpsFlow Domain Model

**Purpose:** Primary business-domain reference for OpsFlow.  
**Audience:** Architects, developers, and AI agents designing or implementing features.  
**Scope:** Business concepts and rules — not physical schema details.  
**Schema companion:** [DATABASE_DESIGN.md](../DATABASE_DESIGN.md)  
**Milestone companions:** [MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) · [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) · [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md)

> Prefer this document when deciding *what* OpsFlow manages.  
> Prefer `DATABASE_DESIGN.md` when deciding *how* it is stored.

---

## Organizational Model

OpsFlow organizes people and permissions under a single logical organization:

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

These are **independent concepts**:

| Concept | Determines |
|---------|------------|
| Role | Permissions / authorization |
| Department | Organizational grouping |
| Job Title | Employee position / title |
| User | Person who authenticates and acts in the system |

A user belongs to:

- **one Role** (required)
- **one Department** (nullable for onboarding / flexible structures)
- **one Job Title** (nullable for onboarding / flexible structures)

---

## Domain Interaction Map

```text
Organization (logical)
    │
    ├── Department ──has many──► User
    ├── JobTitle   ──has many──► User
    ├── Role       ──has many──► User
    │
    └── User
            │
            ├── creates / owns ──► Project (Milestone 4)
            ├── member of ─────► Project (via project_members)
            ├── assigned to ───► Task (Milestone 5)
            ├── authors ───────► Remark (planned)
            └── produces ──────► Activity Log (planned)
```

Typical flow:

1. An **Organization** defines departments, job titles, and roles.
2. A **User** is onboarded with a role and optional department/job title.
3. Users create and manage **Projects** and project membership (Milestone 4).
4. Users receive **Tasks** (Milestone 5), leave **Remarks**, and generate **Activity Logs** (later milestones).

---

## Organization

### Purpose

Represents the company or tenant that owns the OpsFlow workspace. It is the logical root of departments, job titles, roles, and users.

### Responsibilities

- Bound the organizational scope of people and structure
- Provide a future home for organization-level settings and branding
- Clarify that departments, titles, roles, and users belong to one workplace context

### Relationships

- Contains many Departments
- Contains many Job Titles
- Contains many Roles
- Contains many Users

### Important Business Rules

- v1.0 assumes a **single-organization** deployment
- No multi-tenant organization switching in Milestone 3
- Organization is a **domain concept** first; a dedicated `organizations` table is not required for Milestone 3

### Future Expansion

- Organization settings and branding
- Multi-organization / multi-tenant support
- Branches and teams under the organization

---

## Department

### Purpose

Groups users by organizational unit (for example Operations, Engineering, Human Resources).

### Responsibilities

- Provide structure for directories and reporting views
- Support filtering and assignment context for users
- Remain independent from roles and job titles

### Relationships

- Belongs to Organization (logical)
- Has many Users

### Important Business Rules

- Departments are **independent** from Roles and Job Titles
- A user may belong to at most one department
- `department_id` on User is **nullable** (onboarding / flexible structures)
- Milestone 3: **read-only**, seeded by default; CRUD postponed
- Soft deletes are supported at the persistence layer
- Persistence uses human-readable `name` plus stable unique `code` (see `DATABASE_DESIGN.md`)

### Future Expansion

- Department CRUD
- Department managers / hierarchy
- Nested departments

---

## Job Title

### Purpose

Describes an employee’s position or title within the organization (for example Project Manager, Software Engineer).

### Responsibilities

- Capture positional identity separate from permissions
- Support HR-style directories and filters
- Remain independent from departments and roles

### Relationships

- Belongs to Organization (logical)
- Has many Users

### Important Business Rules

- Job Titles are **independent** from Roles and Departments
- A user may have at most one job title
- `job_title_id` on User is **nullable**
- Milestone 3: **read-only**, seeded by default; CRUD postponed
- Soft deletes are supported at the persistence layer
- Persistence uses human-readable `name` plus stable unique `code` (see `DATABASE_DESIGN.md`)
- A Job Title must **not** be treated as a Role (permissions live on Role only)

### Future Expansion

- Job Title CRUD
- Title levels / grades
- Mapping suggestions between titles and default roles (optional; not automatic permission grants)

---

## Role

### Purpose

Defines permission posture for a user. Roles answer: *what is this user allowed to do?*

### Responsibilities

- Classify users for authorization
- Drive coarse access rules for modules (starting with User Management)
- Provide stable seeded values for the product

### Relationships

- Belongs to Organization (logical)
- Has many Users

### Important Business Rules

- Roles are independent from Departments and Job Titles
- A user has **exactly one** Role
- Seeded roles (snake_case names):

| Name | Intent |
|------|--------|
| `administrator` | Full system access |
| `project_manager` | Manage projects and tasks; limited directory access |
| `employee` | Assigned work; limited self-service |

- Milestone 3: Roles remain **read-only** (lookup collection via `/api/v1/lookups/roles` in Phase 3.4)
- Advanced permission matrices and custom roles are out of scope

### Milestone 3 Authorization (coarse — Phase 3.6 — implemented)

| Role | User Management |
|------|-----------------|
| `administrator` | Full user management |
| `project_manager` | Read-only user directory |
| `employee` | View own profile |

Enforced via `UserPolicy` on User Management endpoints.

### Milestone 4 Authorization (coarse — Phase 4.5 — implemented)

| Role | Project Management |
|------|--------------------|
| `administrator` | Full access to all projects |
| `project_manager` | Full access to all projects |
| `employee` | List/view owned or member projects only |

Enforced via `ProjectPolicy` and Employee list scoping in `ProjectQuery`.

### Milestone 5 Authorization (coarse — Phase 5.5 — implemented)

| Role | Task Management |
|------|-----------------|
| `administrator` | Full access to all tasks |
| `project_manager` | Full access to all tasks |
| `employee` | List/view tasks in accessible projects; update status only when assigned to self |

Enforced via `TaskPolicy` and Employee list scoping in `TaskQuery` (Phase 5.5). See [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md).

### Future Expansion

- Permission management UI
- Advanced RBAC (abilities, policies beyond coarse role checks)
- Custom roles
- Multi-role users (explicitly deferred)

---

## User

### Purpose

A person who authenticates into OpsFlow and participates in organizational work.

### Responsibilities

- Authenticate (session via Sanctum SPA cookies)
- Carry organizational placement (role, optional department, optional job title)
- Own or participate in projects (Milestone 4); create/receive Tasks (Milestone 5); remarks and activity later
- Expose a stable profile for directories and self-service

### Relationships

- Belongs to one Role (required)
- Belongs to one Department (optional)
- Belongs to one Job Title (optional)
- Owns Projects via `created_by` (Milestone 4)
- May belong to many Projects as a member via `project_members` (Milestone 4)
- Creates Tasks via `created_by` (Milestone 5)
- May be assigned Tasks via `assigned_to` (Milestone 5; single assignee)
- Will author Remarks (planned)
- Will produce Activity Logs (planned)

### Important Business Rules

- Email is unique and used for login
- Password is stored hashed; never returned by the API
- Status controls whether the account is usable (`active` / `inactive`)
- **Inactive users must not log in** — API returns HTTP `403` with a dedicated inactive-account message
- Soft deletes are supported
- `department_id` and `job_title_id` may be null
- `role_id` is required
- `middle_name` and `avatar` are nullable
- `email_verified_at` is retained for future compatibility (not enforced in Milestone 3)
- `last_login_at` is nullable and updated when login succeeds (implementation concern)
- Referenced Role / Department / Job Title rows use **RESTRICT** on hard delete while users exist
- Multi-role and multi-department membership are out of scope

### Profile fields (domain)

- first name, middle name (optional), last name
- email, password
- avatar (optional)
- status
- last login timestamp (optional)
- role, department (optional), job title (optional)

### Future Expansion

- Invitation emails
- Force password change
- Email verification
- Richer profile / preferences
- Multiple roles or departments (if product later requires it)

---

## Project (Milestone 4)

### Purpose

A body of work with a defined scope, timeline, and ownership used to organize tasks (Milestone 5).

### Responsibilities

- Group related work under a named project
- Track project-level status and dates
- Identify a single owner
- Associate participating users as members
- Provide a unit of planning and reporting

### Relationships

- Belongs to Organization (logical)
- Owned by exactly one User (`created_by`)
- Has many member Users via `project_members`
- Has many Tasks (Milestone 5)
- May attract Remarks and Activity Logs (later)

### Important Business Rules

- Soft deletes are supported
- Status is one of: `planning`, `active`, `on_hold`, `completed`, `archived` (`ProjectStatus`)
- Default status on create: `planning`
- Owner (`created_by`) is required; FK **RESTRICT**; set from the authenticated user on create; no ownership transfer in Phase 4
- Membership is explicit via `project_members` — owner is **not** auto-added as a member (owner ≠ membership)
- `joined_at` is server-generated; client-supplied values are ignored
- Duplicate membership is rejected (HTTP `409`)
- Only active, non-soft-deleted users may be added as members
- No member roles, invitation workflow, or permissions on the membership pivot
- Employee project visibility (Phase 4.5): projects they **own** or are a **member** of
- Schema / API defined in [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) and [decisions/Project-Management.md](../decisions/Project-Management.md)

### Implementation status (Milestone 4)

| Phase | Status |
|-------|--------|
| 4.1 Domain Foundation | ✅ Implemented |
| 4.2 Project CRUD | ✅ Implemented |
| 4.3 Project Members | ✅ Implemented |
| 4.4 Project Queries | ✅ Implemented |
| 4.5 Project Authorization | ✅ Implemented |

### Future Expansion

- Ownership transfer
- Member roles / invitations
- Templates, dashboards, richer archiving workflows beyond `archived` status

---

## Task (Milestone 5)

### Purpose

A unit of assignable work within a project.

### Responsibilities

- Capture title, description, priority, status, and due date
- Assign responsibility to at most one user
- Drive day-to-day execution visibility within a project

### Relationships

- Belongs to exactly one Project (`project_id`)
- Optionally assigned to one User (`assigned_to`, nullable)
- Created by one User (`created_by`)
- May attract Remarks and Activity Logs (later)

### Important Business Rules

- Soft deletes are supported
- Status is one of: `todo`, `in_progress`, `in_review`, `blocked`, `completed`, `cancelled` (`TaskStatus`)
- Default status on create: `todo` (status not accepted on create/update; use status patch)
- Priority is one of: `low`, `medium`, `high`, `urgent` (`TaskPriority`); default on create when omitted: `medium`
- `created_by` is required; FK **RESTRICT**; set from the authenticated user on create; not transferable in Milestone 5
- `project_id` is required on create; not changeable after create in Milestone 5; FK **RESTRICT**
- Single assignee only; `assigned_to` nullable; when set, assignee must be active, not soft-deleted, and project owner **or** member
- Assignment targets Users, not Departments or Job Titles
- Milestone 5 does not enforce a restricted status transition graph
- Employee task visibility (Phase 5.5): tasks in projects they **own** or are a **member** of; may update status only when assigned to self
- Schema / API defined in [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md) and [decisions/Task-Management.md](../decisions/Task-Management.md)

### Implementation status (Milestone 5)

| Phase | Status |
|-------|--------|
| 5.1 Task Domain Foundation | ✅ Implemented |
| 5.2 Task CRUD | ✅ Implemented |
| 5.3 Task Assignment | ✅ Implemented |
| 5.4 Task Queries | ✅ Implemented |
| 5.5 Task Authorization | ✅ Implemented |
| 5.6 Task Status Workflow | ✅ Implemented |

### Future Expansion

- Subtasks, dependencies, kanban views, time tracking
- Multiple assignees, labels, checklists, attachments, recurring tasks

---

## Remark (planned)

### Purpose

A comment or note left by a user on a work item (for example a project or task) to capture discussion and context.

### Responsibilities

- Preserve conversational / contextual commentary
- Attribute remarks to an authoring User
- Attach remarks to a subject entity (polymorphic or typed — decided later)

### Relationships

- Authored by a User
- Belongs to a subject such as Project or Task (design TBD)
- May appear in Activity Logs when created/updated

### Important Business Rules

- Not implemented in Milestone 3
- Remarks are distinct from Activity Logs (human commentary vs system audit trail)

### Future Expansion

- Mentions, attachments, edit history, reactions

---

## Activity Log (planned)

### Purpose

An auditable record of significant actions taken in the system.

### Responsibilities

- Record who did what, in which module, with a description
- Support operational visibility and accountability
- Eventually feed true audit-style recent-activity views

### Relationships

- Produced by a User (when an actor exists)
- May reference Projects, Tasks, Users, or other modules

### Important Business Rules

- **Not** implemented in Milestones 3–6
- Distinct from Remarks: logs are system/audit oriented
- Distinct from Milestone 6 Dashboard **recent work items** (derived from Project/Task `updated_at`)
- Retention and detail level decided in a later milestone

### Future Expansion

- Immutable audit policies, export, filters by module/actor

---

## Dashboard (Milestone 6 — ✅ Implemented)

### Purpose

A **read-only operational summary** for authenticated users: project/task statistics, card metrics (overdue, assigned-to-me), and a recent work-items feed.

### Responsibilities

- Aggregate counts suitable for statistics cards and frontend charts
- Respect the same project/task visibility rules as Milestones 4–5
- Surface recently updated Projects and Tasks (derived feed)

### Relationships

- Not a persisted entity — computed from Project and Task
- Does **not** require Activity Log or Remark tables in Milestone 6

### Important Business Rules

- Single endpoint: `GET /api/v1/dashboard` (**implemented**)
- No new database tables in Milestone 6
- Soft-deleted projects/tasks excluded
- Admin/PM: organization-wide aggregates; Employee: owned-or-member scope
- Overdue tasks: `due_date` &lt; today and status not `completed` / `cancelled`
- Spec: [MILESTONE_6_DASHBOARD.md](MILESTONE_6_DASHBOARD.md) · ADR: [decisions/Dashboard.md](../decisions/Dashboard.md)

### Future Expansion

- Activity-Log-backed feeds, date-range analytics, Reports (Phase 7), Vue widgets, caching

---

## How Entities Interact in OpsFlow

1. **Structure first:** Organization defines Departments, Job Titles, and Roles.
2. **People next:** Users are created with a Role and optional Department/Job Title.
3. **Authorization:** Role decides what the User may do (User Management in Milestone 3; Projects in Milestone 4; Tasks in Milestone 5; Dashboard view in Milestone 6).
4. **Work:** Users create Projects, add members, create/assign Tasks; Dashboard summarizes that work; later leave Remarks and generate Activity Logs.
5. **Independence preserved:** Changing a user’s Job Title must not silently change permissions; changing Role must not silently change department membership; project membership does not grant system Role permissions; task assignment does not grant Project Management mutate rights beyond the Task matrix; Dashboard does not invent separate ACLs beyond Project/Task visibility.

---

## Related Documents

| Document | Use |
|----------|-----|
| [DATABASE_DESIGN.md](../DATABASE_DESIGN.md) | Physical tables, columns, FKs |
| [API_SPECIFICATION.md](../API_SPECIFICATION.md) | HTTP contracts |
| [ARCHITECTURE.md](../ARCHITECTURE.md) | System layering |
| [decisions/Organization-User-Management.md](../decisions/Organization-User-Management.md) | Milestone 3 ADR |
| [decisions/Project-Management.md](../decisions/Project-Management.md) | Milestone 4 ADR |
| [decisions/Task-Management.md](../decisions/Task-Management.md) | Milestone 5 ADR |
| [decisions/Dashboard.md](../decisions/Dashboard.md) | Milestone 6 ADR |
| [decisions/Database.md](../decisions/Database.md) | Database ADR |
| [MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md](MILESTONE_3_ORGANIZATION_USER_MANAGEMENT.md) | Milestone 3 implementation specification |
| [MILESTONE_4_PROJECT_MANAGEMENT.md](MILESTONE_4_PROJECT_MANAGEMENT.md) | Milestone 4 implementation specification |
| [MILESTONE_5_TASK_MANAGEMENT.md](MILESTONE_5_TASK_MANAGEMENT.md) | Milestone 5 implementation specification |
| [MILESTONE_6_DASHBOARD.md](MILESTONE_6_DASHBOARD.md) | Milestone 6 implementation specification |
