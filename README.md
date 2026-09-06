# Shopping Cart MVP

Lightweight backend for discovering products and managing a shopping cart.

## Features

- Browse the list of products.
- View details of a single product.
- Create or retrieve an active shopping cart.
- Add, update, and remove items in the cart.
- Clear the cart.
- View the cart and its running total.

This MVP intentionally excludes checkout, payment, authentication, inventory management, and product-catalog administration.

## Technology Stack

* .NET 10
* ASP.NET Core Minimal APIs
* Entity Framework Core
* PostgreSQL
* xUnit

## Documentation

* [Architecture](docs/architecture.md) — current architecture, project boundaries, and constraints.
* [Product Requirements](docs/product.md) — MVP features and business rules.
* [Agent Guidelines](AGENTS.md) — engineering guidelines for contributors.
* [ADRs](docs/adr/) — accepted architectural decision records.

## Project Structure

```text
src/
├── ShoppingCart.Api
├── ShoppingCart.Application
├── ShoppingCart.Domain
└── ShoppingCart.Infrastructure

tests/
├── ShoppingCart.Domain.Tests
└── ShoppingCart.IntegrationTests
```

The system is a **modular monolith** organized as **vertical slices**. See [docs/architecture.md](docs/architecture.md) for dependency rules and design details.

## Local Development

1. Start PostgreSQL.  
   See [ADR-014](docs/adr/014-local-development-with-docker-compose.md) for Docker Compose guidance.

2. Apply EF Core migrations.  
   See [ADR-015](docs/adr/015-database-migrations.md) for migration commands.

3. Run the API:

   ```bash
   dotnet run --project src/ShoppingCart.Api
   ```

4. Open the generated OpenAPI document/UI to explore the API.

## Testing

```bash
dotnet restore
dotnet build
dotnet test
```

Unit tests cover domain behavior. Integration tests run against an isolated PostgreSQL container. See [ADR-006](docs/adr/006-testing-strategy.md) for the testing policy.
