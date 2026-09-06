# ADR-004: Use a Rich Domain Model for Cart

## Status

Accepted

## Context

The Shopping Cart MVP contains several business rules that belong directly
to cart behavior.

Examples include:

- cart item quantity must be between 1 and 99
- adding an existing product increases its quantity
- inactive products cannot be newly added
- inactive products already in the cart may remain visible
- quantity for an inactive product cannot be increased
- clearing a cart removes all line items but keeps the cart itself

If these rules are implemented only in API endpoints or application handlers,
they may be duplicated or bypassed.

## Decision

Use a rich domain model for the Cart aggregate.

The Cart aggregate will own and enforce core cart invariants and operations.

Application handlers coordinate use cases but should not duplicate domain rules.

Infrastructure concerns such as EF Core persistence remain outside the domain.

## Consequences

### Positive

- Business rules are centralized.
- Domain behavior is easier to test.
- Invalid cart state is harder to create.
- Application handlers remain focused on orchestration.
- Rules are less likely to be duplicated across API operations.

### Negative

- Domain entities contain more behavior than simple data models.
- EF Core mappings may require some additional configuration.
- Developers must distinguish domain rules from application workflow rules.

## Example Responsibility Split

### Cart domain

Responsible for:

- AddItem
- UpdateQuantity
- RemoveItem
- Clear
- enforcing quantity limits
- preventing invalid state

### Application layer / feature handler

Responsible for:

- loading Cart
- loading Product
- checking external data needed by the use case
- calling Cart behavior
- saving changes
- returning the response

### Infrastructure

Responsible for:

- PostgreSQL
- EF Core mappings
- repositories / DbContext
- migration