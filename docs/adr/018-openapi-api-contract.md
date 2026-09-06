# ADR-018: Use OpenAPI as the API Contract

## Status

Accepted

## Context

The Shopping Cart backend exposes HTTP endpoints that may be consumed by:

- frontend applications
- automated tests
- future external clients
- developers using Devin to extend the system

The API contract must remain discoverable and synchronized with the actual
implementation.

Manually maintained endpoint documentation can easily become outdated.

## Decision

Expose an OpenAPI document generated from the ASP.NET Core API.

The OpenAPI contract should describe:

- routes
- HTTP methods
- request schemas
- response schemas
- status codes
- ProblemDetails responses
- API version

The API implementation remains the source from which the OpenAPI document
is generated.

## API Documentation

Development environments should provide an interactive API documentation
experience where appropriate.

The exact UI implementation may use the tooling supported by the selected
ASP.NET Core version.

The OpenAPI JSON document must remain available independently of any UI.

## Endpoint Documentation

Each endpoint should define enough metadata for the generated OpenAPI contract
to be useful.

Examples:

- operation name
- success response type
- expected error status codes
- request schema

Avoid documentation that simply repeats obvious implementation details.

## Response Contracts

Application/domain entities must not be exposed directly as API contracts.

Endpoints should return explicit response DTOs.

Example:

GetCartResponse

rather than serializing the Cart aggregate directly.

This allows the HTTP contract to evolve independently from internal domain
representation.

## Error Contract

ProblemDetails is the standard error contract according to ADR-005.

The OpenAPI document should describe relevant error responses.

Examples:

400 - invalid request
404 - resource not found
409 - business or concurrency conflict

## Versioning

The OpenAPI document must reflect the API versioning strategy from ADR-016.

Version-specific contracts should remain explicit if additional API versions
are introduced later.

## CI / Validation

Where practical, automated tests should verify that:

- the application can generate its OpenAPI document
- endpoint registration does not fail
- important response contracts remain serializable

Future CI may compare or publish OpenAPI contracts, but full schema-diff
enforcement is outside the MVP.

## Devin Workflow

When Devin adds or changes an endpoint, it must also verify:

- OpenAPI metadata remains correct
- request and response DTOs are documented
- expected error responses are represented
- no unrelated API contract changes were introduced

The PR description should call out any intentional API contract changes.

## Consequences

### Positive

- API consumers get machine-readable documentation.
- Documentation remains close to implementation.
- Frontend integration becomes easier.
- Future client generation remains possible.
- API changes are more visible during review.

### Negative

- Endpoint metadata requires some maintenance.
- Poorly named DTOs can produce confusing schemas.
- Generated documentation still needs human review for clarity.

## Alternatives Considered

### Maintain API documentation manually in Markdown only

Rejected because documentation can drift from implementation.

### Expose domain entities directly

Rejected because it couples external contracts to internal domain models.

### Build client SDKs manually during the MVP

Not selected because OpenAPI provides enough contract information for the
initial project.