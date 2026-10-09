# Feature: Operational Status Management

## Header

**Feature ID:** 7  
**Branch pattern:** feature/7-operational-status-management  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Record problems affecting classes or bus routes and follow each problem until it is resolved.  
**Depends on:** [Feature 3 - Church Class Management](feature-3-church-class-management.md), [Feature 6 - Bus Route Management](feature-6-bus-route-management.md)

## User Stories

### US-7.1: Report a problem
**As a** transportation coordinator  
**I want to** report that a driver is sick for a route  
**So that** staff can see what needs attention

**Priority:** P1  
**Independent test:** Report a sick driver for a route and check that the issue is saved.  
**Acceptance scenarios:** See US-7.1 under Gherkin AC.

### US-7.2: Update problem status
**As a** church staff member  
**I want to** mark an issue in progress and add an update  
**So that** others know what is being handled

**Priority:** P1  
**Independent test:** Move an issue to in progress and reopen its history.  
**Acceptance scenarios:** See US-7.2 under Gherkin AC.

### US-7.3: Resolve a problem
**As a** transportation coordinator  
**I want to** close an issue with a resolution note  
**So that** staff know the problem is fixed

**Priority:** P1  
**Independent test:** Assign a replacement driver and close the issue with a note.  
**Acceptance scenarios:** See US-7.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: An issue MUST have a nonempty description and link to exactly one existing class or route.
- **FR-002**: New issues MUST start open and record the report time.
- **FR-003**: Status MUST be open, in progress, or resolved.
- **FR-004**: Each status change MUST keep its date, new status, and not blank update note.
- **FR-005**: Staff MUST be able to view unresolved issues and filter by class or route.
- **FR-006**: Closing an issue MUST require a note explaining how it was resolved.
- **FR-007**: Issue updates MUST keep earlier updates.
- **FR-008**: Reporting or resolving an issue MUST NOT automatically change drivers, schedules, or attendance; staff must make those changes separately.

## Initial Data Model

Each issue is linked to a class or route. Its updates show what happened and when.

### operationalIssues

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique issue ID |
| classId | INTEGER FK | Optional; references classes.id |
| routeId | INTEGER FK | Optional; references routes.id |
| description | TEXT | Required; cannot be blank |
| status | ENUM | open, in progress, resolved; defaults to open |
| reportedAt, updatedAt | DATETIME | Report time and latest update |

Exactly one of classId and routeId must be set.

### issueUpdates

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique update ID |
| issueId | INTEGER FK | Required; references operationalIssues.id |
| status | ENUM | Status recorded by this update |
| note | TEXT | Required; includes resolution explanation when resolved |
| createdAt | DATETIME | Update time |

## Gherkin AC

### US-7.1 - Report a problem

#### Scenario: Report a problem
* **Given** a route driver is sick
* **When** I report the problem for the route
* **Then** an open issue shows the route, description, and report time

#### Scenario: Report a sick driver
* **Given** a route has an assigned driver
* **When** I report that the driver is sick
* **Then** the issue appears as open
* **And** the route's driver assignment stays unchanged until staff update it

### US-7.2 - Update problem status

#### Scenario: Update problem status
* **Given** a class or route has an open issue
* **When** I mark it in progress and add a note
* **Then** the current status and dated note are visible

#### Scenario: Filter unresolved route issues
* **Given** a route has one open issue and one resolved issue
* **And** a class also has an open issue
* **When** I filter unresolved issues for the route
* **Then** only the route's open issue appears

### US-7.3 - Resolve a problem

#### Scenario: Resolve a problem
* **Given** a replacement driver has been assigned and the issue is in progress
* **When** I resolve the issue with a note about the arrangement
* **Then** it shows resolved and keeps the resolution note and earlier updates

#### Scenario: Require a resolution note
* **Given** an issue is open
* **When** I resolve it without a note
* **Then** the system requests a resolution note and leaves the issue open
