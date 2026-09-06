# ADR-017: Separate Request Validation from Domain Invariants

## Status

Accepted

## Context

The Shopping Cart API receives external input such as:

- product identifiers
- cart identifiers
- item quantities
- anonymous customer identifiers

Some validation concerns are purely request-level.

Examples:

- required field missing
- malformed UUID
- quantity field not supplied

Other rules represent business invariants.

Examples:

- quantity must not exceed 99
- inactive products cannot be newly added
- quantity of an inactive cart item cannot be increased
- one active cart per customer

If all validation exists only at the HTTP boundary, another application entry
point could bypass important business rules.

If all validation exists only inside the Domain, clients receive poorer and
less immediate validation feedback.

## Decision

Use two validation layers:

1. Request/Application validation for input shape and use-case requirements.
2. Domain validation for business invariants.

Domain invariants must remain enforced even when similar validation is
performed earlier in the request pipeline.

## Request Validation

Request/Application validation should handle concerns such as:

- required fields
- malformed identifiers
- invalid request structure
- quantity outside the accepted request range
- missing command/query inputs

Validation errors should be returned as:

HTTP 400 Bad Request

using ProblemDetails.

## Domain Validation

The Domain owns rules that must never be bypassed.

Examples:

Cart.AddItem(...)
- ensures resulting quantity does not exceed 99
- prevents invalid cart state

Cart.UpdateQuantity(...)
- prevents invalid quantity
- prevents increasing quantity for an inactive product

Domain entities must not trust callers to have validated correctly.

## Application Validation

Application handlers may validate use-case conditions that require loading
external state.

Examples:

- Product exists
- Cart exists
- Product is active before adding
- Customer already has an active cart

Where a rule belongs to the Cart aggregate itself, the handler should delegate
final enforcement to the Domain.

## Validation Flow

HTTP Request
    ↓
Request validation
    ↓
Application handler
    ↓
Load required state
    ↓
Domain operation
    ↓
Domain invariant enforcement
    ↓
Persistence

## Error Mapping

Typical mappings:

Malformed request
    → 400 Bad Request

Invalid quantity
    → 400 Bad Request

Product not found
    → 404 Not Found

Cart not found
    → 404 Not Found

Business-state conflict
    → 409 Conflict

All API errors should use ProblemDetails according to ADR-005.

## Duplication

Some defensive duplication is acceptable when it serves different purposes.

Example:

The request validator may reject quantity > 99 immediately.

Cart must still reject quantity > 99 because the Domain cannot assume that
every caller came through the HTTP API.

## Validation Library

Do not introduce a third-party validation framework solely for trivial request
validation.

ASP.NET Core validation or small feature-specific validators are sufficient
initially.

A validation library may be introduced later if validation complexity grows
enough to justify it.

## Consequences

### Positive

- Invalid requests fail early.
- Domain rules cannot be bypassed.
- HTTP concerns remain outside the Domain.
- Validation responsibility is clearer.
- Devin has explicit guidance on where new rules belong.

### Negative

- Some rules may appear in more than one layer.
- Developers must distinguish input validation from business invariants.
- Poorly designed validation can still create unnecessary duplication.

## Alternatives Considered

### Validate only at the API boundary

Rejected because domain rules could be bypassed by non-HTTP callers.

### Validate everything only in Domain

Rejected because request-shape validation is not a domain responsibility.

### Add FluentValidation immediately

Not selected because the current MVP validation needs are small and do not yet
justify an additional dependency.