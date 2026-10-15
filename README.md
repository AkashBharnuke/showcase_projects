# Backend & Platform Engineering Showcase

Hi, I'm **Akash Bharnuke** 👋

I'm a **Backend Software Engineer with 3+ years of experience** building enterprise applications and modern backend systems using Python, FastAPI, Django, PostgreSQL, and Docker.

This repository showcases projects across **backend engineering, AI-powered systems, observability, infrastructure, and developer tooling** — with a focus on designing reliable services, containerized applications, and production-oriented architectures.

The projects reflect my interest in building systems that go beyond simply implementing features, with emphasis on **architecture, performance, observability, deployment, and maintainability**.

---

# 🚀 Featured Projects

Ordered by engineering depth — the projects that demonstrate the most system design, reliability work, and real-world verification come first.

## 1️⃣ WatchTower

**Tech Stack**

`Python` • `FastAPI` • `PostgreSQL` • `SQLAlchemy` • `Prometheus` • `Loki` • `Grafana` • `Grafana Alloy` • `Docker Compose`

An operational observability platform that answers "what's running, what's healthy, what failed" across a personal VPS running several other projects at once.

It builds on proven tools (Prometheus, Loki, Grafana) instead of reinventing them, and adds custom logic only where it's actually needed: a service registry, a health-state rollup engine, and a bridge that feeds that health data into Grafana's alerting.

### Highlights

- Service registry with scheduled health checks and status rollup across projects
- Health state exported as Prometheus metrics, so Grafana Alerting can act on it directly
- Infrastructure telemetry (CPU, memory, disk, container stats) via Node Exporter and cAdvisor
- 9 provisioned Grafana alert rules, verified against a real stop/start failure cycle
- A real alerting bug ("no data" read as "healthy") was caught through live testing, not just code review, and fixed
- Spec-driven development with documented design decisions

**Status:** Core platform (phases 0–4) complete and tested. Alerting is implemented and verified; not yet deployed to production infrastructure.

**Links**

- 📄 [Project Overview](./watchtower/README.md)

---

## 2️⃣ AI Intelligence Platform

**Tech Stack**

`Python` • `FastAPI` • `PostgreSQL` • `Qdrant` • `Docker` • `OpenTelemetry` • `Prometheus` • `Grafana` • `Loki` • `Tempo` • `Langfuse`

A middleware gateway that sits between AI applications and LLM providers, built to answer a specific question: how do you make an AI system observable, not just functional?

Every request passing through the gateway is tracked end-to-end — from the initial API call, through document retrieval, to the LLM response — so metrics, logs, and traces can all be tied back to the same request.

### Highlights

- Centralized FastAPI gateway for AI and retrieval-augmented generation (RAG) requests
- RAG pipeline with vector search using Qdrant
- Structured logging and request correlation across the whole pipeline
- Prometheus metrics with Grafana dashboards
- Centralized logs via Loki, distributed tracing via OpenTelemetry and Tempo
- LLM-level observability (token usage, latency) via Langfuse
- Fully containerized development and observability stack

**Links**

- 📄 [Project Overview](./ai-intelligence-platform/README.md)

---

## 3️⃣ Platform VPS Infra

**Tech Stack**

`Bash` • `Nginx` • `Docker Compose` • `Certbot`

The deployment system that runs five live projects on a single VPS, driven entirely by one plain-text config file per project instead of hand-edited server config.

It was built as a direct response to two real incidents in an earlier, ad hoc version — a config overwrite and a routing collision — and is designed so neither can happen again by construction, not just by convention.

### Highlights

- Deploy scripts scoped to one project's files at a time — no wildcard operations that could touch another project by accident
- Automatic rollback if a config change would break the server, verified by a test script
- Duplicate site names are rejected before any file is written
- Certificates issued through a method that keeps all routing config in Git, with no out-of-band changes
- Fully in production: five projects deployed and live

**Links**

- 📄 [Project Overview](./platform_vps_infra/README.md)

---

## 4️⃣ PrivatePlatform

**Tech Stack**

`Bash` • `WireGuard` • `Nginx` • `UFW` • `Docker` • `systemd`

A private network layer for a personal VPS: a VPN plus a second, fully isolated web server, so self-hosted tools don't end up exposed on the public internet by default.

Built as the reusable foundation that WatchTower (above) runs behind, with a scripted process for onboarding the next private application the same way.

### Highlights

- One-command setup pipeline, safe to re-run without duplicating state
- Onboarding script that automatically rejects any app config that would expose it to the public internet — built after that exact mistake happened once
- Private web server kept fully separate from the public-facing one, so a mistake in one can't affect the other
- Two real applications onboarded so far (WatchTower and a Grafana instance)
- Nine written specs tracking what's decided, proposed, or still open

**Status:** In active development, not production-complete. Core networking is live; encryption (TLS) and a few isolation details are still open decisions, tracked explicitly rather than skipped.

**Links**

- 📄 [Project Overview](./private-platform/README.md)

---

## 5️⃣ Wish Me Surprise

**Tech Stack**

`Django` • `Angular` • `PostgreSQL` • `Docker` • `Docker Compose` • `NGINX` • `VPS` • `SSL`

