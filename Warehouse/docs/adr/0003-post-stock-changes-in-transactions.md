# ADR-0003: Post stock changes in transactions

**Status:** Proposed

**Date:** 2026-11-11

**Deciders:** Warehouse project team

## Context

A receipt, transfer, reservation or departure can affect several records. A repeated request or a partial update can leave counts wrong.

## Decision

Use one database transaction for each stock operation. Recheck availability while updating the related balances and reservations, and keep a unique operation reference for repeat-sensitive posts.

## Consequences

- Stock, history and order totals stay consistent when a request fails or is repeated.
- Stock-changing operations need shared tests for retries, failures and concurrent requests.

## Alternatives considered

Independent writes could leave the order updated even if the inventory write fails.

## Related artifacts

[Feature 5](../../features/feature-5-inventory-control.md) · [Feature 10](../../features/feature-10-customer-order-management.md) · [Feature 12](../../features/feature-12-shipping-verification.md)
