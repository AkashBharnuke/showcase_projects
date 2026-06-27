# Wish Me Surprise

> A full-stack web application that evolved into a production-style containerized deployment, exploring backend engineering, infrastructure, and operational practices.

<!-- ![Angular](https://img.shields.io/badge/Angular-17-DD0031?logo=angular&logoColor=white)
![Django](https://img.shields.io/badge/Django-5-092E20?logo=django&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![NGINX](https://img.shields.io/badge/NGINX-009639?logo=nginx&logoColor=white) -->

![Web Application](https://img.shields.io/badge/Project-Web%20Application-6f42c1)
![Dockerized](https://img.shields.io/badge/Containerized-Docker-2496ED?logo=docker&logoColor=white)
![Production Deployment](https://img.shields.io/badge/Deployment-NGINX%20%2B%20VPS-success)
![Infrastructure](https://img.shields.io/badge/Focus-Infrastructure-orange)

> **Status:** Production Deployment Complete

**Tech Stack:** Angular • Django • PostgreSQL • Docker • Docker Compose • NGINX • VPS

## Project Focus

- ✅ Backend Development
- ✅ Docker & Containerization
- ✅ Reverse Proxy & NGINX
- ✅ VPS Deployment
- ✅ Production Architecture

---

## Overview

Wish Me Surprise is a full-stack web application for creating and sharing personalized surprise pages through secure public links.

The project evolved from a simple Angular + Django application into a production-style deployment, focusing on:

- Backend architecture
- Relational database design
- Docker containerization
- Reverse proxy configuration
- VPS deployment
- Infrastructure separation

---

## Project Evolution

| Phase | Focus | Outcome |
|--------|-------|---------|
| **Phase 1** | Full-Stack Application | Angular + Django |
| **Phase 2** | Database | PostgreSQL migration |
| **Phase 3** | Containerization | Docker & Compose |
| **Phase 4** | Production | NGINX + VPS + SSL |
| **Phase 5** | Infrastructure | Deployment separation |

---

### Evolution Journey

```text
Phase 1
────────
Angular
    │
Django
    │
SQLite


        │
        ▼


Phase 2
────────
Angular
    │
Django
    │
PostgreSQL


        │
        ▼


Phase 3
────────
Docker Compose
    ├── Angular
    ├── Django
    └── PostgreSQL


        │
        ▼


Phase 4
────────
Internet
    │
NGINX
    │
Docker Compose
    ├── Angular
    ├── Django
    └── PostgreSQL


        │
        ▼


Phase 5
────────
Platform Infrastructure
        │
Deployment Repository
        │
Application Stack
```

---

## Component Responsibilities

### Frontend

- Angular
- Forms
- Preview workflow
- Public pages

### Backend

- Django
- Validation
- Business logic
- Template rendering
- Link generation

### Database

- PostgreSQL
- Templates
- User content
- Expiration metadata

---

# Application Workflow

The platform allows users to:

- Select predefined surprise templates
- Submit personalized content
- Preview generated pages before publishing
- Generate secure public links
- Share dynamic surprise pages with others

Key engineering challenges included:

- Secure public URL generation
- Managing link expiration
- Dynamic server-side rendering
- Structured relational data storage
- Protecting public-facing endpoints from abuse
- Supporting deployment across multiple environments

---

# Request Lifecycle

## Preview Flow

1. User selects a template and submits input.
2. Backend validates incoming data.
3. Django renders dynamic HTML using stored templates.
4. Preview content is returned to the frontend.

## Link Generation Flow

1. Input is validated and persisted.
2. A UUID-based public identifier is generated.
3. Expiration metadata is applied.
4. A shareable public URL is returned.

## Public Access Flow

1. Visitor opens the generated link.
2. Backend retrieves associated content.
3. Dynamic HTML is rendered using the selected template.
4. Final page is served to the browser.

---

# Production Architecture

```text
                 Internet
                     │
                     ▼
              Platform NGINX
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
 Angular Frontend       Django Backend
                                │
                                ▼
                           PostgreSQL
```

### Public Page Flow

```text
Visitor
    │
    ▼
Public URL (UUID)
    │
    ▼
Django Backend
    │
    ▼
Template + Stored Content
    │
    ▼
Rendered HTML Response
```

---

### Design Decisions

- Templates remain immutable after creation.
- User-generated content is stored separately from templates.
- UUIDs are used for public page access.
- Expiration is enforced at the application layer.
- The schema supports adding new template types without major structural changes.

---

## Deployment Highlights

- Dockerized frontend & backend
- PostgreSQL container
- Docker Compose orchestration
- NGINX reverse proxy
- Docker Hub deployment
- SSL configuration
- VPS hosting

---

# Screenshots

Representative screenshots of the application are available in the `screenshots/` directory.

---

# Key Engineering Takeaways

### Backend Engineering

- Django application architecture
- REST workflow implementation
- Relational database modeling
- UUID-based public resource management
- Expiration-aware content access
- Dynamic server-side template rendering
- Input validation and workflow control
- Rate limiting for public endpoints
- Separation of template metadata and generated content

### Infrastructure

- Docker containerization
- Docker Compose orchestration
- Reverse proxy configuration with NGINX
- Multi-service deployment architecture

### Operations

- VPS deployment workflows
- SSL configuration
- Static asset serving
- Health checks and service readiness
- Separation of application and infrastructure concerns

---

# Live Demo

🌐 https://wishmesurprise.akashbharnuke.tech/

---

## Future Improvements

### Application

- User authentication
- Scheduled surprises
- Email notifications

### Backend

- Background job processing

### Platform

- Monitoring & observability
- CI/CD pipeline

