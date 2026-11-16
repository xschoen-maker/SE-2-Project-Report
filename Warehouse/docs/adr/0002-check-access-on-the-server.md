# ADR-0002: Check access on the server

**Status:** Proposed

**Date:** 2026-11-11

**Deciders:** Warehouse project team

## Context

Staff need access based on their jobs. Customers need private access to their own orders.

## Decision

Check the active session, operation permission and record scope in the API. Resolve customer identity and assigned tasks from the session. Return 404 for records outside the caller’s scope.

## Consequences

- The same checks apply to direct API calls and browser requests.
- Each protected operation must identify the required permission and scope.

## Alternatives considered

Screen-level hiding cannot control requests sent directly to the API.

## Related artifacts

[Feature 1](../../features/feature-1-worker-accounts-and-access.md) · [Feature 17](../../features/feature-17-online-customer-ordering.md)
