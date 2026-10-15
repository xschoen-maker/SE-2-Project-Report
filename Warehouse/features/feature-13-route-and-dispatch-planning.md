# Feature: Route and Dispatch Planning

## Header

**Feature ID:** 13  
**Branch pattern:** feature/13-route-and-dispatch-planning  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Organize daily delivery runs, trucks, drivers, store stops, and shipment manifests.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 4 - Supplier and Customer Management](feature-4-supplier-and-customer-management.md)  
**Sources:** Presentation slide 4: Routes; interview: about 40 stores per day, 10 trucks, 5-8 routes, and routes that differ by day.  
**Draft questions:** Confirm whether 5-8 routes means daily runs or fixed route templates, and which truck capacities the manager wants checked.

## User Stories

### US-13.1: Plan a day's route
**As a** dispatcher  
**I want to** create ordered store stops for a delivery date  
**So that** drivers know which stores to visit

**Priority:** P1  
**Independent test:** Create a run for two stores and check their stop order.  
**Acceptance scenarios:** See US-13.1 under Gherkin AC.

### US-13.2: Assign truck and driver
**As a** dispatcher  
**I want to** assign an available truck and driver  
**So that** each run has someone and a vehicle to carry the goods

**Priority:** P1  
**Independent test:** Assign a driver and truck to a dated run.  
**Acceptance scenarios:** See US-13.2 under Gherkin AC.

### US-13.3: Build a manifest
**As a** shipping clerk  
**I want to** assign shipments to the right store stop and view the load  
**So that** the driver takes the right goods on the right route

**Priority:** P1  
**Independent test:** Assign two store shipments and view the manifest.  
**Acceptance scenarios:** See US-13.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Delivery runs MUST have a unique reference, date, ordered stops, and status.
- **FR-002**: Each stop MUST reference an active customer and snapshot its delivery address.
- **FR-003**: Stops MUST have a positive sequence unique within a run.
- **FR-004**: Runs MUST use active worker driver records and active trucks.
- **FR-005**: A driver or truck MUST NOT be assigned to overlapping runs; start and end times MUST support that check.
- **FR-006**: Shipments MUST be assigned to the stop for their actual customer and MUST NOT appear on two runs.
- **FR-007**: The manifest MUST list driver, truck, date, store stops, shipment references, and loaded item quantities.
- **FR-008**: Changing a driver or truck before departure MUST preserve the run's stop and shipment links.
- **FR-009**: The numbers in the interview MUST be treated as current operating volumes, not hardcoded limits.
- **FR-010**: Automatic route optimization and capacity checking MUST remain outside this draft until constraints are confirmed.

## Key Entities

- **Truck**: vehicle available for delivery work.
- **DeliveryRun**: one day's planned trip.
- **DeliveryStop**: ordered customer visit.
- **RunShipment**: assignment of a shipment to a run stop.

## Initial Data Model

### trucks

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique truck ID |
| vehicleNumber | STRING | Required; unique |
| isActive | BOOLEAN | Defaults to true |

### deliveryRuns

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique run ID |
| reference | STRING | Required; unique |
| deliveryDate | DATE | Required |
| startsAt, endsAt | DATETIME | Planned interval; end after start |
| driverId | INTEGER FK | References workers.id |
| truckId | INTEGER FK | References trucks.id |
| status | ENUM | planned, loaded, departed, completed |

### deliveryStops

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique stop ID |
| runId | INTEGER FK | References deliveryRuns.id |
| customerId | INTEGER FK | References customers.id |
| sequence | INTEGER | Positive; unique within run |
| deliveryAddress | STRING | Required snapshot |

### runShipments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique assignment ID |
| stopId | INTEGER FK | References deliveryStops.id |
| shipmentId | INTEGER FK | References shipments.id; unique |

## Gherkin AC

### US-13.1 - Plan a day's route

#### Scenario: Plan a day's route
* **Given** two active customer stores exist
* **When** the dispatcher saves a dated route with those stores in order
* **Then** the route shows the date, stop sequence, and store addresses

### US-13.2 - Assign truck and driver

#### Scenario: Assign truck and driver
* **Given** an active driver and truck are available
* **When** the dispatcher assigns them to a run
* **Then** the run shows the driver and truck

#### Scenario: Prevent overlapping truck assignments
* **Given** truck T1 is assigned from 08:00 to 12:00
* **When** the dispatcher assigns T1 to another run from 10:00 to 13:00
* **Then** the overlapping assignment is rejected

### US-13.3 - Build a manifest

#### Scenario: Build a manifest
* **Given** verified shipments exist for the route's two stores
* **When** the clerk assigns them to their store stops
* **Then** the manifest lists both shipments grouped in stop order
