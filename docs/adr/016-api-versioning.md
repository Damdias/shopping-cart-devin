# ADR-016: Use Explicit API Versioning

## Status

Accepted

## Context

The Shopping Cart API may evolve after the MVP.

Future changes may introduce:

- authentication
- richer product data
- checkout
- promotions
- different cart behavior
- revised response contracts

Some future API changes may not be backward compatible.

Introducing an explicit version from the beginning avoids changing every route
later when versioning becomes necessary.

## Decision

Use URL-based API versioning.

The initial API version will be:

v1

Routes will use the following structure:

GET    /api/v1/products
GET    /api/v1/products/{productId}

POST   /api/v1/carts
GET    /api/v1/carts/{cartId}

POST   /api/v1/carts/{cartId}/items
PUT    /api/v1/carts/{cartId}/items/{productId}
DELETE /api/v1/carts/{cartId}/items/{productId}
DELETE /api/v1/carts/{cartId}/items

## Versioning Principle

A new API version should be introduced only for breaking contract changes.

Examples of breaking changes include:

- removing a response field
- changing the meaning of a field
- changing required request fields
- changing resource semantics
- changing identifier formats incompatibly

Non-breaking changes should normally remain within the current version.

Examples:

- adding optional response fields
- adding new endpoints
- adding optional query parameters

## Implementation

API versioning should remain an API-layer concern.

Domain and Application projects must not depend on API version numbers.

Feature code should avoid duplicating business logic across API versions.

If a future v2 is introduced:

HTTP contract v1
        ↓
Application use case

HTTP contract v2
        ↓
same application/domain behavior where possible

## Deprecation

When a future version replaces an older version, the older version should not
be removed without an explicit deprecation plan.

Deprecation policy is outside the MVP.

## Consequences

### Positive

- API evolution is explicit.
- Breaking changes can be introduced safely later.
- Client contracts clearly identify the version they target.
- Avoids having to retrofit version prefixes later.

### Negative

- Routes are slightly longer.
- Developers must avoid creating new versions unnecessarily.
- Multiple API versions may eventually require additional maintenance.

## Alternatives Considered

### No API versioning initially

Not selected because the API is intended to evolve and adding versioning later
would require changing existing client URLs.

### Header-based versioning

Not selected because URL-based versioning is simpler to discover, test, and
document for this MVP.

### Media-type versioning

Rejected because it adds unnecessary complexity for the current project.