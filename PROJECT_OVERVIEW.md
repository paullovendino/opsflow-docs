# OpsFlow

## Project Overview

OpsFlow is a modern project and operations management platform for teams and businesses. It enables organizations to manage projects, assign tasks, monitor progress, and improve operational efficiency through a centralized dashboard.

The system provides a centralized platform where administrators, managers, and employees can organize work, monitor progress, assign responsibilities, and improve operational visibility through dashboards and reports.

Organizational people model (see [docs/DOMAIN_MODEL.md](docs/DOMAIN_MODEL.md)):

```text
Organization
    ├── Departments
    ├── Job Titles
    ├── Roles (Permissions)
    └── Users
```

OpsFlow aims to demonstrate production-ready software engineering practices by following clean architecture, RESTful API design, scalable database modeling, and modern frontend development.

---

## Objectives

- Improve project visibility
- Organize tasks efficiently
- Monitor project progress
- Simplify task assignments
- Track employee productivity
- Demonstrate enterprise-grade application architecture

---

## Target Users

- Administrators
- Project Managers
- Employees

---

## Tech Stack

### Backend

- Laravel 13
- PHP 8.3+
- PostgreSQL
- Laravel Sanctum (SPA cookie authentication)
- REST API (`/api/v1`)

### Frontend

- Vue 3
- TypeScript
- Pinia
- Vue Router
- Tailwind CSS
- Axios

### Development Tools

- Cursor
- VS Code
- Git
- GitHub
- Postman
- pgAdmin
- Vercel

---

## Repositories

- `opsflow-api` — Laravel REST API
- `opsflow-web` — Vue 3 frontend (separate repository)
- `opsflow-docs` — Project documentation and decisions

---

## Version

Current Version

v1.0.0 (Development)

### Milestone progress

- ✅ Phase 1 API foundation: completed
- ✅ Phase 2 Authentication (API): completed
- Phase 2 Authentication (Vue/Pinia): ✅ Milestone 8.2
- ✅ **Milestone 3 — Organization & User Management: complete** (Phases 3.1–3.6)
- Milestone 4 — Project Management: ✅ complete (Phases 4.1–4.5)
- Milestone 5 — Task Management: ✅ Complete (5.1–5.6)
- Milestone 6 — Dashboard: ✅ Complete (6.1–6.4)
- Milestone 7 — Reports: ✅ Complete (7.1–7.4)
- Milestone 8 — Frontend Foundation: ✅ Complete (8.1–8.3)
- Milestone 9 — Frontend Modules: 🔄 In progress — ✅ Phase 9.1 Dashboard UI · next Phase 9.2 awaiting approval

This project serves as a production-quality portfolio application demonstrating full-stack software engineering skills including:

- Backend API Development
- Authentication
- Database Design
- Frontend Development
- REST API Integration
- Software Architecture
- Deployment
