# Feature: Supplier and Customer Management

## Header

**Feature ID:** 4  
**Branch pattern:** feature/4-supplier-and-customer-management  
**Assignment branch:** design-draft\
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Keep supplier and customer contacts, addresses, account numbers, and ordering information.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md)  
**Sources:** Supplier_Order.docx and Customer_Order.docx: account number, names, addresses; Inventory.xlsx: supplier, ship days, terms, minimum order; interview: fax and contact details.  
**Draft questions:** Confirm supplier minimum-order units: dollars, cases, or another measure. Confirm whether items may have several suppliers.

## User Stories

### US-4.1: Maintain supplier contacts
**As an** office clerk  
**I want to** save supplier contact and ordering details  
**So that** I know where to send orders and follow up on missing products

**Priority:** P1  
**Independent test:** Save a supplier with address, fax, ship days, and terms.  
**Acceptance scenarios:** See US-4.1 under Gherkin AC.

### US-4.2: Maintain customer records
**As an** office clerk  
**I want to** save a store's customer number and shipping address  
**So that** orders reach the correct store

**Priority:** P1  
**Independent test:** Create a store and find it by customer number.  
**Acceptance scenarios:** See US-4.2 under Gherkin AC.

### US-4.3: Link products to suppliers
**As an** office clerk  
**I want to** record each supplier's product cost and case size  
**So that** purchasing uses the right source and price

**Priority:** P1  
**Independent test:** Link SKU 240 to Arizona with its case cost and case size.  
**Acceptance scenarios:** See US-4.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Supplier and customer records MUST have unique account numbers, names, contact details, addresses, and active status.
- **FR-002**: Supplier and customer fax numbers MUST be supported as text and may be left blank.
- **FR-003**: Customers MUST support a shipping address separate from their business address.
- **FR-004**: Suppliers MUST support shipping lead time in days and payment terms.
- **FR-005**: A supplier's minimum-order value MUST include an explicit measurement type; unclear values MUST NOT be treated as currency or cases automatically.
- **FR-006**: Supplier-item links MUST have a case cost and positive case size; a supplier-item pair MUST be unique.
- **FR-007**: Staff MUST be able to search, correct, and deactivate records.
- **FR-008**: Deleting a party referenced by orders MUST be blocked; deactivation MUST preserve those orders.

## Assumptions

- Fax is optional. Staff must confirm the measurement type for each supplier minimum.

## Edge Cases

- A supplier minimum has a value without a measurement type. Reject the setting until staff select a type.

## Success Criteria

- **SC-001** · Staff can find and deactivate parties while preserving prior orders and costs.

## Key Entities

- **Supplier**: business supplying stock.
- **Customer**: store buying products.
- **SupplierItem**: product purchase terms for a supplier.

## Initial Data Model

### suppliers

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique supplier ID |
| accountNumber | STRING | Required; unique |
| name, address | STRING | Required |
| contactName, phone, fax | STRING | Optional contacts |
| shipDays | INTEGER | Optional; zero or more |
| paymentTerms | STRING | Optional |
| minimumOrderValue | DECIMAL(12,2) | Optional; nonnegative |
| minimumOrderUnit | ENUM | currency or cases; required when value is set |
| isActive | BOOLEAN | Defaults to true |

### customers

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique customer ID |
| customerNumber | STRING | Required; unique |
| name, businessAddress, shippingAddress | STRING | Required |
| contactName, phone, fax | STRING | Optional contacts |
| isActive | BOOLEAN | Defaults to true |

### supplierItems

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique supplier-item link |
| supplierId | INTEGER FK | References suppliers.id |
| itemId | INTEGER FK | References items.id |
| supplierSku | STRING | Optional supplier identifier |
| caseSize | INTEGER | Required; greater than zero |
| caseCost | DECIMAL(12,2) | Required; nonnegative |

## Gherkin AC

### US-4.1 - Maintain supplier contacts

#### Scenario: Maintain supplier contacts
* **Given** a supplier is not listed
* **When** the clerk saves its unique account number, name, address, fax, ship days, and payment terms
* **Then** the supplier appears with those details

### US-4.2 - Maintain customer records

#### Scenario: Maintain customer records
* **Given** a store has customer number 137
* **When** the clerk saves its name, contact details, and shipping address
* **Then** the store can be found by customer number 137

#### Scenario: Reject a duplicate customer number
* **Given** customer number 137 belongs to a store
* **When** the clerk adds a second store with customer number 137
* **Then** the second record is rejected

### US-4.3 - Link products to suppliers

#### Scenario: Link products to suppliers
* **Given** SKU 240 and supplier Arizona exist
* **When** the clerk saves the supplier's case cost and case size for that item
* **Then** the item appears in Arizona's purchasable product list

## Data Ownership & Isolation

Office staff maintain suppliers, customers and supplier-item links. Ordering reads these records and keeps its own snapshots. Supplier costs are restricted to authorized staff. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/suppliers` | Yes | account or name filter | 200 · suppliers |
| `POST` | `/api/suppliers` | Yes | accountNumber name contact address fax leadTimeDays paymentTerms minimum minimumType | 201 · supplier |
| `PATCH` | `/api/suppliers/{id}` | Yes | changed supplier fields and active | 200 · supplier |
| `GET` | `/api/customers` | Yes | account or name filter | 200 · customers |
| `POST` | `/api/customers` | Yes | accountNumber name contact address shippingAddress fax | 201 · customer |
| `PATCH` | `/api/customers/{id}` | Yes | changed customer fields and active | 200 · customer |
| `GET` | `/api/suppliers/{id}/items` | Yes | list supplier items | 200 · links |
| `PUT` | `/api/suppliers/{id}/items/{itemId}` | Yes | caseCost caseSize | 200 · supplier item |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"Choose a measurement type for the supplier minimum."}`.

Example for `PUT /api/suppliers/{id}/items/{itemId}`.

Request: `{"caseCost":"14.40","caseSize":12}`

Response: `{"supplierId":2,"itemId":3,"caseCost":"14.40","caseSize":12}`

## Screen Requirements

### View: Suppliers and Customers

- Route: `/suppliers-and-customers`
- Fields or results: Account number, name, addresses, contact, fax, lead time, payment terms, minimum and unit, supplier item cost and case size.
- Actions: **Save supplier**, **Save customer**, **Save supplier item**, **Deactivate**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/suppliers-and-customers.png) · [Screen flow](../docs/ui/index.html#suppliers-and-customers)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Supplier self-service accounts.
