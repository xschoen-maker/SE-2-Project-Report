# Feature: Online Customer Ordering

## Header

**Feature ID:** 17  
**Branch pattern:** feature/17-online-customer-ordering  
**Assignment branch:** design-draft\
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

## Assumptions

- A customer needs an approved active account linked to their customer record.

## Edge Cases

- The saved cart price changes before submission. Show the updated price for review before creating the order.

## Success Criteria

- **SC-001** · An approved customer can order on a phone and repeating the same request returns the same order.

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

## Data Ownership & Isolation

Online ordering owns cart lines and submission references. The signed-in approved customer owns their cart and can see only their own orders and shipment status. Feature 10 owns orders and reservations. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/shop/items` | Yes | active items | 200 · customer-facing catalog |
| `GET` | `/api/shop/cart` | Yes | current customer's cart | 200 · lines |
| `PUT` | `/api/shop/cart/{itemId}` | Yes | quantity, zero removes line | 200 · cart |
| `POST` | `/api/shop/orders` | Yes | requestReference and reviewed cart lines | 201 on first submission, 200 on replay · order |
| `GET` | `/api/shop/orders` | Yes | current customer's orders | 200 · orders |
| `GET` | `/api/shop/orders/{id}` | Yes | order and shipment status | 200 · order |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"The price changed. Review your cart before submitting."}`.

Example for `POST /api/shop/orders`.

Request: `{"requestReference":"order-26","lines":[{"itemId":3,"reviewedUnitPrice":"2.00","reviewedAvailableUnits":12}]}`

Response: `{"id":42,"status":"confirmed","lines":[{"itemId":3,"requestedUnits":6,"reservedUnits":6,"unfilledUnits":0}]}`

## Screen Requirements

### View: Shop

- Route: `/shop`
- Fields or results: SKU, description, unit price, quantity.
- Actions: **Add to cart**, **Review cart**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/shop.png) · [Screen flow](../docs/ui/index.html#shop)

### View: Cart and Orders

- Route: `/cart-and-orders`
- Fields or results: Cart quantities, updated prices and availability, order status.
- Actions: **Submit order**, **View my orders**.
- Empty: "No records to show." Loading: "Loading..." Error: "Review the latest price and availability."
- [Screen image](../docs/ui/cart-and-orders.png) · [Screen flow](../docs/ui/index.html#cart-and-orders)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Online payments and imports from outside customer systems.
