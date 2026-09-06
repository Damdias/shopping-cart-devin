# Engineering Guidelines for AI Agents

## 1. Project Context

This repository contains the backend for a Shopping Cart application.

Before making changes, read:

* `README.md` (if present)
* `docs/product.md`
* `docs/architecture.md`

Do not make architectural decisions that conflict with these documents without explicitly explaining the reason.

---

## 2. Technology

Use:

* .NET 10
* ASP.NET Core
* C#
* Entity Framework Core
* PostgreSQL
* xUnit

Do not introduce additional frameworks or infrastructure without justification.

---

## 3. Architecture

The solution is a **modular monolith** with Products and Carts as logical modules. Application behavior is organized as **vertical slices** around individual use cases.

Projects:

```text
ShoppingCart.Api
ShoppingCart.Application
ShoppingCart.Domain
ShoppingCart.Infrastructure
```

Dependency direction (consumer → dependency):

```text
Api → Application
Api → Infrastructure
Application → Domain
Infrastructure → Application
Infrastructure → Domain
```

Rules:

* `Domain` must not reference `Application`, `Infrastructure`, or `Api`.
* `Domain` must not reference ASP.NET Core.
* `Domain` must not reference Entity Framework Core.
* API endpoints must not contain business rules.
* Infrastructure concerns must remain outside Domain.
* Application coordinates use cases.
* Domain contains business rules and invariants.
* The Cart aggregate is the main rich domain aggregate and owns cart invariants.

---

## 4. Scope Control

Only implement what is requested in the current task.

Do not automatically add:

* Authentication
* Authorization
* Redis
* Kafka
* Message brokers
* Microservices
* MediatR
* AutoMapper
* CQRS frameworks
* Event sourcing
* Distributed caching

unless the task explicitly requires them.

Do not implement future features preemptively.

Prefer the simplest implementation that satisfies the current requirements.

---

## 5. Coding Guidelines

Use idiomatic modern C#.

Prefer:

* async APIs for I/O operations
* dependency injection
* immutable request/response models where practical
* nullable reference types
* clear domain-oriented names
* small focused methods
* explicit business rules

Avoid:

* unnecessary abstractions
* generic repository abstractions (use purpose-specific persistence abstractions instead)
* service classes that only forward calls
* static global state
* hidden side effects
* duplicated business logic
* premature optimization

---

## 6. CancellationToken

Async application and infrastructure operations should accept and propagate `CancellationToken` where appropriate.

Example flow:

```text
HTTP request
    ↓
Endpoint
    ↓
Application use case
    ↓
Repository / DbContext
```

The token should be propagated through the call chain.

---

## 7. API Guidelines

Use REST-style APIs with URL-based versioning starting at `v1`.

Use appropriate HTTP status codes.

Use ASP.NET Core `ProblemDetails` for errors.

Do not expose domain or application entities directly from API endpoints.

Use explicit request and response contracts.

Generate and expose an OpenAPI document from the API implementation.

Cart updates use optimistic concurrency; conflicts return `409 Conflict` with `ProblemDetails`.

Examples:

```text
GET    /api/v1/products
GET    /api/v1/products/{productId}

POST   /api/v1/carts
GET    /api/v1/carts/{cartId}

POST   /api/v1/carts/{cartId}/items
PUT    /api/v1/carts/{cartId}/items/{productId}
DELETE /api/v1/carts/{cartId}/items/{productId}
DELETE /api/v1/carts/{cartId}/items
```

---

## 8. Database Guidelines

Use PostgreSQL.

Use Entity Framework Core migrations for schema changes.

Do not use `EnsureCreated()` for application database initialization.

Use optimistic concurrency for Cart updates; conflicts are surfaced as application errors.

Persistence-specific configuration should live in `ShoppingCart.Infrastructure`.

Do not add database-specific annotations to Domain entities unless clearly justified.

---

## 9. Testing

Every meaningful business rule should have automated tests.

Use:

* unit tests for Domain business rules
* integration tests for important API/database workflows

When modifying existing functionality:

* update existing tests where required
* add tests for new behavior
* add regression tests for bugs

Do not remove or disable tests simply to make the build pass.

---

## 10. Validation Before Completion

Before marking a task complete, run:

```bash
dotnet restore
dotnet build
dotnet test
```

If the task involves HTTP behavior, also verify the relevant endpoint.

If the task involves database changes, verify the migration can be created/applied successfully.

Report:

* what was changed
* tests added or changed
* commands executed
* test results
* assumptions made
* unresolved issues

---

## 11. Git Changes

Keep commits focused.

Do not mix unrelated refactoring with feature implementation.

Avoid formatting or rewriting unrelated files.

Do not modify requirements or architecture documentation simply to make implementation easier.

If requirements appear inconsistent, raise the issue instead of silently changing them.

---

## 12. Task Execution Workflow

For every implementation task:

```text
1. Read relevant documentation
2. Inspect existing code
3. Understand current conventions
4. Prepare a short implementation plan
5. Implement only the requested scope
6. Add/update tests
7. Run validation commands
8. Review the diff
9. Fix discovered issues
10. Summarize the completed work
```

---

## 13. Important Agent Behavior

If something is ambiguous:

* prefer existing project conventions
* prefer the smallest reasonable implementation
* clearly state important assumptions

If an architectural change is required:

* do not make it silently
* explain the proposed change
* explain the trade-off
* wait for approval when the change materially affects project architecture

Do not claim a task is complete if tests or validation are failing.