A full-stack web app for creating and sharing personalized surprise pages through secure links — the project that evolved step by step from a simple prototype into a production deployment.

### Highlights

* Django REST backend with PostgreSQL
* Angular frontend
* Multi-container architecture using Docker Compose
* NGINX reverse proxy and service routing
* PostgreSQL health checks and container orchestration
* Live on a VPS with its own domain and SSL certificate
* Application code kept separate from infrastructure/deployment concerns

**Links**

* 📄 [Project Overview](./wish-me-surprise/README.md)
* 🌐 [Live Demo](https://wishmesurprise.akashbharnuke.tech/)

---

## 6️⃣ Retro Game Vault

**Tech Stack**

`TypeScript` • `React` • `Vite` • `Tailwind CSS` • `IndexedDB` • `Docker` • `NGINX`

A browser-based launcher for classic J2ME (feature-phone era) games. Bring your own game file and play it instantly — nothing installs, nothing uploads, nothing leaves your device.

The interesting engineering problem here wasn't the UI — it was wiring a real, unmodified third-party game emulator into a web app it was never designed to be embedded in.

### Highlights

- Instant in-browser play for any compatible game file, fully client-side
- Game library stored on-device (IndexedDB) — no backend, nothing uploaded
- Integrated a third-party emulator through a single messaging contract instead of forking its source
- Found and fixed a pointer-rotation bug that only showed up under real mouse input, not in automated tests
- End-to-end test coverage (Playwright) for the full play-through flow

**Links**

- 📄 [Project Overview](./retro-game-vault/README.md)
- 🌐 [Live Demo](https://retrogamevault.akashbharnuke.tech)

---

## 7️⃣ Dockyard

**Tech Stack**

`Python` • `Textual` • `Docker SDK`

A terminal-based tool for managing Docker containers — start, stop, inspect, and monitor containers without leaving the command line or opening a separate GUI.

### Highlights

- Interactive terminal UI built with Textual
- Live container list, logs, and resource metrics via the Docker Engine API
- Start / stop / restart / delete containers from the interface
- Packaged as a standalone executable for easy distribution

**Links**

- 📄 [Project Overview](./dockyard/README.md)
- 🌐 [Live Demo](https://dockyard.akashbharnuke.tech/)

---

## 8️⃣ MentorAI

**Tech Stack**

`Node.js` • `Express` • `OpenAI API` • `JavaScript` • `Tailwind CSS` • `Docker` • `NGINX`

An AI chatbot that simulates conversations with different mentor personas, each with its own teaching style, built directly on the OpenAI API without an orchestration framework.

### Highlights

- Two distinct AI personas with separately engineered prompts
- Multi-turn conversation context carried across messages
- Backend built in plain Express, with no LangChain or similar framework
- Deployed behind NGINX on a VPS

### Architecture

```text
Browser
    │
NGINX
    │
Express Backend
    │
OpenAI API
```

**Links**

- 📄 [Project Overview](./mentor-ai/README.md)
- 🌐 [Live Demo](https://mentorai.akashbharnuke.tech)

---

## 9️⃣ KnowWhere

**Tech Stack**

`TypeScript` • `React` • `Vite` • `Leaflet` • `Python` • `FastAPI` • `Docker` • `NGINX`

A location-aware web app that finds nearby places — cafes, restaurants, hospitals, fuel stations — on an interactive map, using free OpenStreetMap data instead of a paid maps API.

The backend normalizes raw map data into distances and addresses, behind a small in-memory cache and rate limiter. This project is still evolving: it works end-to-end locally, but has no automated tests and no deployment beyond local Docker Compose yet.

### Highlights

- Browser geolocation with a configurable search radius and category filter
- Map and list views kept in sync by selection state
- Backend distance/address calculation from raw OpenStreetMap data, with an in-memory TTL cache and per-IP rate limiting
- Three-container Docker Compose stack (frontend, backend, Nginx reverse proxy) with a multi-stage frontend build

**Status:** Functional full-stack project, Docker Compose only. No automated tests, no CI, and no hosted deployment yet — in active development, not production-complete.

**Links**

- 📄 [Project Overview](./knowwhere/README.md)

---

# 🧠 Engineering Principles

Across these projects, I focus on building systems that are maintainable, observable, and production-oriented.

Common engineering practices include:

- Clear separation of application layers
- Well-designed relational database schemas
- Service-oriented backend architecture
- Containerized development environments
- Infrastructure and deployment automation
- Metrics, logging, and distributed tracing
- Production-style debugging workflows
- Designing for scalability and maintainability

---

# 🚧 Ongoing Exploration

I'm continuously exploring topics such as:

- Distributed systems
- Background job processing
- Event-driven architecture
- Infrastructure automation
- Developer tooling
- Performance engineering
- AI-assisted software development

---

This repository focuses on architectural thinking, engineering practices, and production-oriented system design rather than exposing complete production source code.

---

# 📬 Contact

- **LinkedIn:** https://linkedin.com/in/akash-bharnuke-b1288b219
- **Email:** akash.bharnuke@gmail.com