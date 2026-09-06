# ADR-008: Use Multiple .NET Projects with Clear Dependency Boundaries

## Status

Accepted

## Context

The Shopping Cart MVP uses:

- Modular Monolith
- Vertical Slice Architecture
- Rich Cart domain model
- EF Core + PostgreSQL
- Minimal APIs

Although the MVP is small, we want architectural boundaries to be enforced
by the compiler rather than only by folder conventions.

At the same time, we do not want excessive project fragmentation.

## Decision

Use the following .NET projects:

src/
├── ShoppingCart.Api
├── ShoppingCart.Application
├── ShoppingCart.Domain
└── ShoppingCart.Infrastructure

tests/
├── ShoppingCart.Domain.Tests
└── ShoppingCart.IntegrationTests

Business use cases will be organized as vertical slices primarily inside
the Application project.

## Dependency Rules

ShoppingCart.Domain
- references no other application project

ShoppingCart.Application
- references Domain

ShoppingCart.Infrastructure
- references Application and Domain

ShoppingCart.Api
- references Application and Infrastructure

Dependency direction:

Api
 ├── Application
 └── Infrastructure
        ↓
    Application
        ↓
      Domain

Domain must not reference:

- Application
- Infrastructure
- Api
- EF Core
- ASP.NET Core

## Feature Organization

Application code should be organized by business capability and use case.

Example:

ShoppingCart.Application/

Products/
├── GetProducts/
└── GetProductById/

Carts/
├── CreateCart/
├── GetCart/
├── AddItem/
├── UpdateQuantity/
├── RemoveItem/
└── ClearCart/

Each slice may contain its own:

- request/command/query
- handler
- response DTO
- validation
- application-specific errors

## Infrastructure

Infrastructure contains technical implementations such as:

- EF Core DbContext
- PostgreSQL configuration
- entity mappings
- migrations
- repository implementations when needed

## API

API contains:

- Minimal API endpoint registration
- HTTP request/response mapping
- ProblemDetails mapping
- dependency injection/bootstrap configuration

The API layer should not contain domain business rules.

## Consequences

### Positive

- Architectural boundaries are compiler-enforced.
- Domain remains independent from infrastructure.
- Vertical slices remain easy to implement independently.
- Devin can work on one feature without modifying unrelated areas.
- Infrastructure can be replaced without changing domain logic.

### Negative

- Slightly more project setup than a single-project application.
- Some dependency-injection wiring is required.
- Developers must avoid creating unnecessary abstractions between projects.

## Alternatives Considered

### Single ASP.NET Core project

Rejected because folder boundaries alone are easier to violate accidentally.

### One project per feature

Rejected because it would create excessive project fragmentation for the MVP.

### Traditional layer-first folder structure

Rejected as the primary organization because ADR-002 selected Vertical Slice Architecture.