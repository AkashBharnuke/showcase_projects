# FastAPI Containerized Backend with Observability

![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?logo=fastapi&logoColor=white)
![Monitoring](https://img.shields.io/badge/Monitoring-Grafana-F46800?logo=grafana&logoColor=white)
![Logging](https://img.shields.io/badge/Logging-Loki-0A84FF)
![Tracing](https://img.shields.io/badge/Tracing-Tempo-7B42BC)

> **Status:** Observability Stack Complete

**Tech Stack:** FastAPI, PostgreSQL, SQLAlchemy, Docker Compose, Nginx, Prometheus, Grafana, Loki, Tempo  

---

## Overview

This project demonstrates a containerized backend architecture built using FastAPI and PostgreSQL, orchestrated via Docker Compose and integrated with a full observability stack.

The focus was on understanding service orchestration, networking isolation, and production-style monitoring rather than just CRUD API implementation.

---

## Architecture Overview

The application follows a layered design:

- **API Layer** – Async FastAPI routes handling HTTP requests
- **Service Layer** – Business logic abstraction
- **Persistence Layer** – SQLAlchemy ORM managing database interactions

Pydantic models handle request validation and response schemas, ensuring structured data flow.

---

## Docker Architecture Diagram

            ┌──────────────┐
            │    Nginx     │
            └──────┬───────┘
                   │
                   ▼
            ┌──────────────┐
            │   FastAPI    │
            └──────┬───────┘
                   │ ORM
                   ▼
            ┌──────────────┐
            │ PostgreSQL   │
            └──────────────┘

## Monitoring Stack:

            ┌──────────────┐
            │ Prometheus │ ← metrics scraping
            └──────────────┘

            ┌──────────────┐
            │ Loki │ ← centralized logs
            └──────────────┘

            ┌──────────────┐
            │ Tempo │ ← request tracing
            └──────────────┘

            ┌──────────────┐
            │ Grafana │ ← visualization
            └──────────────┘            

---

Monitoring stack runs as parallel services within Docker Compose.

---

## Observability Design

### Metrics
- Prometheus configured with manual scraping configuration
- Service metrics exposed for request-level monitoring
- Dashboards built in Grafana

### Logging
- Docker container logs collected via Promtail
- Centralized storage in Loki
- Query-based log inspection in Grafana

### Tracing
- Tempo integrated for distributed tracing
- Enables request lifecycle visibility across services

---

## Networking Isolation

Docker networks segmented into:

- **public** – External access (Nginx)
- **app** – Backend service communication
- **db** – Internal database access (restricted)
- **monitoring** – Observability services

This prevents unnecessary cross-service exposure and mirrors production-style isolation.

---

## Key Engineering Concepts

- Asynchronous API development with FastAPI  
- Structured validation using Pydantic  
- Sync SQLAlchemy ORM with PostgreSQL  
- Multi-service orchestration via Docker Compose  
- Health checks and dependency management  
- Metrics, logging, and tracing integration  

---

## Why This Project Matters

This project demonstrates backend systems thinking — integrating application logic, container orchestration, and observability into a cohesive, debuggable architecture.