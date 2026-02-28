# Wish Me Surprise

**Tech Stack:** Django, PostgreSQL, Angular, Render  

## Overview

Wish Me Surprise is a dynamic web application that allows users to generate personalized, shareable surprise pages.

The backend is built using Django and PostgreSQL, focusing on structured data modeling, controlled public link generation, and dynamic server-side rendering.

A lightweight Angular frontend interacts with backend endpoints to handle user input and preview flows.


---

## Problem Statement

## Problem Statement

The system allows users to:

- Select predefined templates  
- Submit structured input  
- Preview rendered output  
- Generate secure public links  
- Share final dynamic pages  

Key challenges included secure link generation, expiration handling, relational modeling, and abuse protection.

---

## Architecture Overview

The system follows a layered backend structure:

- **Routing Layer** – Handles incoming HTTP requests
- **Business Logic Layer** – (preview & generation workflows)
- **Persistence Layer** – Stores template metadata and user-generated content in PostgreSQL

The Angular frontend interacts with Django endpoints for user flows and preview handling.


---

## Request Lifecycle

### Preview Flow

1. User selects template and submits input.
2. Backend validates input.
3. Data is rendered into dynamic HTML using Django templates.
4. Rendered preview is returned to frontend.

### Public Link Generation

1. Validated input is persisted in database.
2. A unique public identifier (UUID-based slug) is generated.
3. Expiration logic is applied.
4. A shareable URL is returned.

### Public Access Flow

1. Public URL is accessed.
2. Backend fetches associated template and stored user data.
3. Final HTML page is dynamically rendered.
4. Response is served.

---

UUID-based slugs to prevent predictable link enumeration.

Expiration logic to limit long-term exposure.

Rate limiting to protect public endpoints.

Clear separation between template definitions and generated content.

---

## System Architecture Diagram

    ┌────────────────────┐
    │     Angular UI     │
    └─────────┬──────────┘
              │ HTTP Requests
              ▼
    ┌────────────────────┐
    │     Django App     │
    │  (Routing + Logic) │
    └─────────┬──────────┘
              │ ORM
              ▼
    ┌────────────────────┐
    │   PostgreSQL DB    │
    └────────────────────┘

## Public Access Flow:

User → Public URL (UUID)  
↓  
Django retrieves stored data + template   
↓  
Dynamic HTML rendered

## Database Design

The system uses relational modeling in PostgreSQL:

- Template metadata storage
- User-generated structured input
- Foreign key relationships between template and generated content
- Expiration timestamp for automatic link validity control

Design focus:
- Controlled public exposure
- Separation between template definition and user data


---

## Key Backend Concepts

- UUID-based slugs to prevent predictable link enumeration.
- Expiration logic to limit long-term exposure.
- Rate limiting to protect public endpoints.
- Clear separation between template definitions and generated content.  

---

## Deployment

- Hosted on Render  
- Environment-based configuration  
- PostgreSQL integration  

Live Application:
👉 https://wish-me-surprise.web.app/

---

## Future Improvements

- User authentication & ownership of generated links
- Enhanced template customization
- Background tasks for scheduled surprise releases
- Monitoring integration