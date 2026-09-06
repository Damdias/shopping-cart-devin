# ADR-002: Use Vertical Slice Architecture

## Status

Accepted

## Context

The Shopping Cart MVP contains a small number of business capabilities
such as browsing products and managing carts.

The application will be implemented incrementally using Devin.
We want each implementation task to have a clear scope and minimize
unnecessary changes across unrelated parts of the system.

Traditional technical-layer organization can cause a single feature
to be spread across controllers, services, repositories, DTO folders,
and other shared locations.

## Decision

Organize application behavior primarily by feature/use case using
Vertical Slice Architecture.

Examples:

- GetProducts
- GetProductById
- CreateCart
- GetCart
- AddCartItem
- UpdateCartItemQuantity
- RemoveCartItem
- ClearCart

Domain concepts and infrastructure concerns will still maintain
appropriate architectural boundaries.

## Consequences

### Positive

- Features are easier to understand independently.
- Devin can implement smaller, well-scoped tasks.
- Related request, handler, validation, and response code stays together.
- Features can be tested independently.
- Adding a feature requires fewer unrelated file changes.

### Negative

- Some duplication between slices may occur.
- Developers must avoid creating unnecessary shared abstractions too early.
- Cross-cutting concerns need consistent handling.

## Alternatives Considered

### Traditional Layered Architecture

Rejected as the primary organizational style because individual
features would be spread across multiple technical folders and layers.

### Microservices

Rejected for the MVP as defined in ADR-001.