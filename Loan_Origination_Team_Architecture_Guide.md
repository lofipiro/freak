# Loan Origination Team Architecture Guide

# Common React + Spring Boot + H2 structure for Customer, Maker, and Checker modules

| Architecture rule The folder structure and layer flow are common to
  the complete Loan Origination system. Customer, Maker, and Checker are
  team roles and functional modules; they do not create three separate
  backend architectures. \|

| --- \|

# 1. Purpose of this guide

This guide defines the shared project structure and the full
request-response flow that every teammate must follow. It is
intentionally role-neutral at the backend layer, so all modules use the
same controller, DTO, entity, service, repository, and database
architecture.

# 2. Team roles and responsibilities

| Role \| Main responsibility \| Typical actions \|

| --- \| --- \| --- \|

| Customer \| Creates and follows a loan application. \| Apply, upload
  documents, view comments, modify, resubmit, and track status. \|

| Maker \| Performs the first operational review. \| Verify information,
  request modifications, add comments, accept, or reject at Maker stage.
  \|

| Checker \| Performs final verification and decision. \| Review Maker
  outcome, approve or reject, and complete the final decision. \|

Although the screens and permitted actions differ by role, all requests
must pass through the same full-stack layers.

# 3. Common team project structure

The frontend may group role-specific UI code under Customer, Maker, and
Checker. The backend remains layer-based and common to all roles.

loan-origination/ ├── frontend/ │ ├── public/ │ └── src/ │ ├──
components/ │ │ ├── customer/ │ │ ├── maker/ │ │ ├── checker/ │ │ └──
common/ │ ├── pages/ │ │ ├── customer/ │ │ ├── maker/ │ │ └── checker/ │
├── services/ │ │ ├── customer/ │ │ ├── maker/ │ │ └── checker/ │ ├──
routes/ │ ├── styles/ │ ├── constants/ │ └── assets/ │ └── backend/ └──
src/ ├── main/ │ ├── java/ │ │ └── com/ │ │ └── loanorigination/ │ │ ├──
controller/ │ │ ├── dto/ │ │ │ ├── request/ │ │ │ └── response/ │ │ ├──
entity/ │ │ ├── service/ │ │ ├── repository/ │ │ ├── exception/ │ │ ├──
enums/ │ │ └── config/ │ └── resources/ └── test/ └── java/

# 4. Why the structure is common

-   The Customer, Maker, and Checker modules work on the same
    loan-application lifecycle.

-   A common backend prevents duplicate controller, service, repository,
    and database designs.

-   Role-specific access rules belong in business logic and security
    checks, not in a separate architecture.

-   Shared DTO and entity conventions keep API requests and responses
    consistent across the team.

-   All teammates can develop independently while preserving one
    predictable integration flow.

# 5. Frontend folder responsibilities

| Folder \| Responsibility \|

| --- \| --- \|

| components/ \| Reusable UI sections. Role-specific components are
  separated into customer, maker, and checker; shared controls belong in
  common. \|

| pages/ \| Complete screens connected to routes. Event-handler
  functions commonly live in these page components. \|

| services/ \| Frontend service functions that use Axios to call Spring
  Boot API endpoints. They contain HTTP operations, not backend business
  logic. \|

| routes/ \| Maps URLs to Customer, Maker, and Checker pages and
  controls navigation. \|

| styles/ \| CSS files used by pages and components. \|

| constants/ \| Fixed UI labels, status mappings, and other non-changing
  frontend values. \|

| assets/ \| Images, icons, and static frontend resources. \|

# 6. Backend package responsibilities

| Package \| Responsibility \|

| --- \| --- \|

| controller/ \| Defines API endpoints, receives Request DTOs, calls the
  service, converts returned entities to Response DTOs, and returns
  ResponseEntity. \|

| dto/request/ \| Defines the JSON input accepted from React. Request
  DTOs protect the entity from direct browser input. \|

| dto/response/ \| Defines the JSON output returned to React. Response
  DTOs expose only the required fields. \|

| entity/ \| Represents persistent domain data mapped to H2 database
  tables using JPA annotations. \|

| service/ \| Contains business rules, validations, workflow decisions,
  and repository calls. \|

| repository/ \| Contains JPA data-access interfaces that read and write
  entities in H2. \|

| exception/ \| Contains application exceptions and common HTTP error
  handling. \|

| enums/ \| Contains controlled values such as role, status, stage,
  comment status, and document type. \|

| config/ \| Contains application configuration such as CORS and other
  shared settings. \|

| resources/ \| Contains application properties, including H2 and H2
  Console configuration. \|

