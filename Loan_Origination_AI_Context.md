# Loan Origination System — AI Context & Architecture Guide

## Purpose

You are working on a **Loan Origination System** built using:

- **Frontend:** React
- **Backend:** Spring Boot
- **Database:** H2
- **ORM / Data Access:** JPA
- **HTTP Client:** Axios

The system has three functional roles:

1. **Customer**
2. **Maker**
3. **Checker**

This document is the **source of truth for the application's architecture and development conventions**.

---

# 1. Core Architecture Principle

Treat this as **ONE Loan Origination System with three roles**, not as three separate applications or three separate backend architectures.

Customer, Maker, and Checker participate in the same loan-application lifecycle.

The frontend may contain role-specific:

- Components
- Pages
- Services
- Routes

However, the **backend architecture remains common and layer-based** for all roles.

Do not create separate backend architectures for Customer, Maker, and Checker.

The fundamental backend flow is:

```text
Controller
    ↓
Request DTO
    ↓
Entity
    ↓
Service
    ↓
Repository
    ↓
H2
```

The response travels back through the application layers:

```text
H2
    ↓
Repository
    ↓
Service
    ↓
Response DTO
    ↓
ResponseEntity
    ↓
React Service
    ↓
React State
    ↓
UI
```

---

# 2. Roles and Responsibilities

## Customer

The Customer creates and follows a loan application.

Typical actions:

- Apply for a loan
- Upload documents
- View comments
- Modify application information
- Resubmit an application
- Track application status

## Maker

The Maker performs the **first operational review**.

Typical actions:

- Verify information
- Request modifications
- Add comments
- Accept the application at the Maker stage
- Reject the application at the Maker stage

## Checker

The Checker performs the **final verification and decision**.

Typical actions:

- Review the Maker's outcome
- Approve the application
- Reject the application
- Complete the final decision

Although each role has different screens and permitted actions, every request must follow the same full-stack architecture.

---

# 3. Project Structure

Use the following conceptual project structure:

```text
loan-origination/
│
├── frontend/
│   ├── public/
│   └── src/
│       ├── components/
│       │   ├── customer/
│       │   ├── maker/
│       │   ├── checker/
│       │   └── common/
│       │
│       ├── pages/
│       │   ├── customer/
│       │   ├── maker/
│       │   └── checker/
│       │
│       ├── services/
│       │   ├── customer/
│       │   ├── maker/
│       │   └── checker/
│       │
│       ├── routes/
│       ├── styles/
│       ├── constants/
│       └── assets/
│
└── backend/
    └── src/
        ├── main/
        │   ├── java/
        │   │   └── com/
        │   │       └── loanorigination/
        │   │           ├── controller/
        │   │           ├── dto/
        │   │           │   ├── request/
        │   │           │   └── response/
        │   │           ├── entity/
        │   │           ├── service/
        │   │           ├── repository/
        │   │           ├── exception/
        │   │           ├── enums/
        │   │           └── config/
        │   │
        │   └── resources/
        │
        └── test/
            └── java/
```

---

# 4. Frontend Responsibilities

## `components/`

Contains reusable UI sections.

Role-specific components are separated into:

```text
components/customer/
components/maker/
components/checker/
```

Shared controls belong in:

```text
components/common/
```

## `pages/`

Contains complete screens connected to routes.

Role-specific pages are organized as:

```text
pages/customer/
pages/maker/
pages/checker/
```

Event-handler functions commonly live in these page components.

## `services/`

Contains frontend service functions that use **Axios** to communicate with Spring Boot API endpoints.

Organization:

```text
services/customer/
services/maker/
services/checker/
```

The frontend service layer contains **HTTP operations**, not backend business logic.

React components and event handlers should call service functions instead of repeatedly writing Axios calls throughout the UI.

## `routes/`

Maps URLs to Customer, Maker, and Checker pages and controls navigation.

## `styles/`

Contains CSS files used by pages and components.

## `constants/`

Contains fixed frontend values such as:

- UI labels
- Status mappings
- Other non-changing frontend values

## `assets/`

Contains:

- Images
- Icons
- Static frontend resources

---

# 5. Backend Responsibilities

The backend is common to all three roles.

## `controller/`

Responsible for:

- Defining API endpoints
- Receiving Request DTOs
- Calling the service layer
- Converting returned entities into Response DTOs
- Returning `ResponseEntity`

Controllers should **not contain business logic**.

Controllers should **not directly communicate with H2**.

## `dto/request/`

Contains Request DTOs representing JSON input received from React.

React should send **Request DTO-shaped JSON**, rather than directly exposing backend entities to browser input.

Request DTOs protect entities from direct browser input.

