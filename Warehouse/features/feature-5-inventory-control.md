# Feature: Inventory Control

## Header

**Feature ID:** 5  
**Branch pattern:** feature/5-inventory-control  
**Assignment branch:** design-draft\
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Keep accurate stock counts by item, warehouse location, and condition.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md)  
**Sources:** Inventory.xlsx: Inv, Bin, Slot; interview: manual counts, human error, damage, and products stored wherever space is available.  
**Draft questions:** Confirm whether inventory currently means individual units; this draft counts units and separates usable stock from damaged stock.

## User Stories

### US-5.1: Find stock by location
**As a** warehouse worker  
**I want to** see where an item is stored and how much is there  
**So that** I can find products even when storage is not arranged by product

**Priority:** P1  
**Independent test:** Find an item held in two bins and check their combined stock.  
**Acceptance scenarios:** See US-5.1 under Gherkin AC.

### US-5.2: Correct a count
**As a** manager  
**I want to** record a counted quantity and the reason for a correction  
**So that** the system matches stock actually in the building

**Priority:** P1  
**Independent test:** Correct a count from 30 to 28 and review the movement history.  
**Acceptance scenarios:** See US-5.2 under Gherkin AC.

### US-5.3: Record damaged goods
**As a** receiving worker  
**I want to** separate damaged units from usable stock  
**So that** damaged goods are not promised to customers

**Priority:** P1  
**Independent test:** Mark 2 of 24 units damaged and check available stock.  
**Acceptance scenarios:** See US-5.3 under Gherkin AC.

### US-5.4: Protect available quantities
**As an** office clerk  
**I want to** see stock already committed to orders  
**So that** two customers are not promised the same goods

**Priority:** P1  
**Independent test:** Reserve 8 of 10 units and check available quantity.  
**Acceptance scenarios:** See US-5.4 under Gherkin AC.

## Functional Requirements

- **FR-001**: Stock MUST be recorded by warehouse, location, item, and usable or damaged condition.
- **FR-002**: All quantities MUST be whole individual units; case quantities MUST be converted using a saved case size.
- **FR-003**: Usable on-hand MUST include received usable goods in receiving, storage, packing, and shipping locations until shipment leaves.
- **FR-004**: Damaged stock MUST be shown separately and MUST NOT be included in usable on-hand or available-to-order quantities.
- **FR-005**: Reserved quantity MUST include active customer order commitments from Feature 10; available-to-order MUST equal usable on-hand minus reserved quantity.
- **FR-006**: Receiving MUST add stock once; put-away and picking MUST move stock without changing total on-hand; shipment departure MUST deduct stock once.
- **FR-007**: Authorized count corrections MUST include the old quantity, new quantity, reason, actor, and time.
- **FR-008**: Updates MUST prevent negative balances and duplicate processing of the same movement.
- **FR-009**: A correction below reserved stock MUST be blocked until staff resolve the affected commitments.
- **FR-010**: Recorded stock movements MUST be preserved; corrections MUST use additional movements rather than rewriting history.

## Assumptions

- Quantities are whole individual units. Availability is calculated per item and warehouse.

## Edge Cases

- A correction would leave fewer usable units than reserved. Reject the correction and show affected commitments.

## Success Criteria

- **SC-001** · Stock changes once per operation and history shows the worker, time and reason for corrections.

## Key Entities

- **StockBalance**: item quantity in a warehouse location and condition.
- **StockMovement**: recorded receipt, transfer, shipment, damage, or count correction.
- **Reservation**: active customer order commitment introduced in Feature 10; physical stock is not removed by reserving it.

## Initial Data Model

### stockBalances

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique balance ID |
| locationId | INTEGER FK | References locations.id; warehouse follows from location |
| itemId | INTEGER FK | References items.id |
| condition | ENUM | usable or damaged |
| quantity | INTEGER | Nonnegative; unique per location, item, and condition |

### stockMovements

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique movement ID |
| reference | STRING | Unique operation reference prevents repeat posting |
| itemId | INTEGER FK | References items.id |
| fromLocationId, toLocationId | INTEGER FK | Optional location links depending on movement type |
| fromCondition, toCondition | ENUM | usable or damaged where applicable |
| quantity | INTEGER | Positive units moved; correction also stores direction |
| type | ENUM | receipt, transfer, shipment, damage, correction |
| quantityBefore, quantityAfter | INTEGER | Required for count corrections; old and corrected balances |
| reason | TEXT | Required for damage and correction |
| actorId | INTEGER FK | References userAccounts.id |
| recordedAt | DATETIME | Required |

## Gherkin AC

### US-5.1 - Find stock by location

#### Scenario: Find stock by location
* **Given** an item has 12 usable units in one bin and 18 in another
* **When** the worker opens that item's stock
* **Then** both bins appear and usable on-hand quantity is 30

### US-5.2 - Correct a count

#### Scenario: Correct a count
* **Given** the system shows 30 usable units and the counted quantity is 28
* **When** an authorized manager saves a correction with a reason
* **Then** on-hand becomes 28 and history records a decrease of 2, the worker, and the time

#### Scenario: Reject an impossible correction
* **Given** 10 units are reserved and the location contains 12 usable units
* **When** the manager corrects usable stock to 8 without resolving reservations
* **Then** the correction is blocked and the commitments are identified

### US-5.3 - Record damaged goods

#### Scenario: Record damaged goods
* **Given** 24 usable units are in a location
* **When** the worker records 2 damaged units and a reason
* **Then** usable stock becomes 22, damaged stock becomes 2, and the change is recorded

### US-5.4 - Protect available quantities

#### Scenario: Protect available quantities
* **Given** usable on-hand stock is 10 and reservations total 8
* **When** the clerk checks availability
* **Then** available-to-order quantity is 2 and physical on-hand remains 10

## Data Ownership & Isolation

Inventory owns balances and stock movements. Feature 10 owns reservations. Receiving, put-away, picking and shipping request movements through the inventory service in their transaction. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/inventory` | Yes | warehouseId itemId locationId filters | 200 · balances and availability |
| `GET` | `/api/inventory/movements` | Yes | warehouseId itemId filters | 200 · movements |
| `POST` | `/api/inventory/corrections` | Yes | locationId itemId countedQuantity reason reference | 201 · movement |
| `POST` | `/api/inventory/damage` | Yes | locationId itemId quantity reason reference | 201 · movement |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"This adjustment would leave too little stock for reserved orders."}`.

Example for `POST /api/inventory/corrections`.

Request: `{"locationId":12,"itemId":3,"countedQuantity":28,"reason":"Physical count","reference":"count-14"}`

Response: `{"id":14,"type":"correction","quantityBefore":30,"quantityAfter":28,"reference":"count-14"}`

## Screen Requirements

### View: Inventory

- Route: `/inventory`
- Fields or results: Warehouse, SKU, location, usable units, damaged units, reserved units, reason.
- Actions: **Find stock**, **Save correction**, **Record damage**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/inventory.png) · [Screen flow](../docs/ui/index.html#inventory)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Inventory valuation and automatic cycle-count scheduling.
