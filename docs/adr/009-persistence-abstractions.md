# ADR-009: Use Purpose-Specific Persistence Abstractions

## Status

Accepted

## Context

The application uses:

- Vertical Slice Architecture
- Rich Cart domain model
- EF Core
- PostgreSQL
- separate Application and Infrastructure projects

The Application layer must not depend directly on EF Core.

At the same time, introducing a generic repository such as:

IRepository<T>

with methods such as:

- Add
- Update
- Delete
- GetById
- GetAll

would hide useful domain intent and often duplicate functionality already
provided by EF Core.

## Decision

Do not introduce a generic repository abstraction.

Create persistence abstractions only where they represent meaningful
application/domain needs.

The Cart aggregate will use an aggregate-oriented repository.

Product reads will use a product-specific read abstraction.

Infrastructure will implement these abstractions using EF Core.

## Cart Repository

The Cart repository represents persistence of the Cart aggregate.

Example responsibilities:

- get cart by id
- get active cart for a customer identifier
- add a new cart
- track aggregate changes for the unit of work

Example interface:

ICartRepository
- GetByIdAsync(...)
- GetActiveByCustomerAsync(...)
- AddAsync(...)

The repository should operate on the Cart aggregate rather than exposing
CartItem persistence independently.

## Product Reads

Product behavior in the MVP is primarily read-only.

Use a purpose-specific abstraction for product queries.

Examples:

IProductReader
- GetByIdAsync(...)
- GetProductsAsync(...)

The Application layer should not need to understand EF Core query details.

## Unit of Work

Use a minimal `IUnitOfWork` abstraction to commit the unit of work once per
application command.

`IUnitOfWork` is defined in `ShoppingCart.Application` and implemented in
`ShoppingCart.Infrastructure` using EF Core's `DbContext`. It exposes
`SaveChangesAsync(CancellationToken)` so the Application layer can commit
without referencing EF Core.

Repositories do not call `SaveChanges` internally. They load, add, or update
aggregates and rely on the handler to commit through `IUnitOfWork`.

## Infrastructure

ShoppingCart.Infrastructure will contain:

- EF Core DbContext
- repository implementations
- product query implementations
- mappings
- migrations

Example:

CartRepository : ICartRepository
ProductReader : IProductReader

## Dependency Direction

Application
    ↓
interfaces

Infrastructure
    ↓
implements interfaces using EF Core

Domain
    ↑
remains persistence-independent

## Consequences

### Positive

- Application does not depend on EF Core.
- Persistence APIs reflect business intent.
- Cart aggregate boundaries remain clear.
- Avoids generic repository boilerplate.
- EF Core remains an implementation detail.
- Easier for Devin to understand what each abstraction is intended to do.

### Negative

- Additional interfaces and implementations are required.
- New query requirements may require extending or adding abstractions.
- Care must be taken not to create one interface per trivial operation.

## Alternatives Considered

### Generic IRepository<T>

Rejected because it obscures domain intent and duplicates many capabilities
already provided by EF Core.

### Direct DbContext usage from Application

Rejected because it would make the Application project depend on EF Core
and persistence-specific behavior.

### Repository for every entity

Rejected because entities such as CartItem belong to the Cart aggregate and
should not be persisted independently.