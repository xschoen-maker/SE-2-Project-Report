# Feature: Delivery Confirmation

## Header

**Feature ID:** 14  
**Branch pattern:** feature/14-delivery-confirmation  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Record actual store deliveries, recipient confirmation, and shortages or damage.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 12 - Shipping Verification](feature-12-shipping-verification.md), [Feature 13 - Route and Dispatch Planning](feature-13-route-and-dispatch-planning.md)  
**Sources:** Bill_Of_Lading.docx: delivery date and received by; interview: count delivered products, note damage, and obtain a signature.  
**Draft questions:** Confirm signature method and how rejected or returned goods are handled. This draft records returns in transit; warehouse stock changes only after a separately confirmed return receipt.

## User Stories

### US-14.1: Record a completed delivery
**As a** driver  
**I want to** record what the store received and who accepted it  
**So that** the office has proof of delivery

**Priority:** P1  
**Independent test:** Confirm 12 delivered units with recipient name and time.  
**Acceptance scenarios:** See US-14.1 under Gherkin AC.

### US-14.2: Record damage or a shortage
**As a** driver  
**I want to** note differences between shipped and accepted goods  
**So that** the manager can investigate lost or damaged products

**Priority:** P1  
**Independent test:** Record 10 accepted and 2 damaged from a 12-unit shipment.  
**Acceptance scenarios:** See US-14.2 under Gherkin AC.

### US-14.3: Record a failed delivery
**As a** driver  
**I want to** record why a store could not receive its order  
**So that** dispatch knows which goods still need a decision

**Priority:** P1  
**Independent test:** Mark a delivery failed when the store is closed.  
**Acceptance scenarios:** See US-14.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Delivery records MUST link a departed shipment, run stop, driver, delivery time, and recipient confirmation when accepted.
- **FR-002**: Every line MUST distinguish accepted, damaged, short or missing, and rejected or undelivered units.
- **FR-003**: The total accounted units on a line MUST equal shipped units; unexplained differences MUST be rejected.
- **FR-004**: Damage, shortage, rejection, and failure MUST require a note.
- **FR-005**: A successful delivery MUST save the recipient name and confirmation using the agreed signature method.
- **FR-006**: Failed deliveries MUST remain undelivered and visible to dispatch.
- **FR-007**: Delivery confirmation MUST NOT deduct warehouse stock again after shipment departure.
- **FR-008**: Goods reported as returning MUST remain outside warehouse on-hand until an authorized return receipt confirms their arrival and condition.
- **FR-009**: Corrections MUST preserve the earlier delivery record and who changed it.

## Key Entities

- **DeliveryConfirmation**: outcome of a store visit for a shipment.
- **DeliveryLine**: accepted quantity and discrepancies.
- **RecipientConfirmation**: name and agreed proof of receipt.

## Initial Data Model

### deliveryConfirmations

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique confirmation ID |
| shipmentId | INTEGER FK | References shipments.id |
| stopId | INTEGER FK | References deliveryStops.id |
| driverId | INTEGER FK | References workers.id |
| deliveredAt | DATETIME | Actual attempt time |
| recipientName | STRING | Required for accepted delivery |
| confirmationReference | STRING | Recipient confirmation; exact signature format to confirm |
| outcome | ENUM | delivered, discrepancies, failed |
| note | TEXT | Required when outcome is not delivered |

### deliveryLines

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique line ID |
| confirmationId | INTEGER FK | References deliveryConfirmations.id |
| shipmentLineId | INTEGER FK | References shipmentLines.id |
| acceptedUnits, damagedUnits, missingUnits, undeliveredUnits | INTEGER | Nonnegative; sum equals shipped units |
| note | TEXT | Required for a discrepancy |

## Gherkin AC

### US-14.1 - Record a completed delivery

#### Scenario: Record a completed delivery
* **Given** a departed shipment contains 12 units
* **When** the driver records 12 accepted units, the delivery time, recipient name, and confirmation
* **Then** the shipment is marked delivered with that record

### US-14.2 - Record damage or a shortage

#### Scenario: Record damage or a shortage
* **Given** a shipment contains 12 units
* **When** the driver records 10 accepted units and 2 damaged units with a note
* **Then** the delivery shows the difference and retains the damage note

#### Scenario: Reject unexplained delivery quantities
* **Given** a shipment line has 12 units
* **When** the driver saves 10 accepted units and leaves the other quantities zero
* **Then** the system requests an explanation for the remaining 2 units

### US-14.3 - Record a failed delivery

#### Scenario: Record a failed delivery
* **Given** a truck arrives at a closed store
* **When** the driver records a failed delivery with a reason
* **Then** the shipment remains undelivered and is flagged for office follow-up
