# ADR-001: Use a Modular Monolith

## Status

Accepted

## Context

The Shopping Cart MVP contains Product and Cart capabilities.

The system does not currently require independent deployment,
independent scaling, or separate ownership of these capabilities.

## Decision

Use a modular monolith for the MVP.

## Consequences

- Single deployment unit
- Simpler development and operations
- Product and Cart remain separate logical modules
- A shared database may be used initially
- Modules can potentially be extracted later if required