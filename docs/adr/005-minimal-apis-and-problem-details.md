# ADR-005: Use Minimal APIs and ProblemDetails

## Status

Accepted

## Context

The Shopping Cart backend uses Vertical Slice Architecture.

Each use case should remain small and independently understandable.

The API also needs a consistent error representation for cases such as:

- cart not found
- product not found
- invalid quantity
- inactive product
- validation failures

## Decision

Use ASP.NET Core Minimal APIs for HTTP endpoints.

Use RFC-compatible ProblemDetails responses for API errors.

Each feature may define its endpoint close to its request/handler/response
implementation.

## Consequences

### Positive

- Fits well with Vertical Slice Architecture
- Less controller boilerplate
- Feature code can remain colocated
- Consistent API error responses
- Easier for Devin to implement one endpoint/use case at a time

### Negative

- Endpoint organization must remain disciplined as the application grows
- Cross-cutting concerns need shared conventions
- Large endpoint files must be avoided

## API Conventions

Use resource-oriented REST endpoints where practical.

Examples:

GET /api/products

GET /api/products/{productId}

POST /api/carts

GET /api/carts/{cartId}

POST /api/carts/{cartId}/items

PUT /api/carts/{cartId}/items/{productId}

DELETE /api/carts/{cartId}/items/{productId}

DELETE /api/carts/{cartId}/items

## Error Handling

Use ProblemDetails for errors.

Examples:

- 400 Bad Request for invalid input
- 404 Not Found when a product or cart does not exist
- 409 Conflict when a business operation conflicts with current state

Domain or application errors should be translated into HTTP responses at
the API boundary.

Domain objects must not depend on HTTP concepts.