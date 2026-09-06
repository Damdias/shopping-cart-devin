# ADR-014: Use Docker Compose for Local Infrastructure

## Status

Accepted

## Context

The Shopping Cart backend depends on PostgreSQL.

Local development should be:

- easy to start
- reproducible across developers
- close to production behavior
- independent from manually installed PostgreSQL instances

At the same time, forcing every developer to run the ASP.NET Core API inside
Docker can slow the inner development loop.

## Decision

Use Docker Compose to run local infrastructure.

For the MVP, Docker Compose will run PostgreSQL.

The ASP.NET Core API will normally run directly on the developer machine using:

dotnet run

The API may be containerized later for deployment or end-to-end environment
testing, but running it in Docker is not required for the normal local
development workflow.

## Local Development Flow

Developer starts infrastructure:

docker compose up -d

Then runs the application:

dotnet run --project src/ShoppingCart.Api

The API connects to the PostgreSQL container using local development
configuration.

## Docker Compose

The repository should contain:

docker-compose.yml

The PostgreSQL service should define:

- PostgreSQL image
- database name
- development username
- development password supplied through environment configuration
- persistent development volume
- health check
- exposed local port

No production credentials may be committed.

## Database Initialization

Schema creation should be handled through EF Core migrations.

Docker startup should not contain hidden business schema creation scripts
unless explicitly required.

The expected workflow is:

PostgreSQL starts
    ↓
Application / developer applies EF Core migration
    ↓
Database schema becomes ready

## Integration Tests

Integration tests remain independent from the development Docker Compose
environment.

According to ADR-006, integration tests should create their own disposable
PostgreSQL test container.

Tests must not depend on the developer's running Docker Compose database.

## API Containerization

The API may have a Dockerfile if required for deployment.

However, local development should not require rebuilding the API container
after every code change.

## Consequences

### Positive

- Developers do not need to install PostgreSQL manually.
- Local infrastructure is reproducible.
- Database version can be controlled.
- The normal .NET development loop remains fast.
- Integration tests remain isolated from local development state.

### Negative

- Docker or a compatible container runtime is required.
- Developers must understand container networking and volumes.
- Local infrastructure still requires some environment configuration.

## Alternatives Considered

### Install PostgreSQL directly on each developer machine

Rejected because versions and configuration may differ between developers.

### Run API and PostgreSQL entirely in Docker

Not selected as the default local workflow because rebuilding/restarting the
API container can slow development.

### Use an in-memory database locally

Rejected because PostgreSQL-specific behavior should be exercised during
development.