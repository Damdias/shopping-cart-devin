# ADR-006: Testing Strategy

## Status

Accepted

## Context

The Shopping Cart backend contains domain business rules and persistence/API behavior.

Because implementation work will be delegated incrementally to Devin, each task
needs objective validation criteria before it is considered complete.

Mock-heavy tests alone would not prove that:

- EF Core mappings are correct
- PostgreSQL constraints work
- migrations are valid
- HTTP endpoints are correctly wired
- serialization and ProblemDetails responses behave correctly

## Decision

Use two primary test levels:

1. Unit tests for domain behavior
2. Integration tests for API and persistence behavior

Use xUnit as the test framework.

Integration tests run against a real PostgreSQL instance provisioned by
Testcontainers for .NET. The tests drive the API using ASP.NET Core's
`WebApplicationFactory` and apply the real EF Core migration history to the
disposable test database.

## Unit Tests

Unit tests should primarily cover domain behavior.

Examples:

- adding a new product creates a cart item
- adding the same product increases quantity
- quantity greater than 99 is rejected
- negative quantity is rejected
- updating quantity to zero removes the item
- inactive products cannot be newly added
- quantity of an inactive item cannot be increased
- clearing a cart removes all items

Unit tests should not require:

- PostgreSQL
- HTTP server
- EF Core
- external infrastructure

## Integration Tests

Integration tests should verify behavior across application boundaries.

Examples:

- GET /api/v1/products returns seeded products
- GET /api/v1/products/{id} returns 404 for unknown product
- POST /api/v1/carts creates a cart
- POST /api/v1/carts/{id}/items persists an item
- GET /api/v1/carts/{id} returns persisted cart state
- invalid requests return ProblemDetails
- EF Core migrations apply successfully
- database constraints behave as expected

## Test Database

Integration tests use an isolated, disposable PostgreSQL container managed by
Testcontainers for .NET. Each test run (or fixture) starts a fresh container,
applies the committed EF Core migration history, and uses `WebApplicationFactory`
to host the API in-process.

Tests must not depend on a developer's manually configured local database or the
Docker Compose development database. The test database must be disposable and
reproducible.

## Devin Completion Rule

A feature is not complete merely because the code compiles.

Before opening a PR, Devin must:

1. Build the solution.
2. Run unit tests.
3. Run integration tests relevant to the change.
4. Fix failing tests caused by the change.
5. Review the diff for unrelated changes.
6. Report what was tested in the PR description.

## Consequences

### Positive

- Business rules are protected by fast tests.
- Database behavior is tested against PostgreSQL rather than mocks.
- Devin receives objective completion criteria.
- Regression risk is reduced.
- Tests serve as executable requirements.

### Negative

- Integration tests are slower than unit tests.
- Container-based tests require Docker or compatible container runtime.
- Test setup requires additional infrastructure code.

## Alternatives Considered

### Unit tests only

Rejected because they do not validate EF Core, PostgreSQL, migrations, or HTTP behavior.

### Mock PostgreSQL/repositories

Rejected as the primary integration strategy because mocks can diverge from
actual database behavior.

### Shared development database for tests

Rejected because tests could interfere with each other and would not be
reproducible.