# ADR-010: Define Transactions Around Application Commands

## Status

Accepted

## Context

The Shopping Cart application contains commands that modify domain state.

Examples:

- CreateCart
- AddCartItem
- UpdateCartItemQuantity
- RemoveCartItem
- ClearCart

A command may require several steps before persistence.

For example, AddCartItem may:

1. Load the Product.
2. Verify that it exists and is active.
3. Load the Cart.
4. Apply Cart domain behavior.
5. Persist the resulting Cart state.

We want each command to either complete successfully or leave persisted
state unchanged.

At the same time, unnecessary explicit database transactions add complexity.

## Decision

Treat one application command as one logical transaction boundary.

Application handlers should:

1. Load required state.
2. Perform validation and orchestration.
3. Invoke domain behavior.
4. Register aggregate changes with the appropriate repository.
5. Call `IUnitOfWork.SaveChangesAsync()` once at the end where practical.

`IUnitOfWork` is a minimal abstraction defined in `ShoppingCart.Application`
and implemented in `ShoppingCart.Infrastructure` using EF Core's `DbContext`.
It exposes a single `SaveChangesAsync(CancellationToken)` method that triggers
EF Core's transactional `SaveChanges`.

Repositories (`ICartRepository`, `IProductReader`) do not call `SaveChanges`
internally. They load, add, or update aggregates; the handler commits the unit
of work through `IUnitOfWork`.

EF Core's normal SaveChanges transaction behavior should be used for
single-database changes.

Do not manually begin database transactions by default.

Explicit transactions should only be introduced when a use case requires
multiple persistence operations that cannot safely be committed through
a single SaveChanges operation.

## Example

AddCartItem:

Load Product
    ↓
Load Cart
    ↓
cart.AddItem(...)
    ↓
Register updated Cart with ICartRepository
    ↓
IUnitOfWork.SaveChangesAsync
    ↓
Commit

If domain validation fails before SaveChanges, no database state should
be changed.

## Failure Behavior

If persistence fails:

- the command is considered unsuccessful
- partial database changes must not be committed
- the API returns an appropriate error response

Concurrency conflicts are handled according to ADR-007.

## External Systems

Database transactions must not be held open while calling external systems.

If future requirements introduce operations such as:

- payment providers
- message brokers
- notification systems

their consistency strategy should be handled separately, potentially using
patterns such as the transactional outbox.

Those concerns are outside the current MVP.

## Consequences

### Positive

- Commands have clear atomic boundaries.
- Domain state is not partially persisted.
- Handlers remain relatively simple.
- We use EF Core transaction behavior rather than duplicating it.
- Devin receives a clear persistence rule.

### Negative

- Long-running workflows may eventually require different consistency patterns.
- Cross-system transactions are intentionally not supported.
- Developers must commit through `IUnitOfWork` and avoid calling `SaveChanges`
  repeatedly inside one simple command.
- Requires a small `IUnitOfWork` abstraction to keep EF Core out of `Application`.

## Alternatives Considered

### Explicit transaction in every handler

Rejected because EF Core already provides transactional behavior for normal
SaveChanges operations and explicit transactions would add unnecessary
boilerplate.

### Save after every repository operation

Rejected because partial state could be committed before the complete
business operation succeeds.

### Distributed transactions

Rejected because the MVP uses one PostgreSQL database and has no external
transactional systems.