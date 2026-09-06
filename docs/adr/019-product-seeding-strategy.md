# ADR-019: Use Explicit Product Seeding for Development and Tests

## Status

Accepted

## Context

The Shopping Cart MVP requires Product data for:

- browsing products
- viewing product details
- adding products to carts

Product catalog administration is outside the MVP.

Therefore, the application needs a simple way to create predictable product
data for local development and automated testing without introducing a full
catalog-management feature.

We must also avoid treating EF Core migrations as a general product-data
management mechanism.

## Decision

Use explicit, environment-specific product seeding.

Development environments may load a small deterministic product catalog.

Integration tests will create their own isolated test data.

Production environments must not automatically receive development/sample
products.

## Development Seed Data

Provide a small development catalog containing predictable products.

Example:

- Laptop
- Keyboard
- Mouse
- Monitor

Each seeded product should include the MVP fields such as:

- ProductId
- Name
- Description
- Price
- ImageUrl
- IsActive

Seed identifiers should be deterministic where useful so developers and
automated scenarios can refer to known products.

## Seed Execution

Development seeding must be explicit.

It may be implemented as:

- a dedicated seed command
- a development-only startup option
- a small CLI/tooling command

It must not silently populate production databases.

The seed operation should be safe to run repeatedly.

Where practical, it should be idempotent.

## Integration Tests

Integration tests must not depend on development seed data.

Each test or test fixture should arrange the minimum product data required
for its scenario.

Example:

Given an active product priced at 100
And an empty cart
When the product is added
Then the cart contains one item

Test data should be deterministic and isolated.

## Migrations

EF Core migrations remain responsible for database schema evolution.

Do not use migrations as the normal mechanism for maintaining a changing
development product catalog.

Static reference data may be migration-managed only when it is truly part of
the application's structural/reference model.

## Production

Production sample-data seeding is disabled by default.

A future catalog-management process or integration may own production Product
creation.

That concern is outside the MVP.

## Consequences

### Positive

- Developers can run the application immediately with useful sample data.
- Integration tests remain independent and reproducible.
- Production does not accidentally receive fake catalog data.
- Product seeding remains separate from schema migrations.
- Devin has a clear place to add sample data when needed.

### Negative

- A small amount of seeding infrastructure is required.
- Development seed data must be maintained when the Product model changes.
- Production catalog population remains a future concern.

## Alternatives Considered

### Seed Products through EF Core migrations

Rejected as the default approach because changing catalog data is not a schema
migration concern.

### Require developers to manually insert Product rows

Rejected because local setup would become inconsistent and error-prone.

### Build Product administration endpoints now

Rejected because catalog management is outside the MVP.