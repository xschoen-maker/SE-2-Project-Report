# Feature: Supplier Receiving

## Header

**Feature ID:** 8  
**Branch pattern:** feature/8-supplier-receiving  
**Assignment branch:** design-draft\
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Record what arrived, compare it with the supplier order, and note missing or damaged products.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md), [Feature 7 - Supplier Order Management](feature-7-supplier-order-management.md)  
**Sources:** Presentation slide 5: Receive Orders from Suppliers; interview: verify received products, missing goods, damage, and notes.  
**Draft questions:** Confirm treatment of damaged arrivals: this draft stores them in a damaged area and leaves replacement units outstanding until cancelled or replaced. Confirm over-delivery approval.

## User Stories

### US-8.1: Receive an order
**As a** receiving worker  
**I want to** scan and count arriving products against the purchase order  
**So that** the warehouse records what actually arrived

**Priority:** P1  
**Independent test:** Accept one case of 24 units from a 48-unit order.  
**Acceptance scenarios:** See US-8.1 under Gherkin AC.

### US-8.2: Record damaged arrivals
**As a** receiving worker  
**I want to** separate damaged products and write a note  
**So that** unusable goods do not enter sellable stock

**Priority:** P1  
**Independent test:** Receive 22 usable units and 2 damaged units against a 24-unit order.  
**Acceptance scenarios:** See US-8.2 under Gherkin AC.

### US-8.3: Track incomplete deliveries
**As a** receiving worker  
**I want to** leave missing quantities open  
**So that** office staff can follow up with the supplier

**Priority:** P1  
**Independent test:** Receive 30 of 48 units and check the missing quantity.  
**Acceptance scenarios:** See US-8.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Receipts MUST identify purchase order, receiving location, worker, and receipt time.
- **FR-002**: Each receipt line MUST reference an ordered item and record accepted and damaged units separately.
- **FR-003**: Quantities MUST be whole units, nonnegative, and not all zero.
- **FR-004**: Posting accepted units MUST add usable stock at receiving and reduce outstanding on-order once.
- **FR-005**: Damaged units MUST be stored separately, excluded from usable stock, and recorded with a note.
- **FR-006**: Under the draft replacement rule, damaged units MUST NOT satisfy outstanding ordered quantities until the missing replacement is accepted or cancelled.
- **FR-007**: Unreceived quantities MUST remain visible; an incomplete arrival MUST NOT complete the purchase order.
- **FR-008**: Unexpected items and quantities above the remaining expected physical shipment MUST be blocked pending an authorized review.
- **FR-009**: A posted receipt MUST NOT be posted twice; correction MUST keep the original record and use a reviewed reversal.
- **FR-010**: A receipt MUST record both the actual arrival and remaining order quantity so supplier shortages can be reported.

## Assumptions

- Damaged units remain outstanding until replaced or cancelled under the draft rule.

## Edge Cases

- A line reports damage without a note. Reject posting and leave the order and inventory as they were.

## Success Criteria

- **SC-001** · Posting records accepted and damaged units separately. Repeating the request leaves quantities correct.

## Key Entities

- **SupplierReceipt**: an actual supplier arrival.
- **SupplierReceiptLine**: accepted, damaged, and noted quantities on an ordered line.

## Initial Data Model

### supplierReceipts

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique receipt ID |
| reference | STRING | Required; unique |
| purchaseOrderId | INTEGER FK | References purchaseOrders.id |
| receivingLocationId | INTEGER FK | References locations.id; receiving type |
| receivedById | INTEGER FK | References userAccounts.id |
| receivedAt | DATETIME | Required |
| status | ENUM | draft, posted, reversed |

### supplierReceiptLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique receipt line |
| receiptId | INTEGER FK | References supplierReceipts.id |
| purchaseOrderLineId | INTEGER FK | References purchaseOrderLines.id |
| acceptedUnits, damagedUnits | INTEGER | Nonnegative; at least one positive |
| damageNote | TEXT | Required when damagedUnits is positive |

## Gherkin AC

### US-8.1 - Receive an order

#### Scenario: Receive an order
* **Given** a supplier order has 48 units outstanding
* **When** the worker posts a receipt with 24 accepted units
* **Then** usable stock increases by 24 and outstanding on-order becomes 24

#### Scenario: Avoid posting a receipt twice
* **Given** a receipt for 24 accepted units is posted
* **When** the worker retries posting that receipt
* **Then** usable stock and on-order stay unchanged

### US-8.2 - Record damaged arrivals

#### Scenario: Record damaged arrivals
* **Given** 24 units are expected from a supplier
* **When** the worker records 22 usable units, 2 damaged units, and a damage note
* **Then** usable stock increases by 22, damaged stock increases by 2, and 2 replacement units remain outstanding

### US-8.3 - Track incomplete deliveries

#### Scenario: Track incomplete deliveries
* **Given** 48 units are outstanding
* **When** the worker posts 30 accepted units
* **Then** the order is partially received and 18 units remain outstanding

## Data Ownership & Isolation

Receiving owns receipt records and lines. A posted receipt updates purchasing and inventory together. Put-away reads accepted receipts. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/receipts` | Yes | purchaseOrderId filter | 200 · receipts |
| `POST` | `/api/receipts` | Yes | purchaseOrderId locationId reference lines with acceptedUnits damagedUnits note | 201 · receipt |
| `POST` | `/api/receipts/{id}/reverse` | Yes | reference reason | 201 · reversal |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"Add a note for the damaged items."}`.

Example for `POST /api/receipts`.

Request: `{"purchaseOrderId":7,"locationId":4,"reference":"receipt-9","lines":[{"purchaseOrderLineId":11,"acceptedUnits":12,"damagedUnits":0}]}`

Response: `{"id":9,"purchaseOrderId":7,"lines":[{"purchaseOrderLineId":11,"acceptedUnits":12,"damagedUnits":0,"remainingUnits":12}]}`

## Screen Requirements

### View: Supplier Receiving

- Route: `/supplier-receiving`
- Fields or results: PO number, receiving location, expected, accepted, damaged, remaining units, damage note.
- Actions: **Post receipt**, **Reverse receipt**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/supplier-receiving.png) · [Screen flow](../docs/ui/index.html#supplier-receiving)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Automatic supplier claims.