# 7. Request architecture: React to H2

Every create, update, view, review, comment, document, and tracking
operation must use this direction.

| 1 \| React Component Displays the screen and captures the user action.
  \|

| --- \| --- \|

↓

| 2 \| React Event Handler Collects component state and calls the
  appropriate frontend service function. \|

| --- \| --- \|

↓

| 3 \| React Service Function Uses Axios to send an HTTP request
  containing JSON or multipart data. \|

| --- \| --- \|

↓

| 4 \| Spring Controller Matches the endpoint and receives input through
  a Request DTO. \|

| --- \| --- \|

↓

| 5 \| Request DTO to Entity The controller transforms validated request
  data into an entity object. \|

| --- \| --- \|

↓

| 6 \| Spring Service Applies business rules and calls the appropriate
  repository. \|

| --- \| --- \|

↓

| 7 \| Repository Runs JPA data-access operations. \|

| --- \| --- \|

↓

| 8 \| H2 Database Stores or retrieves the persistent entity data. \|

| --- \| --- \|

Response begins here

# 8. Response architecture: H2 to React

The response must return through the same layers in reverse. The
controller must not send the entity directly to React.

| 1 \| H2 Database Returns stored data to the repository. \|

| --- \| --- \|

↓

| 2 \| Repository Returns an entity or a collection of entities to the
  service. \|

| --- \| --- \|

↓

| 3 \| Spring Service Returns the processed entity result to the
  controller. \|

| --- \| --- \|

↓

| 4 \| Entity to Response DTO The controller converts the entity result
  into the required response shape. \|

| --- \| --- \|

↓

| 5 \| ResponseEntity The controller returns Response DTO data with the
  correct HTTP status. \|

| --- \| --- \|

↓

| 6 \| React Service Function Axios receives the JSON response and
  returns the response data. \|

| --- \| --- \|

↓

| 7 \| React Event Handler Stores the result in React state and handles
  errors. \|

| --- \| --- \|

↓

| 8 \| React Component Re-renders and displays the updated information.
  \|

| --- \| --- \|

# 9. H2 database position

| H2 is the persistence layer H2 sits behind the repository. Controllers
  and services must never execute H2 queries directly. The repository is
  the only application layer that communicates with the database. \|

| --- \|

| Item \| Location or rule \|

| --- \| --- \|

| Configuration \| backend/src/main/resources/application.properties \|

| H2 Console \| Enabled through Spring Boot configuration and opened
  through the configured console path. \|

| JPA entities \| Mapped from the entity package to H2 tables. \|

| Data access \| Performed through repository interfaces. \|

| Development use \| In-memory H2 data is cleared when the application
  stops unless file-based H2 is configured. \|

# 10. Architecture rules for all teammates

| Rule \| Required practice \|

| --- \| --- \|

| Do not skip the React service layer \| Components and handlers should
  call a frontend service function instead of writing repeated Axios
  calls everywhere. \|

| Do not send entities from React \| React sends Request DTO-shaped
  JSON. \|

| Do not receive entities in controllers \| Controller request bodies
  use Request DTOs. \|

| Do not place business logic in controllers \| Business validation and
  workflow decisions belong in the Spring service layer. \|

| Do not access H2 from services directly \| Services call repositories;
  repositories communicate with H2. \|

| Do not return entities to React \| Controller converts entities to
  Response DTOs. \|

| Return the HTTP status with data \| Controller returns ResponseEntity
  containing a Response DTO or collection. \|

| Preserve one common backend \| Customer, Maker, and Checker features
  use the same backend layers and conventions. \|

# 11. Team implementation guide

-   Confirm the common folder structure before creating feature files.

-   Each teammate places role-specific React code under the appropriate
    Customer, Maker, or Checker frontend folder.

-   Each frontend event handler calls a service function; the service
    function sends the Axios request.

-   For every endpoint, define its Request DTO and Response DTO contract
    before implementation.

-   Implement the controller transformation, service business logic,
    repository operation, and H2 persistence in the approved order.

-   Test the backend endpoint first, then connect the corresponding
    React service function.

-   Verify the full response flow, HTTP status, state update, and UI
    rendering.

-   Review new code against the architecture rules before merging
    branches.

# 12. Final reference

| One system, three roles, one architecture Customer, Maker, and Checker
  have different screens and permissions, but the technical flow remains
  consistent: React Component → Event Handler → React Service →
  Controller → Request DTO → Entity → Spring Service → Repository → H2,
  followed by Entity → Response DTO → ResponseEntity → React State → UI.
  \|

| --- \|
