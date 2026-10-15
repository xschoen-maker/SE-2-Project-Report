# Feature: Stock Put Away

## Header

**Feature ID:** 9  
**Branch pattern:** feature/9-stock-put-away  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Move received products into recorded storage locations.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md), [Feature 8 - Supplier Receiving](feature-8-supplier-receiving.md)  
**Sources:** Presentation slide 5: Put Away Orders from Suppliers; Inventory.xlsx: Bin and Slot; interview: products stored wherever space is available.  
**Draft questions:** Confirm whether managers assign put-away tasks or workers choose from a shared queue.

## User Stories

### US-9.1: View put-away work
**As a** put-away worker  
**I want to** see received goods waiting for storage  
**So that** I know what still needs to be moved

**Priority:** P1  
**Independent test:** Post a receipt and find its usable units in the put-away queue.  
**Acceptance scenarios:** See US-9.1 under Gherkin AC.

### US-9.2: Move goods to storage
**As a** put-away worker  
**I want to** scan the item and destination and confirm the quantity moved  
**So that** the next worker can find the goods

**Priority:** P1  
**Independent test:** Move 24 units from receiving to a storage bin.  
**Acceptance scenarios:** See US-9.2 under Gherkin AC.

### US-9.3: Split goods between locations
**As a** put-away worker  
**I want to** record parts of a receipt in different bins  
**So that** the system matches the space actually used

**Priority:** P1  
**Independent test:** Move 10 of 24 units to one bin and 14 to another.  
**Acceptance scenarios:** See US-9.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Put-away tasks MUST reference posted usable receipts and show remaining units at receiving.
- **FR-002**: A transfer MUST identify item, source location, destination, units, worker, and time.
- **FR-003**: Source and destination MUST be different active locations in the same warehouse.
- **FR-004**: Destination MUST be a usable storage location for the item condition.
- **FR-005**: Transferred quantity MUST be positive and not exceed usable units at the source.
- **FR-006**: Partial transfers MUST leave remaining units on the task; completion MUST require all task units to be handled.
- **FR-007**: Transfers MUST change location balances without adding another receipt or changing warehouse total on-hand.
- **FR-008**: Scanning a case MUST use the saved item case size and show the resulting unit quantity.
- **FR-009**: A completed transfer MUST be applied once and kept in stock movement history.

## Key Entities

- **PutAwayTask**: received units waiting to be stored.
- **StockMovement**: location transfer from Feature 5.

## Initial Data Model

### putAwayTasks

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique put-away task |
| receiptLineId | INTEGER FK | References supplierReceiptLines.id |
| sourceLocationId | INTEGER FK | References receiving location |
| assignedWorkerId | INTEGER FK | Optional; references workers.id |
| totalUnits, remainingUnits | INTEGER | Original accepted units and unhandled remainder |
| status | ENUM | pending, in progress, completed |

### putAwayTransfers

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique task transfer |
| taskId | INTEGER FK | References putAwayTasks.id |
| movementId | INTEGER FK | References stockMovements.id; unique |

## Gherkin AC

### US-9.1 - View put-away work

#### Scenario: View put-away work
* **Given** a posted receipt has 24 usable units at receiving
* **When** the worker opens pending put-away work
* **Then** the 24 units appear with their item and receiving location

### US-9.2 - Move goods to storage

#### Scenario: Move goods to storage
* **Given** 24 usable units are at receiving and an active storage bin exists
* **When** the worker confirms moving all 24 units to the bin
* **Then** receiving has zero, the bin has 24, and total usable stock is unchanged

#### Scenario: Reject moving more than is present
* **Given** 10 usable units remain at receiving
* **When** the worker attempts to put away 12 units
* **Then** the move is rejected and both location balances remain unchanged

### US-9.3 - Split goods between locations

#### Scenario: Split goods between locations
* **Given** 24 units are at receiving
* **When** the worker moves 10 to bin A and 14 to bin B
* **Then** both bin quantities are recorded and the task is complete