## `dto/response/`

Contains Response DTOs representing data returned to React.

Never directly return JPA entities to React.

The controller converts entities into Response DTOs and returns those.

Response DTOs should expose only the fields required by the frontend.

## `entity/`

Contains persistent domain objects.

Entities are mapped to H2 database tables using JPA annotations.

## `service/`

Contains business logic, including:

- Business rules
- Validations
- Workflow decisions
- Repository calls

Business validation and workflow decisions belong in the Spring Service layer, **not in controllers**.

Services must not directly execute H2 queries.

## `repository/`

Contains JPA data-access interfaces.

Repositories perform data-access operations and communicate with H2.

The repository is the **only application layer that communicates with the database**.

## `exception/`

Contains application exceptions and common HTTP error handling.

## `enums/`

Contains controlled values such as:

- Role
- Status
- Stage
- Comment status
- Document type

## `config/`

Contains application-wide configuration such as:

- CORS
- Other shared Spring Boot configuration

## `resources/`

Contains application configuration, including:

```text
backend/src/main/resources/application.properties
```

This includes H2 and H2 Console configuration.

---

# 6. Request Architecture — React to H2

Every create, update, view, review, comment, document, and tracking operation must follow this direction:

```text
1. React Component
        ↓
2. React Event Handler
        ↓
3. React Service Function
        ↓
4. Spring Controller
        ↓
5. Request DTO → Entity
        ↓
6. Spring Service
        ↓
7. Repository
        ↓
8. H2 Database
```

## Step 1 — React Component

Displays the screen and captures the user's action.

## Step 2 — React Event Handler

Collects component state and calls the appropriate frontend service function.

## Step 3 — React Service Function

Uses Axios to send an HTTP request containing:

- JSON, or
- Multipart data

## Step 4 — Spring Controller

Matches the endpoint and receives input through a Request DTO.

## Step 5 — Request DTO → Entity

The controller transforms validated request data into an entity object.

## Step 6 — Spring Service

Applies business rules and calls the appropriate repository.

## Step 7 — Repository

Runs JPA data-access operations.

## Step 8 — H2 Database

Stores or retrieves persistent entity data.

---

# 7. Response Architecture — H2 to React

The response must return through the application layers in reverse.

The controller must **not send an entity directly to React**.

```text
1. H2 Database
        ↓
2. Repository
        ↓
3. Spring Service
        ↓
4. Entity → Response DTO
        ↓
5. ResponseEntity
        ↓
6. React Service Function
        ↓
7. React Event Handler
        ↓
8. React Component
```

## Step 1 — H2 Database

Returns stored data to the repository.

## Step 2 — Repository

Returns an entity or collection of entities to the service.

## Step 3 — Spring Service

Returns the processed entity result to the controller.

## Step 4 — Entity → Response DTO

The controller converts the entity result into the required response shape.

## Step 5 — ResponseEntity

The controller returns Response DTO data with the correct HTTP status.

## Step 6 — React Service Function

Axios receives the JSON response and returns the response data.

## Step 7 — React Event Handler

Stores the result in React state and handles errors.

## Step 8 — React Component

Re-renders and displays the updated information.

---

# 8. H2 Database Position

H2 is the **persistence layer**.

The database sits behind the repository.

The architecture must remain:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
H2
```

Controllers and services must never execute H2 queries directly.

### H2-related configuration

```text
backend/src/main/resources/application.properties
```

### H2 Console

H2 Console is enabled through Spring Boot configuration and opened through the configured console path.

### JPA Entities

JPA entities are mapped from the `entity` package to H2 tables.

### Data Access

Data access is performed through repository interfaces.

### Development Behavior

In-memory H2 data is cleared when the application stops unless file-based H2 is configured.

---

# 9. Mandatory Architecture Rules

These rules must be followed whenever implementing, modifying, reviewing, debugging, or explaining code.

## Rule 1 — Do not skip the React service layer

Components and handlers should call a frontend service function instead of writing repeated Axios calls everywhere.

Correct:

```text
React Component
      ↓
Event Handler
      ↓
React Service
      ↓
Axios
```

Do not scatter Axios calls throughout React components.

---

## Rule 2 — Do not send entities from React

React sends **Request DTO-shaped JSON**.

Do not design the frontend around directly sending backend JPA entities.

---

## Rule 3 — Controllers receive Request DTOs

Controller request bodies must use Request DTOs.

Correct:

```text
React JSON
    ↓
Request DTO
    ↓
