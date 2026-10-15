# Feature: Customer Order Management

## Header

**Feature ID:** 10  
**Branch pattern:** feature/10-customer-order-management  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Record store orders, reserve available stock, and track the order through fulfillment.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md), [Feature 4 - Supplier and Customer Management](feature-4-supplier-and-customer-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md)  
**Sources:** Customer_Order.docx: customer number, PO number, SKU, description, quantity, price, totals, authorization, date; interview: fax orders and stock shortages.  
**Draft questions:** Confirm backorder policy and customer quantity units. Draft behavior reserves available units and leaves unfilled quantities visible for review rather than promising stock that is not available.

## User Stories

### US-10.1: Enter a store order
**As an** office clerk  
**I want to** enter the products a store ordered by fax or phone  
**So that** the order can be processed without a paper-only record

**Priority:** P1  
**Independent test:** Save 3 units at a price of 2.85 and check the total.  
**Acceptance scenarios:** See US-10.1 under Gherkin AC.

### US-10.2: Confirm available stock
**As an** office clerk  
**I want to** reserve the stock the warehouse can supply  
**So that** the store is not promised unavailable items

**Priority:** P1  
**Independent test:** Confirm an order for 8 units when 10 are available.  
**Acceptance scenarios:** See US-10.2 under Gherkin AC.

### US-10.3: Handle a shortage
**As an** office clerk  
**I want to** see which requested quantities cannot be filled  
**So that** I can discuss shortages with the store

**Priority:** P1  
**Independent test:** Request 12 units when only 10 are available.  
**Acceptance scenarios:** See US-10.3 under Gherkin AC.

### US-10.4: Cancel unshipped quantities
**As an** office clerk  
**I want to** cancel quantities the store no longer wants  
**So that** unused commitments return to available stock

**Priority:** P1  
**Independent test:** Cancel 4 reserved but unpicked units.  
**Acceptance scenarios:** See US-10.4 under Gherkin AC.

## Functional Requirements

- **FR-001**: Orders MUST link a customer, unique warehouse order number, customer PO number when supplied, order date, source, and authorized worker.
- **FR-002**: Order lines MUST save item, SKU and description snapshots, requested whole units, and selling price per unit.
- **FR-003**: Line totals MUST equal quantity multiplied by saved price; order total MUST equal its line totals with exact currency arithmetic.
- **FR-004**: Draft orders MUST NOT reserve stock.
- **FR-005**: Confirming MUST reserve only current available stock and MUST perform an atomic availability check so concurrent orders cannot reserve the same units.
- **FR-006**: Requested, reserved, picked, shipped, cancelled, and unfilled quantities MUST be distinguished.
- **FR-007**: A shortage MUST remain visible; the system MUST NOT show an unfilled quantity as shipped or guaranteed.
- **FR-008**: Cancelling unpicked and unshipped units MUST release their reservation without deducting physical stock.
- **FR-009**: Picked quantities MUST return from packing to a recorded usable location before cancellation releases them; shipped quantities require a separate return policy.
- **FR-010**: States MUST distinguish draft, confirmed, partly fulfilled, fulfilled, and cancelled; completion MUST require all lines to be supplied or explicitly cancelled.

## Key Entities

- **CustomerOrder**: store's request and processing status.
- **CustomerOrderLine**: requested quantities and saved selling price.
- **StockReservation**: active unshipped stock committed to an order line.

## Initial Data Model

### customerOrders

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique order ID |
| orderNumber | STRING | Required; unique |
| customerId | INTEGER FK | References customers.id |
| customerPoNumber | STRING | Optional store PO reference |
| orderDate | DATE | Required |
| source | ENUM | office entry or online |
| status | ENUM | draft, confirmed, partly fulfilled, fulfilled, cancelled |
| authorizedById | INTEGER FK | References userAccounts.id |

### customerOrderLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique line ID |
| orderId | INTEGER FK | References customerOrders.id |
| itemId | INTEGER FK | References items.id |
| sku, description | STRING | Saved snapshots |
| requestedUnits, cancelledUnits | INTEGER | Requested positive units; cancelled nonnegative units |
| unitPrice | DECIMAL(12,2) | Saved selling price |

### stockReservations

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique reservation ID |
| orderLineId | INTEGER FK | References customerOrderLines.id |
| warehouseId | INTEGER FK | References warehouses.id |
| remainingUnits | INTEGER | Reserved unshipped units; nonnegative |
| createdAt | DATETIME | Required |

## Gherkin AC

### US-10.1 - Enter a store order

#### Scenario: Enter a store order
* **Given** customer 137 and an active item exist
* **When** the clerk saves an order for 3 units at 2.85 each
* **Then** the line total is 8.55 and the order is linked to customer 137

### US-10.2 - Confirm available stock

#### Scenario: Confirm available stock
* **Given** 10 units are available and an order requests 8
* **When** the clerk confirms the order
* **Then** 8 units are reserved, 2 remain available, and physical on-hand stays 10

#### Scenario: Prevent two orders reserving the same stock
* **Given** 10 units are available and two orders each request 8
* **When** both orders are confirmed at the same time
* **Then** the total reserved quantity does not exceed 10 and the remaining shortage is shown

### US-10.3 - Handle a shortage

#### Scenario: Handle a shortage
* **Given** an order requests 12 units and only 10 are available
* **When** the clerk confirms the available quantity
* **Then** 10 units are reserved and 2 remain visibly unfilled without a delivery promise

### US-10.4 - Cancel unshipped quantities

#### Scenario: Cancel unshipped quantities
* **Given** an order has 4 reserved unpicked units
* **When** an authorized clerk cancels those units
* **Then** the reservation is released, the cancellation is saved, and physical stock stays unchanged
