# Wish Me Surprise

**Tech Stack:** Angular, Django, PostgreSQL, Docker, Docker Compose, NGINX, VPS

---

## Overview

Wish Me Surprise is a full-stack web application that allows users to create and share personalized surprise pages through secure public links.

The project began as a traditional Angular + Django application and gradually evolved into a containerized, production-style architecture. Along the way, it became a practical learning ground for backend development, relational data modeling, Docker-based deployments, reverse proxying, and infrastructure separation.

The platform combines a client-side Angular application for user interactions with a Django backend responsible for template rendering, link generation, content persistence, and public page delivery.

---

## Problem Statement

The platform allows users to:

* Select predefined surprise templates
* Submit personalized content
* Preview generated pages before publishing
* Generate secure public links
* Share dynamic surprise pages with others

Key engineering challenges included:

* Secure public URL generation
* Managing link expiration
* Dynamic server-side rendering
* Structured relational data storage
* Protecting public-facing endpoints from abuse
* Supporting deployment across multiple environments

---

## Architecture Overview

The application follows a layered architecture:

### Frontend Layer (Angular)

Responsible for:

* User interactions
* Form submission
* Preview workflows
* Public page navigation

### Backend Layer (Django)

Responsible for:

* Request validation
* Business workflows
* Template rendering
* Public link generation
* Content retrieval

### Persistence Layer (PostgreSQL)

Responsible for:

* Template storage
* Generated page data
* Metadata management
* Expiration tracking

---

## Request Lifecycle

### Preview Flow

1. User selects a template and submits input.
2. Backend validates incoming data.
3. Django renders dynamic HTML using stored templates.
4. Preview content is returned to the frontend.

### Link Generation Flow

1. Input is validated and persisted.
2. A UUID-based public identifier is generated.
3. Expiration metadata is applied.
4. A shareable public URL is returned.

### Public Access Flow

1. Visitor opens the generated link.
2. Backend retrieves associated content.
3. Dynamic HTML is rendered using the selected template.
4. Final page is served to the browser.

---

## System Architecture

```text
Browser
    ↓
Angular Frontend
    ↓
Django Backend
    ↓
PostgreSQL
```

### Public Page Flow

```text
Visitor
    ↓
Public URL (UUID)
    ↓
Django Application
    ↓
Template + Stored Content
    ↓
Rendered HTML Response
```

---

## Database Design

The system uses PostgreSQL with a relational schema designed around:

* Template definitions
* Generated surprise pages
* User-provided content
* Expiration metadata

Design priorities included:

* Clear ownership of generated content
* Separation between templates and user data
* Controlled public exposure
* Extensibility for future template types

---

## Key Backend Concepts

* UUID-based public identifiers
* Expiration-aware content access
* Dynamic server-side template rendering
* Input validation and workflow control
* Relational modeling with PostgreSQL
* Rate limiting for public endpoints
* Separation of template metadata and generated content

---

## Deployment Evolution

The project originally ran as a traditional frontend and backend setup.

As the project grew, it was migrated toward a production-style deployment architecture involving:

* Dockerized Angular frontend
* Dockerized Django backend
* PostgreSQL containerization
* Docker Compose orchestration
* NGINX reverse proxy routing
* Docker Hub image distribution
* VPS-based deployment

This evolution helped establish clear separation between:

```text
Application Code
        ↓
Deployment Orchestration
        ↓
Platform Infrastructure
```

while keeping the application itself independent of deployment concerns.

---

## Engineering Learnings

This project provided hands-on experience with:

* Backend architecture design
* Docker containerization
* Multi-service orchestration
* Reverse proxy routing
* Production configuration management
* Static asset serving in production
* Health checks and service readiness
* VPS deployment workflows
* Infrastructure separation and scalability

---

## Live Application

👉 [ https://wishmesurprise.akashbharnuke.tech/ ] 

---

## Future Improvements

* User authentication and ownership tracking
* Scheduled surprise releases
* Email notifications
* Background job processing
* Monitoring and observability integration
* CI/CD automation
