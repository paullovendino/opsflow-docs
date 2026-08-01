# Database Design

## Tables

### Roles

- id
- name
- description
- created_at
- updated_at

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