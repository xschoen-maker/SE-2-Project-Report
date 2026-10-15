# Feature: Supplier Order Management

## Header

**Feature ID:** 7  
**Branch pattern:** feature/7-supplier-order-management  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Create and track purchase orders sent to suppliers.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md), [Feature 4 - Supplier and Customer Management](feature-4-supplier-and-customer-management.md)  
**Sources:** Supplier_Order.docx: ordered by/from, customer number, PO number, dates, SKU, description, cases, price, totals, authorized by; interview: orders received and not received.  
**Draft questions:** Confirm how purchase orders are sent: printed or exported form, fax service, email, or a supplier system. No automatic fax integration is assumed.

## User Stories

### US-7.1: Create a purchase order
**As a** purchasing clerk  
**I want to** prepare a supplier order with whole cases and prices  
**So that** the supplier receives a clear list of products to send

**Priority:** P1  
**Independent test:** Create a two-case order at a case cost of 34.20.  
**Acceptance scenarios:** See US-7.1 under Gherkin AC.

### US-7.2: Submit a purchase order
**As an** authorized office clerk  
**I want to** mark an approved order as sent  
**So that** the warehouse knows goods are on the way

**Priority:** P1  
**Independent test:** Submit a three-case order and check outstanding units.  
**Acceptance scenarios:** See US-7.2 under Gherkin AC.

### US-7.3: Track and cancel outstanding quantities
**As a** purchasing clerk  
**I want to** see partial receipts and cancel quantities no longer expected  
**So that** open orders show only stock still due

**Priority:** P1  
**Independent test:** Cancel the remaining 24 units after 24 of 48 units were received.  
**Acceptance scenarios:** See US-7.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Purchase orders MUST link a supplier, warehouse, unique PO number, order date, ordering company, and authorized account.
- **FR-002**: Lines MUST store SKU and description snapshots, whole cases, case size, case cost, ordered units, and line totals.
- **FR-003**: Ordered units MUST equal cases multiplied by saved case size; totals MUST use exact currency arithmetic.
- **FR-004**: Submitting MUST require valid supplier items, at least one positive line, and the submit permission.
- **FR-005**: Supplier minimum-order rules MUST be checked in their explicitly configured units before submission.
- **FR-006**: Only submitted orders MUST add outstanding on-order quantities; saving a draft MUST NOT do so.
- **FR-007**: States MUST distinguish draft, submitted, partially received, completed, and cancelled.
- **FR-008**: Outstanding units MUST equal ordered units minus accepted receipts minus cancelled units.
- **FR-009**: Cancellation MUST affect only unreceived units and preserve receipt and order history.
- **FR-010**: The order document MUST support the fields in Supplier_Order.docx; sending method remains a draft decision.

## Key Entities

- **PurchaseOrder**: purchase commitment to a supplier.
- **PurchaseOrderLine**: cases, costs, and receipt progress for one item.

## Initial Data Model

### purchaseOrders

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique purchase order ID |
| poNumber | STRING | Required; unique |
| supplierId, warehouseId | INTEGER FK | References suppliers.id and warehouses.id |
| orderDate | DATE | Required |
| supplierCustomerNumber | STRING | Warehouse account number with this supplier |
| status | ENUM | draft, submitted, partially received, completed, cancelled |
| authorizedById | INTEGER FK | References userAccounts.id |

### purchaseOrderLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique line ID |
| purchaseOrderId | INTEGER FK | References purchaseOrders.id |
| itemId | INTEGER FK | References items.id |
| sku, description | STRING | Order snapshots |
| cases, caseSize | INTEGER | Positive saved values |
| caseCost | DECIMAL(12,2) | Nonnegative saved purchase price |
| orderedUnits, cancelledUnits | INTEGER | Saved ordered units and cancelled remainder |

## Gherkin AC

### US-7.1 - Create a purchase order

#### Scenario: Create a purchase order
* **Given** an active supplier supplies an item at 34.20 per case
* **When** the clerk drafts an order for 2 cases
* **Then** the line total is 68.40 and the order remains draft

### US-7.2 - Submit a purchase order

#### Scenario: Submit a purchase order
* **Given** a draft order contains 3 cases of 24 units
* **When** the authorized clerk submits it
* **Then** the order is submitted and outstanding on-order increases by 72 units

#### Scenario: Prevent duplicate submission
* **Given** a purchase order is already submitted
* **When** the clerk retries the submit action
* **Then** the order stays submitted and its on-order quantity is not added again

### US-7.3 - Track and cancel outstanding quantities

#### Scenario: Track and cancel outstanding quantities
* **Given** a submitted 48-unit order has 24 accepted units
* **When** the authorized clerk cancels the remaining 24 units with a reason
* **Then** outstanding on-order becomes zero and received stock is unchanged
