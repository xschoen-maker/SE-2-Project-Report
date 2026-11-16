# Feature: Company and Warehouse Management

## Header

**Feature ID:** 2  
**Branch pattern:** feature/2-company-and-warehouse-management  
**Assignment branch:** design-draft\
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Keep company details, warehouse addresses, and storage locations in one place.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md)  
**Sources:** Presentation slide 4: Company and Warehouse; Inventory.xlsx: Bin and Slot; supplier, customer, and shipping forms: company name and address.  
**Draft questions:** Confirm the meaning of Bin versus Slot in the spreadsheet and the number of warehouses.

## User Stories

### US-2.1: Maintain company details
**As an** office clerk  
**I want to** save the business name and address  
**So that** orders and shipping documents show the right sender

**Priority:** P1  
**Independent test:** Update the company address and view the company record.  
**Acceptance scenarios:** See US-2.1 under Gherkin AC.

### US-2.2: Maintain warehouse locations
**As a** manager  
**I want to** add a warehouse and its receiving, storage, packing, and shipping locations  
**So that** workers can record where products actually are

**Priority:** P1  
**Independent test:** Create a warehouse and a named storage location.  
**Acceptance scenarios:** See US-2.2 under Gherkin AC.

### US-2.3: Retire an unused location
**As a** manager  
**I want to** deactivate a location that is no longer used  
**So that** workers do not put stock in an unavailable location

**Priority:** P2  
**Independent test:** Deactivate an empty location and try to choose it for put-away.  
**Acceptance scenarios:** See US-2.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Company records MUST include a nonblank name and address.
- **FR-002**: Warehouses MUST have a name, address, company link, and active status.
- **FR-003**: Each location MUST belong to one warehouse and have a code unique within that warehouse.
- **FR-004**: Location types MUST distinguish receiving, storage, packing, shipping, and damaged-stock areas.
- **FR-005**: Locations MUST support bin and slot labels without assuming that items are stored in a fixed product order.
- **FR-006**: Workers MUST be able to find locations by warehouse and code.
- **FR-007**: Deleting a referenced company, warehouse, or location MUST be blocked; staff may deactivate it instead.
- **FR-008**: A location with stock or an open task MUST NOT be deactivated until the stock and task are handled.

## Assumptions

- A location belongs to one warehouse. A company can operate more than one warehouse.

## Edge Cases

- A location still has stock or an open task. Block deactivation and show why.

## Success Criteria

- **SC-001** · Location codes are unique per warehouse, and staff can find active locations by code.

## Key Entities

- **Company**: the warehouse business shown on forms.
- **Warehouse**: a physical site belonging to a company.
- **Location**: a labeled place within a warehouse, including a bin or slot.

## Initial Data Model

### companies

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique company ID |
| name | STRING | Required |
| address | STRING | Required |
| phone, fax | STRING | Optional contact details |
| isActive | BOOLEAN | Defaults to true |

### warehouses

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique warehouse ID |
| companyId | INTEGER FK | References companies.id |
| name, address | STRING | Required |
| isActive | BOOLEAN | Defaults to true |

### locations

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique location ID |
| warehouseId | INTEGER FK | References warehouses.id |
| code | STRING | Required; unique within warehouse |
| bin, slot | STRING | Optional spreadsheet location labels |
| type | ENUM | receiving, storage, packing, shipping, damaged |
| isActive | BOOLEAN | Defaults to true |

## Gherkin AC

### US-2.1 - Maintain company details

#### Scenario: Maintain company details
* **Given** the company has a saved address
* **When** an authorized clerk corrects the address
* **Then** the new address is saved for future documents

### US-2.2 - Maintain warehouse locations

#### Scenario: Maintain warehouse locations
* **Given** a warehouse needs to be tracked
* **When** the manager saves its name, address, and a storage location code
* **Then** the warehouse and location appear in the location list

### US-2.3 - Retire an unused location

#### Scenario: Retire an unused location
* **Given** a location is empty and has no open task
* **When** the manager deactivates it
* **Then** it remains in history but cannot be selected for new stock tasks

#### Scenario: Keep an occupied location active
* **Given** a storage location holds 24 units
* **When** the manager attempts to deactivate it
* **Then** the system blocks deactivation and shows that stock remains

## Data Ownership & Isolation

Managers maintain companies, warehouses and locations. Workers read locations for their warehouse. Stock and task features own the records that block location deactivation. The API checks scope before returning or changing records.

## API Requirements

| Method | Endpoint | Auth | Input or action | Success |
|--------|----------|------|-----------------|---------|
| `GET` | `/api/warehouses` | Yes | company filter | 200 · warehouses |
| `POST` | `/api/companies` | Yes | name address | 201 · company |
| `PATCH` | `/api/companies/{id}` | Yes | name address active | 200 · company |
| `POST` | `/api/warehouses` | Yes | companyId name address | 201 · warehouse |
| `PATCH` | `/api/warehouses/{id}` | Yes | name address active | 200 · warehouse |
| `GET` | `/api/warehouses/{id}/locations` | Yes | code filter | 200 · locations |
| `POST` | `/api/warehouses/{id}/locations` | Yes | code type bin slot | 201 · location |
| `PATCH` | `/api/locations/{id}` | Yes | code type bin slot active | 200 · location |

Validation returns `400`. Missing sign-in returns `401` on protected routes. A role without the operation permission receives `403`. Missing or out-of-scope records return `404`. Error example: `{"message":"This location still holds stock."}`.

Example for `POST /api/warehouses/{id}/locations`.

Request: `{"code":"A-02","type":"storage","bin":"A","slot":"02"}`

Response: `{"id":12,"warehouseId":1,"code":"A-02","type":"storage","bin":"A","slot":"02","active":true}`

## Screen Requirements

### View: Warehouse Setup

- Route: `/warehouse-setup`
- Fields or results: Company, warehouse, address, location code, type, bin, slot.
- Actions: **Save location**, **Deactivate**.
- Empty: "No records to show." Loading: "Loading..." Error: "Could not complete the request."
- [Screen image](../docs/ui/warehouse-setup.png) · [Screen flow](../docs/ui/index.html#warehouse-setup)

## Definition of Done

- [ ] Check the Gherkin acceptance scenarios and the edge case above.
- [ ] Verify access through the UI and direct API requests.
- [ ] Check that the screen fields and API results support SC-001.

## Out of Scope

- Automatic slot assignment.