Entity
```

Do not use JPA entities directly as controller request bodies.

---

## Rule 4 — Do not put business logic in controllers

Controllers should coordinate the request/response flow.

The following belong in the Spring Service layer:

- Business validation
- Business rules
- Workflow decisions

---

## Rule 5 — Services must not directly access H2

Services call repositories.

Repositories communicate with H2.

Correct:

```text
Service
    ↓
Repository
    ↓
H2
```

Incorrect:

```text
Service
    ↓
H2
```

---

## Rule 6 — Do not return entities to React

The controller must convert entities into Response DTOs.

Correct:

```text
Entity
    ↓
Response DTO
    ↓
ResponseEntity
    ↓
React
```

---

## Rule 7 — Return HTTP status with data

The controller should return `ResponseEntity` containing a Response DTO or collection, together with the appropriate HTTP status.

---

## Rule 8 — Preserve one common backend

Customer, Maker, and Checker features must use the same backend layers and conventions.

Do not create separate backend architectures such as:

```text
customer-controller/
maker-controller/
checker-controller/
```

Role-specific behavior should be implemented through the common architecture, business rules, and security/access checks.

---

# 10. Team Development Workflow

Before implementing a feature:

1. Confirm the common folder structure.
2. Put role-specific React code under the appropriate Customer, Maker, or Checker frontend folder.
3. Ensure every frontend event handler calls a frontend service function.
4. Ensure the service function sends the Axios request.
5. Define the Request DTO and Response DTO contract for every endpoint **before implementation**.
6. Implement the backend in the approved order:
   - Controller transformation
   - Service business logic
   - Repository operation
   - H2 persistence
7. Test the backend endpoint first.
8. Connect the corresponding React service function.
9. Verify:
   - Full response flow
   - HTTP status
   - React state update
   - UI rendering
10. Review new code against the architecture rules before merging branches.

---

# 11. Instructions for Any AI Working on This Project

When asked to implement, modify, review, debug, explain, or generate code for this Loan Origination System:

### First determine where the requested feature belongs.

Identify the relevant layers before writing code.

For a typical feature, consider:

- React component/page
- React event handler
- Role-specific frontend service
- Spring Controller
- Request DTO
- Entity, if required
- Spring Service
- Repository
- Response DTO
- ResponseEntity
- H2 persistence

### Preserve the architecture.

Do not invent a different architecture unless explicitly asked.

Do not bypass layers for convenience.

Do not put business logic in React when it belongs to the backend.

Do not put business logic in controllers.

Do not put database operations in controllers or services.

Do not return JPA entities directly to the frontend.

Do not create duplicate backend architectures for Customer, Maker, and Checker.

### When generating code

Before providing implementation code, mentally verify:

```text
Does the frontend use a service layer?
Does the frontend send Request DTO-shaped data?
Does the Controller receive a Request DTO?
Is business logic inside Service?
Does Service communicate through Repository?
Does Repository communicate with H2?
Is the Entity kept inside the backend domain/data layer?
Is the Entity converted into a Response DTO?
Does the Controller return ResponseEntity?
Does the response flow correctly back to React?
Is role-specific behavior handled without creating a separate backend architecture?
```

If any answer is "no", revise the implementation before presenting it.

---

# 12. Golden Architecture Reference

The complete system flow is:

```text
                         REQUEST
                            ↓
                   React Component
                            ↓
                  React Event Handler
                            ↓
                  React Service / Axios
                            ↓
                   Spring Controller
                            ↓
                      Request DTO
                            ↓
                          Entity
                            ↓
                     Spring Service
                            ↓
                       Repository
                            ↓
                       H2 Database
                            ↓
                         RESPONSE
                            ↓
                       Repository
                            ↓
                     Spring Service
                            ↓
                     Response DTO
                            ↓
                     ResponseEntity
                            ↓
                  React Service / Axios
                            ↓
                  React Event Handler
                            ↓
                    React Component
                            ↓
                            UI
```

The fundamental architecture is:

```text
React Component
→ Event Handler
→ React Service
→ Controller
→ Request DTO
→ Entity
→ Spring Service
→ Repository
→ H2
→ Response DTO
→ ResponseEntity
→ React State
→ UI
```

---

# 13. Final Principle

## One System, Three Roles, One Architecture

Customer, Maker, and Checker have different:

- Screens
- Responsibilities
- Permissions
- Allowed actions

But their technical implementation must remain consistent with the same full-stack architecture.

Always preserve the separation of responsibilities:

```text
React
  → UI and user interaction

React Service
  → HTTP communication

Controller
  → API boundary and request/response coordination

Request / Response DTOs
  → API data contracts

Entity
  → Persistent domain model

Service
  → Business rules, validation, and workflow

Repository
  → Database access

H2
  → Persistence
```

**This architecture is the source of truth.**
