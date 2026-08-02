# Functional Requirements

## Authentication

- [x] User login (`POST /api/v1/auth/login`)
- [x] User logout (`POST /api/v1/auth/logout`)
- [x] Protected routes (`auth:sanctum`)
- [x] Current authenticated user (`GET /api/v1/auth/me`)
- [ ] Coarse role-based access for User Management (Milestone 3 — Planned)
- [ ] Inactive users cannot log in (`403` dedicated inactive-account response) (Milestone 3 — Planned)
- [ ] Frontend Pinia authentication

---

## Organization Structure

> Milestone 3 — Planned (read-only reference data)

- [ ] Seeded Departments (list/show) — approved seed list
- [ ] Seeded Job Titles (list/show) — approved seed list
- [ ] Seeded Roles remain available (list/show)
- [ ] Lookup endpoints accessible to all authenticated users
- [ ] Users belong to one Role; optional Department and Job Title

Department CRUD, Job Title CRUD, teams, branches, and organization settings are out of scope for Milestone 3.

---

## Dashboard

- Project summary
- Task summary
- Recent activities
- Statistics cards

---

## User Management

> Milestone 3 — Planned

- [ ] Create users
- [ ] Update users
- [ ] Soft-delete users
- [ ] Activate / deactivate users (`PATCH /users/{id}/status`)
- [ ] Assign role (required)
- [ ] Assign department (optional)
- [ ] Assign job title (optional)
- [ ] List users with filters: `search`, `role_id`, `department_id`, `job_title_id`, `status` + pagination
- [ ] Coarse authorization:
  - Administrator — full user management
  - Project Manager — read-only directory
  - Employee — view own profile

---

## Project Management

- Create project
- Update project
- Delete project
- Archive project
- View project details

---

## Task Management

- Create task
- Update task
- Delete task
- Assign employee
- Set priority
- Set due date
- Update status
- Add description

---

## Remarks

> Planned — future

- Author remarks on work items (design in [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md))

---

## Activity Logs

- User actions
- Task updates
- Project updates

---

# Non-Functional Requirements

- Responsive UI
- RESTful API
- Secure Authentication
- Fast Response Time
- Scalable Architecture
- Clean Code
- Mobile Friendly
- Production Ready
- Documentation-first milestone design before implementation
