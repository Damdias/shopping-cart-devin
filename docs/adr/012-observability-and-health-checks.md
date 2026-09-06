# ADR-012: Use Structured Logging, Correlation IDs, and Health Checks

## Status

Accepted

## Context

The Shopping Cart backend must be diagnosable during development,
testing, and production operation.

Because implementation is delegated incrementally to Devin, we also want
consistent observability rules across features.

The MVP does not require a full observability platform yet, but the application
should produce useful operational signals.

## Decision

The application will use:

- structured application logging
- request/correlation identifiers
- ASP.NET Core health checks
- consistent error logging
- logging scopes for request context

The application should remain compatible with future OpenTelemetry-based
metrics and tracing, but full distributed tracing is not required for the MVP.

## Structured Logging

Logs must use structured properties rather than string concatenation.

Prefer:

logger.LogInformation(
    "Cart {CartId} updated for customer {CustomerId}",
    cartId,
    customerId);

Avoid:

logger.LogInformation(
    "Cart " + cartId + " updated for customer " + customerId);

Important identifiers should be logged as named fields where appropriate.

Examples:

- CartId
- ProductId
- AnonymousCustomerId
- RequestId
- Operation

## Sensitive Data

Do not log:

- passwords
- authentication tokens
- secrets
- payment data
- full request bodies by default

Anonymous customer identifiers may be logged only where useful for diagnostics
and must not be treated as authenticated identity.

## Correlation / Request IDs

Every incoming HTTP request should have a request identifier.

If the client supplies an accepted correlation identifier, the application may
reuse it.

Otherwise, the application generates one.

The identifier should be included in:

- application log scope
- error responses where appropriate
- diagnostics for failed operations

## Error Logging

Expected business validation failures should not be logged as critical errors.

Examples:

- invalid quantity
- product not found
- cart not found

Unexpected failures should be logged with sufficient context and exception
details.

Domain objects should not perform logging directly.

Logging belongs in application/API/infrastructure boundaries.

## Health Checks

Expose health endpoints.

At minimum:

GET /health/live

Used to determine whether the process is running.

GET /health/ready

Used to determine whether the application is ready to serve requests.

Readiness should include PostgreSQL connectivity.

## Logging Responsibility

API
- request context
- correlation ID
- HTTP failure information

Application
- meaningful use-case events where useful
- command/query execution context

Infrastructure
- database or integration failures

Domain
- no logging framework dependency

## Consequences

### Positive

- Production problems are easier to diagnose.
- Logs can be searched by CartId or RequestId.
- Health probes support container/cloud deployment.
- Observability rules remain consistent across Devin-generated features.
- Future OpenTelemetry integration remains straightforward.

### Negative

- Developers must avoid excessive logging.
- Logging identifiers requires attention to privacy and security.
- Health checks require some infrastructure wiring.

## Alternatives Considered

### Console text logging only

Rejected because unstructured logs are harder to search and aggregate.

### Full distributed tracing immediately

Rejected because the MVP is a single modular monolith and does not yet
require distributed tracing infrastructure.

### No database readiness check

Rejected because the application is not meaningfully ready if PostgreSQL
cannot be reached.