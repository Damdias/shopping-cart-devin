# ADR-013: Use Environment-Based Configuration and Keep Secrets Out of Source Control

## Status

Accepted

## Context

The Shopping Cart backend requires configuration for values such as:

- PostgreSQL connection string
- logging levels
- health-check behavior
- future external service endpoints

These values may differ between:

- local development
- automated tests
- staging
- production

Sensitive values must not be stored in source control.

## Decision

Use ASP.NET Core's standard configuration system.

Configuration sources may include:

- appsettings.json
- appsettings.{Environment}.json
- environment variables
- .NET user secrets for local development

Sensitive values must not be committed to Git.

Production secrets should be injected through the deployment environment
or a dedicated secret-management system.

## Local Development

Non-sensitive defaults may be stored in:

appsettings.Development.json

Secrets such as database passwords should use either:

- environment variables
- dotnet user-secrets

Example:

dotnet user-secrets set \
  "ConnectionStrings:ShoppingCart" \
  "Host=localhost;Database=shoppingcart;Username=...;Password=..."

## Environment Variables

Environment variables should be supported using ASP.NET Core conventions.

Example:

ConnectionStrings__ShoppingCart

## Source Control

The repository must not contain:

- production passwords
- API keys
- access tokens
- private certificates
- cloud credentials

Example configuration files should use placeholders where necessary.

## Docker / Container Environments

Container deployments should receive configuration through environment
variables or mounted secret mechanisms.

Docker images must not contain embedded environment-specific secrets.

## Configuration Validation

Required configuration should be validated during application startup
where practical.

The application should fail fast when critical configuration such as the
database connection is missing or invalid.

## Logging

Secrets and full connection strings must not be written to logs.

## Consequences

### Positive

- Environment-specific configuration remains flexible.
- Secrets stay out of source control.
- Local development remains simple.
- Deployment systems can inject configuration independently.
- Devin has clear rules about where configuration belongs.

### Negative

- Developers must configure local secrets separately.
- Deployment environments require configuration management.
- Missing configuration may prevent startup.

## Alternatives Considered

### Hard-coded configuration

Rejected because it prevents environment separation and risks exposing secrets.

### Store secrets in appsettings.json

Rejected because repository contents may be shared, copied, or exposed.

### Build separate binaries for each environment

Rejected because configuration should be external to the application binary.