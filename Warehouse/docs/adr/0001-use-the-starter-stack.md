# ADR-0001: Use the starter stack

**Status:** Proposed

**Date:** 2026-11-11

**Deciders:** Warehouse project team

## Context

Warehouse workers, managers, drivers and customers need browser screens and shared data.

## Decision

Use the starter Vue and Vuetify frontend, one Express API under /api, and one MySQL database through Sequelize. Separate features by responsibility inside the API.

## Consequences

- The team can use the starter project and run the application locally.
- The API and database need to be reachable for stock and order work.

## Alternatives considered

Separate services would require extra deployments and make shared stock updates harder to coordinate.

## Related artifacts

[C4 diagrams](../Arch_Diagrams/README.md)
