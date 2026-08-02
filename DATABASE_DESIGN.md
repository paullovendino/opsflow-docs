# Database Design

## Tables

### Roles

- id
- name (unique, lowercase snake_case)
- description
- created_at
- updated_at

Seeded roles:

| name | description |
|------|-------------|
| `administrator` | Full system access |
| `project_manager` | Manage projects and tasks |
| `employee` | Assigned work and updates |

---

### Users

- id
- role_id
- first_name
- last_name
- email
- password
- avatar
- status
- created_at
- updated_at

---

### Projects

- id
- name
- description
- status
- start_date
- due_date
- created_by
- created_at
- updated_at

---

### Tasks

- id
- project_id
- assigned_to
- title
- description
- priority
- status
- due_date
- created_by
- created_at
- updated_at

---

### Activity Logs

- id
- user_id
- action
- module
- description
- ip_address
- created_at

---

## Relationships

Role

1 → Many Users

Users

1 → Many Projects

Project

1 → Many Tasks

User

1 → Many Tasks

User

1 → Many Activity Logs

---

## Morph Map

Use Laravel `Relation::enforceMorphMap`.

Registered aliases (existing models only):

- `user` → `App\Models\User`
- `role` → `App\Models\Role`

Add future aliases incrementally when models are introduced:

- `project` → `App\Models\Project`
- `task` → `App\Models\Task`
- `activity_log` → `App\Models\ActivityLog`
