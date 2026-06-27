# Dockyard

A terminal-native Docker management application built with Python and Textual.

Dockyard provides an interactive terminal interface for managing Docker containers without leaving the command line. The project was built to explore terminal UI development, Docker Engine integration, and developer tooling.

![Developer Tool](https://img.shields.io/badge/Project-Developer%20Tool-purple)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Docker SDK](https://img.shields.io/badge/Docker-SDK-2496ED?logo=docker&logoColor=white)
![Terminal UI](https://img.shields.io/badge/UI-Textual-blueviolet)

> **Status:** Production Ready Demo

<!-- ![License](https://img.shields.io/badge/License-MIT-green) -->

<!-- > **Status:** MVP Complete • Deployed -->
---

## Motivation

While Docker CLI is powerful, common container management tasks often require switching between multiple commands or graphical tools.

Dockyard explores how a rich terminal interface can provide an efficient middle ground by combining the speed of the CLI with the discoverability of a GUI.

---

## Why Dockyard?

- Most of my previous projects focused on backend services, APIs, and deployment infrastructure. 

- With Dockyard, I wanted to explore a different side of software engineering by building a tool for developers.

- The project challenged me to think beyond business logic and CRUD operations, focusing instead on user experience within the terminal, event-driven interfaces, and integration with the Docker Engine. 

- It also provided hands-on experience designing reusable UI components, service abstractions, and packaging a desktop-style application for distribution.

---

## Tech Stack

- Python
- Textual
- Docker SDK
- PyInstaller

---

## Features

- Discover running containers
- View container metadata
- Start / Stop / Restart containers
- Delete containers
- Live refresh
- Responsive terminal UI
- Keyboard navigation

---

## Architecture

```text
                User
                  │
                  ▼
         Textual Terminal UI
                  │
        ┌─────────┴─────────┐
        │                   │
 Dashboard Screen     Details Panel
        │                   │
        └─────────┬─────────┘
                  │
           Docker Service
                  │
             Docker SDK
                  │
           Docker Engine
```

---

## Engineering Challenges

Some of the key engineering challenges while building Dockyard included:

- Designing an event-driven terminal UI with reactive state updates.
- Integrating with the Docker SDK while handling container lifecycle operations safely.
- Structuring the application into reusable screens, widgets, and services.
- Packaging a Textual application as a standalone executable using PyInstaller.
- Maintaining a responsive interface while periodically refreshing Docker state.

---

## Project Structure

```text
dockyard/
├── app/
│   ├── screens/
│   ├── widgets/
│   ├── services/
│   ├── models/
│   └── utils/
├── assets/
├── docs/
└── main.py
```
## Key Concepts Explored

- Terminal User Interfaces (TUI)
- Event-driven application architecture
- Docker Engine API integration
- Service layer abstraction
- Reactive UI state management
- Application packaging and distribution

## Screenshots

Representative screenshots of the application can be found in the [`screenshots/`](./screenshots) directory.

## Future Improvements

### Docker

- Compose support
- Image management
- Volume management
- Network visualization

### Monitoring

- Container metrics
- Resource usage
- Health monitoring

### Developer Experience

- Container logs
- Remote Docker hosts
- Keyboard shortcuts
- Theme customization

## What I Learned

Building Dockyard helped me gain practical experience with:

- Building interactive terminal applications using Textual
- Working with the Docker Engine through the Docker SDK
- Designing event-driven interfaces
- Managing application state in a TUI
- Packaging Python desktop applications with PyInstaller

