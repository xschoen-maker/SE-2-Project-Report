# Feature: Bus Route Management

## Header

**Feature ID:** 6  
**Branch pattern:** feature/6-bus-route-management  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Organize expanding bus routes with assigned drivers, ordered pickup stops, and riders.  
**Depends on:** [Feature 1 - Member Record Management](feature-1-member-record-management.md)

## User Stories

### US-6.1: Create a bus route
**As a** transportation coordinator  
**I want to** save a route and its assigned driver  
**So that** the church can add routes as it expands

**Priority:** P1  
**Independent test:** Create two routes and verify their driver assignments.  
**Acceptance scenarios:** See US-6.1 under Gherkin AC.

### US-6.2: Organize stops and riders
**As a** transportation coordinator  
**I want to** order pickup stops and assign people to them  
**So that** drivers know where each rider needs pickup

**Priority:** P1  
**Independent test:** Add two ordered stops and assign Jordan to one.  
**Acceptance scenarios:** See US-6.2 under Gherkin AC.

### US-6.3: Update route arrangements
**As a** transportation coordinator  
**I want to** change the driver or rider assignments  
**So that** route information stays current

**Priority:** P1  
**Independent test:** Replace a driver and reopen the route.  
**Acceptance scenarios:** See US-6.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Staff MUST be able to create multiple routes without a fixed route limit.
- **FR-002**: A route MUST have a nonempty name, active status, and an optional driver referencing a person.
- **FR-003**: Routes without drivers MUST be visibly marked unassigned.
- **FR-004**: Each stop MUST belong to one route and have a nonempty address and positive position unique within that route.
- **FR-005**: Stops MUST be shown in position order.
- **FR-006**: Rider assignments MUST reference existing people and stops; a person MUST NOT have multiple stops on one route.
- **FR-007**: Staff MUST be able to change drivers, move riders, and remove assignments without deleting people.
- **FR-008**: An occupied stop MUST NOT be removed until riders are moved or unassigned.
- **FR-009**: Inactive routes MUST keep stops and assignments.

## Initial Data Model

A route has pickup stops in order and can have an assigned driver. Each rider is linked to a pickup stop.

### routes

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique route ID |
| name | STRING | Required; cannot be blank |
| driverId | INTEGER FK | Optional; references people.id |
| isActive | BOOLEAN | Defaults to true |

### routeStops

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique stop ID |
| routeId | INTEGER FK | Required; references routes.id |
| pickupAddress | STRING | Required; cannot be blank |
| position | INTEGER | Positive; unique within route |

### riderAssignments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique assignment ID |
| personId | INTEGER FK | Required; references people.id |
| stopId | INTEGER FK | Required; references routeStops.id |

A person can have only one assigned stop within each route.

## Gherkin AC

### US-6.1 - Create a bus route

#### Scenario: Create a bus route
* **Given** a driver has a person record
* **When** I create a named route and assign that driver
* **Then** the route appears with the assigned driver

#### Scenario: Create a route without a driver
* **Given** the church is arranging a new route
* **When** I save a named route without assigning a driver
* **Then** it appears with its driver marked unassigned

### US-6.2 - Organize stops and riders

#### Scenario: Organize stops and riders
* **Given** a route and Jordan exist
* **When** I add ordered pickup addresses and assign Jordan to a stop
* **Then** the route lists the stops in order and Jordan at the selected stop

#### Scenario: Protect an occupied stop
* **Given** Jordan is assigned to a pickup stop
* **When** I remove the stop without moving or unassigning Jordan
* **Then** the system blocks removal and explains that riders are still assigned

#### Scenario: Assign a rider to a second stop on the same route
* **Given** Jordan is assigned to the first stop on a route
* **When** I try to add Jordan to another stop on that route without moving the existing assignment
* **Then** the system rejects the duplicate assignment
* **And** Jordan remains assigned only to the first stop

### US-6.3 - Update route arrangements

#### Scenario: Update route arrangements
* **Given** a route has a driver, stops, and riders
* **When** I assign a replacement driver
* **Then** the replacement is shown and existing stops and riders remain
