# Feature: Management Reporting

## Header

**Feature ID:** 15  
**Branch pattern:** feature/15-management-reporting  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Show stockouts, supplier shortages, delivery differences, and monthly shipment quantities.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md), [Feature 7 - Supplier Order Management](feature-7-supplier-order-management.md), [Feature 8 - Supplier Receiving](feature-8-supplier-receiving.md), [Feature 10 - Customer Order Management](feature-10-customer-order-management.md), [Feature 12 - Shipping Verification](feature-12-shipping-verification.md), [Feature 14 - Delivery Confirmation](feature-14-delivery-confirmation.md)  
**Sources:** Presentation slide 5: out-of-stock items, suppliers that do not ship ordered products, deliveries that do not match shipments, monthly shipped quantities.  
**Draft questions:** Confirm which timezone defines reporting dates and whether stockout means zero physical usable stock, zero available-to-order stock, or both. This draft shows them separately.

## User Stories

### US-15.1: Review stockouts
**As a** manager  
**I want to** see which products ran out and when  
**So that** I can improve purchasing decisions

**Priority:** P1  
**Independent test:** Show zero usable stock separately from stock fully reserved.  
**Acceptance scenarios:** See US-15.1 under Gherkin AC.

### US-15.2: Review supplier shortages
**As a** manager  
**I want to** compare ordered, accepted, cancelled, and outstanding units  
**So that** I can follow up with suppliers that sent too little

**Priority:** P1  
**Independent test:** Compare 48 ordered units with 30 accepted units.  
**Acceptance scenarios:** See US-15.2 under Gherkin AC.

### US-15.3: Review delivery differences
**As a** manager  
**I want to** compare shipped quantities with what each store accepted  
**So that** I can investigate damage and missing goods

**Priority:** P1  
**Independent test:** Compare 12 shipped units with 10 accepted and 2 damaged.  
**Acceptance scenarios:** See US-15.3 under Gherkin AC.

### US-15.4: Review monthly shipped quantities
**As a** manager  
**I want to** see the units shipped per item during a month  
**So that** I know which products move most often

**Priority:** P1  
**Independent test:** Report one 24-unit departed shipment and exclude a draft shipment.  
**Acceptance scenarios:** See US-15.4 under Gherkin AC.

## Functional Requirements

- **FR-001**: Reports MUST allow a date range and relevant item, warehouse, supplier, or customer filters.
- **FR-002**: Physical stockout and zero available-to-order MUST be reported separately with their definition shown.
- **FR-003**: Inventory history MUST support identifying when an item reached zero; current balances alone MUST NOT be treated as historical evidence.
- **FR-004**: Supplier reports MUST distinguish outstanding, overdue, damaged, accepted, and cancelled units.
- **FR-005**: Overdue status MUST use the order's agreed expected date or configured lead time, not label every outstanding order late.
- **FR-006**: Delivery reports MUST compare departed shipment quantities with saved delivery outcomes and notes.
- **FR-007**: Monthly shipment totals MUST sum actual departed units by item and departure date, excluding drafts and duplicate processing.
- **FR-008**: Reports MUST show unit quantities and state when a value is shown as cases.
- **FR-009**: Reports MUST be restricted to authorized managers and MUST not expose supplier cost to customers.
- **FR-010**: Invalid date ranges MUST be rejected and empty results MUST show zero values or an empty result message.

## Key Entities

- **Report**: calculated view of saved transactions, not a replacement for them.
- **StockoutInterval**: period derived from stock movement history.
- **SupplierShortage and DeliveryDifference**: comparisons using linked order, shipment, and receipt records.

## Initial Data Model

### calculatedReports

| Field | Type | Rules |
|-------|------|-------|
| itemId, warehouseId | INTEGER | Filters or grouping keys from existing records |
| usableOnHand, availableToOrder | INTEGER | Separate current stock measures |
| stockoutStartedAt, stockoutEndedAt | DATETIME | Derived from stock movement history |
| orderedUnits, acceptedUnits, cancelledUnits, outstandingUnits | INTEGER | Supplier order totals |
| shippedUnits, deliveredUnits, discrepancyUnits | INTEGER | Shipment and delivery comparisons |
| monthlyShippedUnits | INTEGER | Departed units grouped by item and report month |

## Gherkin AC

### US-15.1 - Review stockouts

#### Scenario: Review stockouts
* **Given** one item has no usable stock and another has usable stock entirely reserved
* **When** the manager opens stockout reporting
* **Then** physical stockouts and zero available-to-order stock are identified separately

### US-15.2 - Review supplier shortages

#### Scenario: Review supplier shortages
* **Given** a supplier order has 48 ordered and 30 accepted units with no cancellation
* **When** the manager opens supplier order reporting
* **Then** 18 outstanding units are shown; an order not yet due is not labeled overdue

### US-15.3 - Review delivery differences

#### Scenario: Review delivery differences
* **Given** a delivery records 12 shipped, 10 accepted, and 2 damaged units
* **When** the manager runs the delivery discrepancy report
* **Then** the 2 damaged units and their note are shown

### US-15.4 - Review monthly shipped quantities

#### Scenario: Review monthly shipped quantities
* **Given** a month contains one departed shipment of 24 units and a draft shipment of 12 units for the same SKU
* **When** the manager runs the month's shipment report
* **Then** the item shows 24 shipped units, and the draft shipment is excluded

#### Scenario: Handle a month with no shipments
* **Given** no shipments departed during a selected month
* **When** the manager runs the monthly report
* **Then** no-shipment results are shown without invented quantities
