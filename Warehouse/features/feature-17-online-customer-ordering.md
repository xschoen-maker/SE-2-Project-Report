# Feature: Online Customer Ordering

## Header

**Feature ID:** 17  
**Branch pattern:** feature/17-online-customer-ordering  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Allow store customers to place and view their own orders online.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md), [Feature 4 - Supplier and Customer Management](feature-4-supplier-and-customer-management.md), [Feature 10 - Customer Order Management](feature-10-customer-order-management.md)  
**Sources:** Manager interview: customers should order directly online instead of paper; some customers use different systems.  
**Draft questions:** Confirm account approval, customer price rules, shortage handling, and whether external customer-system imports are required. No external integration is assumed.

## User Stories

### US-17.1: Browse the customer catalog
**As a** store customer  
**I want to** find items by SKU or barcode and see selling prices  
**So that** I can order the right products

**Priority:** P1  
**Independent test:** Find an active item and verify supplier costs are hidden.  
**Acceptance scenarios:** See US-17.1 under Gherkin AC.

### US-17.2: Submit an online order
**As a** store customer  
**I want to** send an order online and get an order number  
**So that** I do not need to fax the order

**Priority:** P1  
**Independent test:** Submit 3 available units at 2.85 each.  
**Acceptance scenarios:** See US-17.2 under Gherkin AC.

### US-17.3: View my order progress
**As a** store customer  
**I want to** see the status and shipped quantities of my own orders  
**So that** I know what to expect from a delivery

**Priority:** P1  
**Independent test:** View one owned order and try another store's order.  
**Acceptance scenarios:** See US-17.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Only active approved customer accounts linked to a customer record MUST be able to submit online orders.
- **FR-002**: The customer catalog MUST show active products, SKU, description, and selling price without supplier purchase costs.
- **FR-003**: Requested quantities MUST be positive whole units under this draft's unit convention.
- **FR-004**: Customer identity MUST come from the signed-in account and MUST NOT be accepted from a different customer ID in the request.
- **FR-005**: Submitting MUST use Feature 10's current price, stock reservation, and shortage rules; the system MUST check them again at submission.
- **FR-006**: A change in price or available quantity since the cart was prepared MUST be shown for customer confirmation before completing the order.
- **FR-007**: A submission retried with the same unique request reference MUST return the same order rather than create another one.
- **FR-008**: Customers MUST only read their own order and shipment information, including when they request another order directly.
- **FR-009**: The online workflow MUST work on a phone.
- **FR-010**: Importing orders from unrelated customer systems MUST stay outside this draft until formats and access requirements are confirmed.

## Key Entities

- **CustomerAccount**: UserAccount linked to a customer.
- **OnlineOrderRequest**: unique submission reference linked to a normal customer order.
- **CartLine**: unsent item quantities; creates no stock reservation.

## Initial Data Model

### onlineOrderRequests

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique request ID |
| reference | STRING | Required; unique per customer |
| customerId | INTEGER FK | References customers.id |
| customerOrderId | INTEGER FK | References customerOrders.id; one resulting order |
| submittedAt | DATETIME | Required |

### cartLines

| Field | Type | Rules |
|-------|------|-------|
| itemId | INTEGER | Existing active item |
| quantityUnits | INTEGER | Positive; unsent until confirmation |

## Gherkin AC

### US-17.1 - Browse the customer catalog

#### Scenario: Browse the customer catalog
* **Given** an approved customer is signed in
* **When** the customer searches for an active SKU
* **Then** its description and selling price appear without supplier costs or warehouse-only records

### US-17.2 - Submit an online order

#### Scenario: Submit an online order
* **Given** the customer has selected 3 available units at 2.85 each
* **When** the customer submits the order
* **Then** one customer order is created with a total of 8.55 and an order number

#### Scenario: Avoid duplicate online orders
* **Given** a customer already submitted an order with request reference R100
* **When** the same request is retried after a connection problem
* **Then** the existing order is returned and stock is not reserved again

### US-17.3 - View my order progress

#### Scenario: View my order progress
* **Given** two stores have separate customer accounts and orders
* **When** one store opens its order history
* **Then** only that store's orders and delivery progress are visible

#### Scenario: Block another store's order
* **Given** store A is signed in and an order belongs to store B
* **When** store A requests store B's order directly
* **Then** access is denied without revealing the order details
