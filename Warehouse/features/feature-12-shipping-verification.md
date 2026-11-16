# Feature: Shipping Verification

## Header

**Feature ID:** 12  
**Branch pattern:** feature/12-shipping-verification  
**Assignment branch:** design-draft\
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Verify picked orders, prepare shipping documents, and record shipment departure.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md), [Feature 10 - Customer Order Management](feature-10-customer-order-management.md), [Feature 11 - Order Picking](feature-11-order-picking.md)  
**Sources:** Presentation slide 5: Ship Orders to Customers; interview: shipping clerks verify, wrap, manifest, and hand goods to drivers; Bill_Of_Lading.docx.  
**Draft questions:** Confirm who approves partial shipments and whether wrapping or package counts need to be tracked.

## User Stories

### US-12.1: Verify picked goods
**As a** shipping clerk  
**I want to** compare the tub contents with the order  
**So that** wrong quantities do not go out unnoticed

**Priority:** P1  
**Independent test:** Verify a tub containing 10 of 12 requested units.  
**Acceptance scenarios:** See US-12.1 under Gherkin AC.

### US-12.2: Prepare a shipment
**As a** shipping clerk  
**I want to** prepare a bill of lading for verified quantities  
**So that** the driver and store have the same shipment details

**Priority:** P1  
**Independent test:** Create a bill of lading for a verified order.  
**Acceptance scenarios:** See US-12.2 under Gherkin AC.

### US-12.3: Record departure
**As a** shipping clerk  
**I want to** record when verified goods leave the warehouse  
**So that** stock and order progress reflect what was shipped

**Priority:** P1  
**Independent test:** Depart a 10-unit shipment and check stock and reservations.  
**Acceptance scenarios:** See US-12.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Shipping verification MUST compare actual picked units with the customer order and show differences.
- **FR-002**: Shipment lines MUST contain only verified picked quantities and MUST NOT exceed unshipped order commitments.
- **FR-003**: Partial shipment MUST keep the unsupplied remainder visible and require an authorized approval.
- **FR-004**: A bill of lading MUST save sender and receiver addresses, customer number, customer PO reference, shipping date, driver when assigned, SKU, description, quantity, saved price, and totals.
- **FR-005**: Document fields MUST be snapshots so later contact or price changes do not rewrite a shipped document.
- **FR-006**: Preparing or verifying a shipment MUST NOT deduct physical stock.
- **FR-007**: Confirming departure MUST deduct usable stock and release the matching reservation exactly once.
- **FR-008**: Departure MUST require a verified shipment assigned to a delivery run from Feature 13.
- **FR-009**: A shipment MUST show draft, verified, departed, and delivered states; delivery updates come from Feature 14.
- **FR-010**: A failed or repeated departure action MUST NOT leave stock and order totals partly updated.

## Assumptions

- The shipment leaves stock only when departure is confirmed.

## Edge Cases

- A departure is attempted without a delivery run. Block the departure before deducting stock or releasing reservations.

## Success Criteria

- **SC-001** · Verified units match the loaded shipment, and a repeated departure leaves quantities correct.

## Key Entities

- **Shipment**: verified goods leaving for a customer.
- **ShipmentLine**: actual quantities and document values.
- **BillOfLading**: saved shipment document using the supplied form.

## Initial Data Model

### shipments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique shipment ID |
| shipmentNumber | STRING | Required; unique |
| customerOrderId | INTEGER FK | References customerOrders.id |
| deliveryRunId | INTEGER FK | Required before departure; references deliveryRuns.id |
| status | ENUM | draft, verified, departed, delivered |
| verifiedById | INTEGER FK | References userAccounts.id |
| verifiedAt, departedAt | DATETIME | Set at corresponding steps |

### shipmentLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique line ID |
| shipmentId | INTEGER FK | References shipments.id |
| orderLineId | INTEGER FK | References customerOrderLines.id |
| quantityUnits | INTEGER | Positive verified quantity |
| sku, description | STRING | Saved document values |
| unitPrice | DECIMAL(12,2) | Saved order price |

### billsOfLading

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique document ID |
| shipmentId | INTEGER FK | References shipments.id |
| senderName, senderAddress, receiverName, receiverAddress | STRING | Required snapshots |
| customerNumber, poNumber | STRING | Saved customer and PO references |
| driverName | STRING | Saved when assigned |
| shippingDate | DATE | Required for departure |

## Gherkin AC

### US-12.1 - Verify picked goods

#### Scenario: Verify picked goods
* **Given** an order requests 12 units and the tub contains 10 picked units
* **When** the clerk checks the tub
* **Then** the two-unit difference is shown and the clerk cannot silently verify 12

### US-12.2 - Prepare a shipment

#### Scenario: Prepare a shipment
* **Given** an order's quantities have been verified
* **When** the clerk prepares its shipment document
* **Then** the document shows company and store addresses, customer and PO references, SKU, quantities, prices, and totals

### US-12.3 - Record departure

#### Scenario: Record departure
* **Given** 10 units are verified and reserved for a shipment
* **When** the clerk confirms departure
* **Then** on-hand and its reservation decrease by 10 and the shipment records its departure time

#### Scenario: Do not deduct a shipment twice
* **Given** a 10-unit shipment has already departed
* **When** the clerk repeats the departure action
* **Then** stock and reservations are not deducted again

## Data Ownership & Isolation

Shipping owns shipments, lines and bills of lading. Dispatch assigns shipments to runs. Departure updates stock and reservations together. Delivery records the final outcome. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/shipments` | Yes | orderId status filters | 200 · shipments |
| `POST` | `/api/shipments` | Yes | orderId lines | 201 · draft shipment |
| `POST` | `/api/shipments/{id}/verify` | Yes | verified lines and partial approval when needed | 200 · shipment |
| `GET` | `/api/shipments/{id}/bill-of-lading` | Yes | view saved shipping document | 200 · bill |
| `POST` | `/api/shipments/{id}/depart` | Yes | reference | 200 · shipment |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"Assign the shipment to a delivery run before departure."}`.

Example for `POST /api/shipments/{id}/depart`.

Request: `{"reference":"departure-17"}`

Response: `{"id":17,"status":"departed","reference":"departure-17"}`

## Screen Requirements

### View: Shipping

- Route: `/shipping`
- Fields or results: Order, picked units, verified units, difference, partial approval, bill of lading, assigned run.
- Actions: **Verify shipment**, **View bill of lading**, **Confirm departure**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/shipping.png) · [Screen flow](../docs/ui/index.html#shipping)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Carrier label integration.
