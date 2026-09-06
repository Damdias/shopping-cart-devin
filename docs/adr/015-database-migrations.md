# ADR-015: Manage Database Schema with Explicit EF Core Migrations

## Status

Accepted

## Context

The Shopping Cart application uses:

- PostgreSQL
- Entity Framework Core
- separate Infrastructure project
- Docker Compose for local PostgreSQL
- automated integration tests

The database schema will evolve as Product, Cart, and other capabilities
are implemented.

We need a repeatable and reviewable way to evolve the schema.

Automatically applying migrations whenever the API starts can be convenient
for development, but it creates operational risks in production environments.

## Decision

Use EF Core migrations as the source of truth for database schema evolution.

Migrations will live in:

ShoppingCart.Infrastructure

Database migrations must be created intentionally and committed to source
control.

The normal application startup must not automatically apply pending
migrations.

Migrations should be applied explicitly as part of:

- local development setup
- deployment workflow
- CI/CD where appropriate

## Migration Creation

When a persistence change requires a schema modification, create a migration.

Example:

dotnet ef migrations add AddCartTables \
  --project src/ShoppingCart.Infrastructure \
  --startup-project src/ShoppingCart.Api

The generated migration must be reviewed before committing.

## Applying Migrations

Local development:

dotnet ef database update \
  --project src/ShoppingCart.Infrastructure \
  --startup-project src/ShoppingCart.Api

Production deployment should apply migrations through an explicit deployment
step rather than application startup.

## Review Requirements

Before committing a migration, review:

- created tables
- modified columns
- indexes
- foreign keys
- unique constraints
- nullability changes
- destructive operations

Devin must not assume that a generated migration is correct simply because
EF Core created it.

## Integration Tests

Integration tests should apply the application's migrations to their
disposable PostgreSQL database.

This verifies that a new database can be created successfully from the
committed migration history.

Tests should not depend on EnsureCreated as the primary schema strategy.

## Seed Data

Development/test product data may be seeded separately.

Schema migrations should not become a general-purpose mechanism for
maintaining changing business/catalog data.

Static reference data may be handled through migrations only when that data
is truly part of the application's schema/reference definition.

## Production Safety

The application must not automatically execute:

Database.Migrate()

during normal production startup.

This avoids situations where multiple application instances attempt to
modify the production schema simultaneously.

## Consequences

### Positive

- Database changes are visible in source control.
- Schema changes can be reviewed in pull requests.
- Deployment controls when schema changes occur.
- Integration tests validate the actual migration chain.
- Production startup does not unexpectedly modify the database.

### Negative

- Developers must explicitly apply migrations.
- Deployment requires a migration step.
- Failed deployments may require coordination between application and
  database versions.

## Alternatives Considered

### Automatically migrate on API startup

Rejected because startup-time schema changes introduce production deployment
and concurrency risks.

### EnsureCreated

Rejected as the normal schema strategy because it bypasses migration history
and does not represent production schema evolution.

### Hand-written SQL migrations only

Not selected because EF Core migrations provide sufficient support for the
MVP while remaining reviewable.