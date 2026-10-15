# Feature: Replenishment Planning

## Header

**Feature ID:** 6  
**Branch pattern:** feature/6-replenishment-planning  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Suggest whole-case purchases using minimum stock, maximum stock, and goods already on order.  
**Depends on:** [Feature 3 - Item Catalog Management](feature-3-item-catalog-management.md), [Feature 4 - Supplier and Customer Management](feature-4-supplier-and-customer-management.md), [Feature 5 - Inventory Control](feature-5-inventory-control.md)  
**Sources:** Interview: minimum, maximum, case rounding, and on-order stock; Inventory.xlsx column O formula IF(K< L,(M-K-N)/H,0).  
**Draft questions:** Draft trigger follows the spreadsheet's on-hand-below-minimum test. Confirm whether the manager instead wants on-hand plus on-order to control the trigger, and whether reservations affect it.

## User Stories

### US-6.1: Set reorder limits
**As a** manager  
**I want to** set minimum and maximum stock for each item  
**So that** purchasing has a clear target

**Priority:** P1  
**Independent test:** Save minimum 15 and maximum 50 for an item.  
**Acceptance scenarios:** See US-6.1 under Gherkin AC.

### US-6.2: Calculate whole-case orders
**As a** purchasing clerk  
**I want to** see the case quantity needed when stock is low  
**So that** I can restock without calculating every item by hand

**Priority:** P1  
**Independent test:** Calculate a purchase for 10 on hand, zero on order, maximum 50, and case size 12.  
**Acceptance scenarios:** See US-6.2 under Gherkin AC.

### US-6.3: Allow for stock on order
**As a** purchasing clerk  
**I want to** subtract outstanding supplier quantities from the purchase suggestion  
**So that** I do not buy the same shortage twice

**Priority:** P1  
**Independent test:** Calculate with 10 on hand and 12 already on order.  
**Acceptance scenarios:** See US-6.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Reorder settings MUST be stored per item and warehouse with minimum greater than or equal to zero and maximum greater than or equal to minimum.
- **FR-002**: The draft rule MUST return zero suggested cases when usable on-hand is at or above the minimum.
- **FR-003**: When on-hand is below minimum, the unit shortage MUST equal max(0, maximum - usableOnHand - outstandingOnOrder).
- **FR-004**: Suggested cases MUST equal ceiling(unitShortage / supplierCaseSize); suggested units MUST equal cases multiplied by case size.
- **FR-005**: Outstanding on-order MUST include only submitted supplier order quantities still awaiting receipt; draft and cancelled quantities MUST be excluded.
- **FR-006**: A submitted order already in transit MUST be counted once, and received quantities MUST move from on-order to on-hand without increasing their combined total.
- **FR-007**: Case size MUST be positive; invalid or missing settings MUST show an error rather than produce an order.
- **FR-008**: Suggestions MUST show on-hand, on-order, minimum, maximum, case size, cases, units, supplier, and estimated case cost.
- **FR-009**: Case rounding MAY place projected stock above the maximum; the result MUST show that quantity.
- **FR-010**: Suggestions MUST require clerk review and MUST NOT submit supplier orders automatically.

## Key Entities

- **ReorderSetting**: item minimum and maximum within a warehouse.
- **OutstandingOnOrder**: remaining units on submitted supplier orders from Feature 7.
- **PurchaseSuggestion**: calculated values; not a posted inventory change.

## Initial Data Model

### reorderSettings

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique setting ID |
| itemId | INTEGER FK | References items.id |
| warehouseId | INTEGER FK | References warehouses.id |
| minimumUnits | INTEGER | Nonnegative |
| maximumUnits | INTEGER | At least minimumUnits |
| supplierItemId | INTEGER FK | References chosen supplierItems.id |

### calculatedSuggestions

| Field | Type | Rules |
|-------|------|-------|
| usableOnHand, outstandingOnOrder | INTEGER | Current units for item and warehouse |
| shortageUnits | INTEGER | max(0, maximum - onHand - onOrder) |
| suggestedCases | INTEGER | Rounded up to whole cases |
| suggestedUnits | INTEGER | suggestedCases multiplied by supplier case size |
| estimatedCost | DECIMAL(12,2) | suggestedCases multiplied by supplier case cost |

## Gherkin AC

### US-6.1 - Set reorder limits

#### Scenario: Set reorder limits
* **Given** an item has a valid case size
* **When** the manager saves minimum 15 and maximum 50
* **Then** the limits are saved for that item and warehouse

### US-6.2 - Calculate whole-case orders

#### Scenario: Calculate whole-case orders
* **Given** on-hand is 10, minimum is 15, maximum is 50, on-order is 0, and case size is 12
* **When** the clerk runs replenishment planning
* **Then** the suggestion is 4 cases, or 48 units; the projected quantity is 58 after case rounding

#### Scenario: Do not reorder at the minimum
* **Given** on-hand is 15 and minimum is 15
* **When** the clerk calculates replenishment
* **Then** suggested cases are zero

#### Scenario: Round the spreadsheet example to whole cases
* **Given** SKU 240 has 150 on hand, minimum 200, maximum 300, zero on order, and case size 24
* **When** the clerk calculates replenishment
* **Then** the suggestion is 7 cases or 168 units rather than 6.25 cases

### US-6.3 - Allow for stock on order

#### Scenario: Allow for stock on order
* **Given** on-hand is 10, minimum is 15, maximum is 50, on-order is 12, and case size is 12
* **When** the clerk runs replenishment planning
* **Then** the suggestion is 3 cases, or 36 units, with a projected quantity of 58

#### Scenario: Do not reorder stock already covered
* **Given** on-hand is 10, minimum is 15, maximum is 50, and on-order is 48
* **When** the clerk calculates replenishment
* **Then** the suggestion is zero and never negative
