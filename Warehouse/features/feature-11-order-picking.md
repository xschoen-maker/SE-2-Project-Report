# Feature: Order Picking

## Header

**Feature ID:** 11  
**Branch pattern:** feature/11-order-picking  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Guide workers to the stock locations and record what they place in an order's tub.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md), [Feature 10 - Customer Order Management](feature-10-customer-order-management.md)  
**Sources:** Presentation slide 5: Pick Orders from Customers; interview: workers walk to products, collect them into a tub, and take the tub to shipping.  
**Draft questions:** Confirm whether the warehouse uses a tub barcode or a manually entered tub number.

## User Stories

### US-11.1: View an assigned pick
**As a** picker  
**I want to** see an order's items, quantities, and stock locations  
**So that** I know what to collect

**Priority:** P1  
**Independent test:** Open a confirmed order with reserved stock and view its pick list.  
**Acceptance scenarios:** See US-11.1 under Gherkin AC.

### US-11.2: Pick into a tub
**As a** picker  
**I want to** scan the item, location, and order tub and record the picked quantity  
**So that** shipping receives the correct products for the order

**Priority:** P1  
**Independent test:** Pick 12 reserved units into tub T100.  
**Acceptance scenarios:** See US-11.2 under Gherkin AC.

### US-11.3: Report a picking problem
**As a** picker  
**I want to** record a missing or damaged item instead of marking it picked  
**So that** staff can fix the shortage before shipping

**Priority:** P1  
**Independent test:** Report a shortage of 2 units on a 12-unit pick.  
**Acceptance scenarios:** See US-11.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Pick tasks MUST be created only for confirmed reserved order quantities.
- **FR-002**: Each tub MUST have a unique identifier and be linked to one active customer order at a time.
- **FR-003**: Tasks MUST show SKU, description, reserved units, and usable stock locations.
- **FR-004**: A pick MUST verify the scanned item, source location, order tub, and quantity.
- **FR-005**: Picking MUST move usable stock from storage to packing and keep the reservation; it MUST NOT reduce total warehouse on-hand.
- **FR-006**: Picked units MUST NOT exceed reserved units or stock at the selected source.
- **FR-007**: Shortages or damage MUST be recorded with a note and keep the unpicked remainder visible.
- **FR-008**: A task MUST NOT be completed until each reserved quantity is picked or the order commitment is explicitly changed.
- **FR-009**: Pick records MUST include worker and time and MUST not be applied twice.

## Key Entities

- **PickTask**: work for a confirmed order.
- **Tub**: identified container holding the order's goods.
- **PickLine**: actual quantity collected for an order line.

## Initial Data Model

### tubs

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique tub ID |
| code | STRING | Required; unique |
| customerOrderId | INTEGER FK | Current order; references customerOrders.id |

### pickTasks

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique task ID |
| customerOrderId | INTEGER FK | References customerOrders.id |
| assignedWorkerId | INTEGER FK | References workers.id |
| tubId | INTEGER FK | References tubs.id |
| status | ENUM | pending, in progress, needs review, completed |

### pickLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique pick entry |
| taskId | INTEGER FK | References pickTasks.id |
| orderLineId | INTEGER FK | References customerOrderLines.id |
| movementId | INTEGER FK | References stockMovements.id; unique |
| pickedUnits | INTEGER | Positive units |
| problemNote | TEXT | Optional; required for a reported problem |
| pickedAt | DATETIME | Required |

## Gherkin AC

### US-11.1 - View an assigned pick

#### Scenario: View an assigned pick
* **Given** a confirmed order has 12 reserved units
* **When** the picker opens its assigned task
* **Then** the task shows the item, required units, and locations holding usable stock

### US-11.2 - Pick into a tub

#### Scenario: Pick into a tub
* **Given** 12 reserved units are in a storage bin and tub T100 belongs to the order
* **When** the picker confirms moving 12 units from the bin into that tub at packing
* **Then** the pick records 12 units, the location balance moves to packing, and warehouse on-hand is unchanged

#### Scenario: Reject the wrong product
* **Given** a task requests SKU 240
* **When** the picker scans a different SKU for that task
* **Then** the system blocks the pick and stock remains unchanged

### US-11.3 - Report a picking problem

#### Scenario: Report a picking problem
* **Given** the task calls for 12 units but only 10 usable units are found
* **When** the picker records 10 picked units and a shortage note
* **Then** 2 units remain unpicked and the task is flagged for review
