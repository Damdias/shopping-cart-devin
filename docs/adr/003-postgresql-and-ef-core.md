# ADR-003: Use PostgreSQL with Entity Framework Core

## Status

Accepted

## Context

The Shopping Cart MVP requires persistent storage for:

- Products
- Carts
- Cart items

The system is being built as a modular monolith and does not require
independent databases for each capability.

The development team needs:

- reliable relational persistence
- transactional consistency
- migrations
- strong ASP.NET Core integration
- straightforward local development and testing

## Decision

Use PostgreSQL as the relational database.

Use Entity Framework Core as the primary data access technology.

Products and Carts will use the same PostgreSQL database for the MVP.

Logical boundaries between Product and Cart data should still be preserved
in the application and persistence design.

## Consequences

### Positive

- Strong integration with ASP.NET Core
- EF Core provides migrations and change tracking
- PostgreSQL is production-ready and widely supported
- Relational constraints can enforce important data integrity rules
- A single database keeps MVP deployment and local development simple

### Negative

- Product and Cart persistence are not independently deployable
- Care is required to avoid unnecessary coupling between modules
- EF Core abstractions may not fit every future high-performance query

## Alternatives Considered

### In-memory persistence

Rejected because the MVP should behave like a real production-style backend
and data should survive application restarts.

### Separate databases for Products and Carts

Rejected because the MVP does not require independent deployment,
scaling, or ownership boundaries.

### Redis as the cart store

Rejected for the MVP to avoid unnecessary infrastructure complexity.

### Dapper

Not selected as the default because EF Core provides simpler migrations,
mapping, and unit-of-work behavior for this project.