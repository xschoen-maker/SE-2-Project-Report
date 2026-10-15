# Feature: Item Catalog Management

## Header

**Feature ID:** 3  
**Branch pattern:** feature/3-item-catalog-management  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Identify the products sold, their barcodes, case sizes, supplier costs, and selling prices.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md)  
**Sources:** Inventory.xlsx: SKU, Item UPC, Case UPC, Description, Case Qty, Case Cost, Price; presentation slide 4: Items.  
**Draft questions:** Confirm whether customer quantities and prices are per unit or per case; this draft uses individual units for stock and customer order quantities.

## User Stories

### US-3.1: Add an item
**As an** office clerk  
**I want to** add a product with its SKU, description, barcodes, and case size  
**So that** workers can identify it while ordering and handling stock

**Priority:** P1  
**Independent test:** Save SKU 240 with a case size of 24 and find it by barcode.  
**Acceptance scenarios:** See US-3.1 under Gherkin AC.

### US-3.2: Update prices and case details
**As an** office clerk  
**I want to** correct prices and case sizes  
**So that** new orders use current information

**Priority:** P1  
**Independent test:** Change a price and check that an earlier order keeps its saved price.  
**Acceptance scenarios:** See US-3.2 under Gherkin AC.

### US-3.3: Retire an item
**As a** manager  
**I want to** stop offering an item without losing its history  
**So that** old orders remain understandable

**Priority:** P2  
**Independent test:** Deactivate an item and check customer ordering and history.  
**Acceptance scenarios:** See US-3.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Every item MUST have a unique SKU, nonblank description, positive whole-number case size, selling price, and active status.
- **FR-002**: Item and case UPCs MUST be stored as text so leading zeros are kept.
- **FR-003**: Barcode values MUST be unique across the item catalog and identify whether they refer to an item or a case.
- **FR-004**: Scanning an item barcode MUST represent one unit; scanning a case barcode MUST use the item's case size.
- **FR-005**: Selling price and supplier case cost MUST be nonnegative currency amounts stored with exact decimal precision.
- **FR-006**: Supplier costs MUST be stored per item and supplier rather than guessed from the selling price.
- **FR-007**: Changes to prices, descriptions, and case sizes MUST NOT change historical order line snapshots.
- **FR-008**: Case sizes of zero or less MUST be rejected.
- **FR-009**: Items referenced by orders or stock history MUST NOT be permanently deleted; inactive status MUST be available.

## Key Entities

- **Item**: a product identified by SKU.
- **ItemBarcode**: item or case barcode for a product.
- **SupplierItem**: a supplier's case cost and case size for that item; owned in Feature 4.
- **OrderLine snapshot**: saved description, unit basis, case size, and price at ordering time.

## Initial Data Model

### items

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique item ID |
| sku | STRING | Required; unique |
| description | STRING | Required |
| caseSize | INTEGER | Required; greater than zero |
| sellingPrice | DECIMAL(12,2) | Nonnegative price per unit in this draft |
| isActive | BOOLEAN | Defaults to true |

### itemBarcodes

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique barcode record |
| itemId | INTEGER FK | References items.id |
| value | STRING | Required; globally unique; preserve leading zeros |
| kind | ENUM | item or case |

## Gherkin AC

### US-3.1 - Add an item

#### Scenario: Add an item
* **Given** SKU 240 is not in the catalog
* **When** the clerk saves its description, item UPC, case UPC, case size 24, and selling price
* **Then** the item can be found by SKU and either barcode

#### Scenario: Reject a reused barcode
* **Given** barcode 073360709890 is assigned to one item
* **When** the clerk assigns it to another item
* **Then** the system rejects the duplicate barcode

### US-3.2 - Update prices and case details

#### Scenario: Update prices and case details
* **Given** an item has an earlier order and a current selling price
* **When** the clerk changes the selling price
* **Then** new orders use the new price and the earlier order price stays unchanged

#### Scenario: Reject an invalid case size
* **Given** an item has case size 24
* **When** the clerk changes its case size to zero
* **Then** the change is rejected and case size stays 24

### US-3.3 - Retire an item

#### Scenario: Retire an item
* **Given** an item appears on previous orders
* **When** the manager marks it inactive
* **Then** it is unavailable for new orders while previous orders still show it
