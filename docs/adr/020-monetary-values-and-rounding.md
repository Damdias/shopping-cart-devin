# ADR-020: Monetary Values and Rounding

## Status

Accepted

## Context

The Shopping Cart MVP must display product prices and compute cart totals
deterministically. The system uses a single currency and does not require
currency conversion.

Because money values are sensitive to rounding and representation, we need to
choose:

- the in-memory type for money values
- the database type for persisted money values
- the single currency
- rounding rules for line totals and the cart total

## Decision

Use C# `decimal` for all money values in the domain and application layers.

Persist money values as PostgreSQL `numeric(18,2)`.

The single currency is LKR. No currency conversion is supported in the MVP.

Each line item stores a unit price snapshot captured when the item is added.
Line totals are calculated using that snapshot and rounded to 2 decimal places
using `MidpointRounding.AwayFromZero`.

The cart total is the sum of rounded line totals.

Later catalog price changes do not alter existing cart items.

## Consequences

### Positive

- `decimal` avoids floating-point rounding errors common with `double` or `float`.
- `numeric(18,2)` preserves precision and scale in PostgreSQL.
- A single, explicit currency removes conversion complexity.
- `MidpointRounding.AwayFromZero` gives predictable, half-away-from-zero rounding.
- Capturing unit price snapshots in `CartItem` makes cart totals stable even when
catalog prices change.

### Negative

- Currency is hard-coded as LKR for the MVP; multi-currency support will require
a future redesign.
- `decimal` arithmetic is slightly slower than floating-point arithmetic, but
negligible for this workload.
- The choice of `MidpointRounding.AwayFromZero` must be applied consistently in
both domain calculations and tests.

## Alternatives Considered

### Use `double` or `float`

Rejected because binary floating-point types introduce representation and
rounding errors that are unacceptable for monetary calculations.

### Introduce a `Money` value object

Not selected for the MVP. A `Money` value object would encapsulate amount and
currency, but the single-currency assumption keeps the model simpler. It can be
introduced later if multi-currency or more complex money behavior is needed.

### Use `numeric` without scale

Rejected because `numeric(18,2)` explicitly enforces two decimal places and
keeps the database and application representations aligned.

### Round line totals with `MidpointRounding.ToEven`

Rejected for the MVP because `AwayFromZero` is more intuitive for business
users and matches common accounting expectations.
