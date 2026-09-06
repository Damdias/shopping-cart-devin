# ADR-007: Use Optimistic Concurrency for Cart Updates

## Status

Accepted

## Context

Multiple requests may attempt to modify the same shopping cart concurrently.

Examples include:

- two browser tabs updating quantities
- repeated client requests
- concurrent add/remove operations

Silently overwriting a newer cart state could result in lost updates.

The Shopping Cart MVP does not require distributed locking or pessimistic
database locks.

## Decision

Use optimistic concurrency for Cart persistence.

PostgreSQL's system `xmin` column is the concurrency token for the whole `Cart`
aggregate (including its `CartItem` collection).

`GET /api/v1/carts/{cartId}` returns the current `xmin` value in an `ETag`
response header. Mutating cart requests (`POST /api/v1/carts/{cartId}/items`,
`PUT /api/v1/carts/{cartId}/items/{productId}`,
`DELETE /api/v1/carts/{cartId}/items/{productId}`, and
`DELETE /api/v1/carts/{cartId}/items`) must include the `ETag` value in the
`If-Match` request header.

When EF Core attempts to update a `Cart`, the existing `xmin` must match the
value that was originally loaded. If another request has already modified the
aggregate, the update fails with a concurrency conflict.

The application translates that conflict into a `409 Conflict` `ProblemDetails`
response.

## Consequences

### Positive

- Prevents silent lost updates.
- Avoids database locks during normal cart usage.
- Fits well with relatively short cart transactions.
- Straightforward to support with EF Core.

### Negative

- Clients must supply the `If-Match` header on mutating requests and handle `409 Conflict` responses.
- Conflict handling must be implemented consistently.
- Integration tests must cover concurrent modification behavior.

## API Behavior

A concurrency conflict returns:

HTTP 409 Conflict

using `ProblemDetails` with a stable machine-readable `code` indicating that
the `Cart` has changed. The response should prompt the client to reload the
latest cart state (via `GET /api/v1/carts/{cartId}`) before retrying.

`GET /api/v1/carts/{cartId}` returns the current aggregate state with an
`ETag` header. Mutating requests must supply that value in the `If-Match`
header.

## Domain Boundary

Concurrency is a persistence/application concern.

The `Cart` aggregate as a whole is the consistency boundary for optimistic
concurrency; `CartItem` is part of the aggregate and is not persisted
independently.

The `Cart` domain model should not contain HTTP or EF Core-specific logic.

## Alternatives Considered

### Last-write-wins

Rejected because it can silently lose cart updates.

### Pessimistic database locking

Rejected for the MVP because it introduces unnecessary locking complexity.

### Distributed locks

Rejected because the system is a modular monolith and does not require this
level of infrastructure.