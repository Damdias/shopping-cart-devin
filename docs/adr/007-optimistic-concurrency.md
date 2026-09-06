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

Each Cart will have a concurrency version.

When EF Core attempts to update a Cart, the existing version must match the
version that was originally loaded.

If another request has already modified the Cart, the update should fail with
a concurrency conflict.

The application should translate that conflict into an appropriate application
error and API response.

## Consequences

### Positive

- Prevents silent lost updates.
- Avoids database locks during normal cart usage.
- Fits well with relatively short cart transactions.
- Straightforward to support with EF Core.

### Negative

- Clients may occasionally need to retry an operation.
- Conflict handling must be implemented consistently.
- Integration tests must cover concurrent modification behavior.

## API Behavior

A concurrency conflict should return:

HTTP 409 Conflict

using ProblemDetails.

The response should indicate that the cart has changed and the client should
reload the latest cart state before retrying.

## Domain Boundary

Concurrency is a persistence/application concern.

The Cart domain model should not contain HTTP or EF Core-specific logic.

## Alternatives Considered

### Last-write-wins

Rejected because it can silently lose cart updates.

### Pessimistic database locking

Rejected for the MVP because it introduces unnecessary locking complexity.

### Distributed locks

Rejected because the system is a modular monolith and does not require this
level of infrastructure.