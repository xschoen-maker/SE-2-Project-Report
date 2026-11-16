# Quality attributes

App-wide non-functional targets for **this** product.

**Teaching policy:** Specs say *what* to build. This table says *how good* the system should be. Only **Accepted** rows (and feature **Requirements (FR-00N)** / **Success Criteria (SC-00N)**) constrain implementation. **Deferred** is the quality backlog / example. **Out of scope** is what not to build. See [`.cursor/rules/quality-attributes.mdc`](../../.cursor/rules/quality-attributes.mdc).

## Column meanings

| Column | Meaning |
|--------|---------|
| **Target** | Measure or bar — prefer a **number**. Illustrative until Status is **Accepted** and tests enforce it. |
| **Approach** | How the target is realized or limited |
| **Status** | **Accepted** / **Accepted (minimal)** / **Deferred** / **Out of scope** (see below) |
| **Links** | ADR (why), Cursor rule (how), code path, or **—** |

## Status values

| Status | Meaning |
|--------|---------|
| **Accepted** | In force — do not regress |
| **Accepted (minimal)** | Thin bar in force — do not expand |
| **Deferred** | Documented example — not a coding requirement unless a feature spec says so |
| **Out of scope** | Do not invent |

## Links column

| Link type | When |
|-----------|------|
| ADR | Approach decision |
| Cursor rule | Day-to-day agent/code constraint |
| Code path | Concrete implementation |
| **—** | No artifact yet |

| Attribute | Target | Approach | How we verify | Status | Links |
|-----------|--------|----------|---------------|--------|-------|
| **Security** | Every protected endpoint checks session and role. Customer records are scoped to the signed-in customer. | Use the shared permission and scope checks. | Try signed-out requests and a different customer’s order ID. | Accepted (minimal) | [ADR-0002](../adr/0002-check-access-on-the-server.md) |
| **Reliability** | A repeated or failed stock post leaves zero duplicate movements or partial changes. | Use a transaction and unique operation reference. | Retry a receipt and departure and simulate a failed write. | Accepted (minimal) | [ADR-0003](../adr/0003-post-stock-changes-in-transactions.md) |
| **Usability** | Assigned tasks and online ordering fit at a 360 px viewport without horizontal scrolling. | Use labeled fields and a layout that stacks on narrow screens. | Walk through a worker task and customer checkout at phone width. | Accepted (minimal) | [Screen flow](../ui/index.html) |
| **Accessibility (a11y)** | Primary screens are usable by keyboard and controls have visible labels. | Use native controls and preserve focus indicators. | Tab through sign-in, picking and online ordering. | Accepted (minimal) | [Screen flow](../ui/index.html) |
| **Maintainability** | **100%** Gherkin scenarios mapped before merge; `npm test` green | Cursor rules + feature specs as source of truth | Merge checklist; `npm test` | Accepted | [framework.md](../../features/framework.md), [quality-attributes.mdc](../../.cursor/rules/quality-attributes.mdc) |

---

## Feature-local NFRs

If only one feature needs a bar:

1. Put it in that feature’s **Requirements (FR-00N)** or **Success Criteria (SC-00N)**.
2. Add Gherkin when tests must prove it.
3. Optionally link from a row here (“see Feature N **FR-00N** / **SC-00N**”).

See [feature spec template](../../features/framework.md#feature-spec-template) in the teaching repo.

---

## Changing a bar

1. Edit **Target**, **Approach**, **Status**, **Links**. Prefer a number in **Target**.
2. **Approach** change → [ADR](../adr/README.md); add to **Links**.
3. Agent coding constraint → `.cursor/rules/`; add to **Links**.
4. API/schema change → `features/reference/` in the same feature PR ([Agent implementation request](../../features/framework.md#agent-implementation-request)).
5. **Deferred** → **Accepted** only with matching verification.
