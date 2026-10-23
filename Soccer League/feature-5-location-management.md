# Feature: Location Management

## Header

**Feature ID:** 5  
**Branch pattern:** feature/5-location-management  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Maintain game fields and unavailable time periods so games are not assigned to unusable locations.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md)  
**Needs clarification:** Confirm which field names, addresses, game durations, and unavailable periods the supervisor wants to use. Treat separate playing fields as separate locations.

## User Stories

### US-5.1: Maintain playing locations
**As a** supervisor  
**I want to** record a field's name and address  
**So that** teams and spectators can find their games

**Priority:** P1  
**Independent test:** Create a field with a name and address.  
**Acceptance scenarios:** See US-5.1 under Gherkin AC.

### US-5.2: Mark a field unavailable
**As a** supervisor  
**I want to** record when a field is damaged or otherwise booked  
**So that** new games are not scheduled there during that time

**Priority:** P1  
**Independent test:** Block a field for one evening and check schedule validation.  
**Acceptance scenarios:** See US-5.2 under Gherkin AC.

### US-5.3: Update location details
**As a** supervisor  
**I want to** correct a field address or retire a field  
**So that** the league stops using incorrect or unavailable locations

**Priority:** P2  
**Independent test:** Retire a field with a future game and check the warning.  
**Acceptance scenarios:** See US-5.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Each location MUST have a required field name, address, and active status.
- **FR-002**: Unavailable periods MUST include location, start, end, and reason; end MUST be after start.
- **FR-003**: New and rescheduled games MUST use active locations and MUST NOT overlap an unavailable period.
- **FR-004**: A location MUST not host overlapping game intervals; intervals that end exactly when another begins MUST not overlap.
- **FR-005**: An unavailable period affecting existing games MUST show those games for supervisor action without automatically relocating them.
- **FR-006**: Locations referenced by teams or game history MUST remain in records; retiring a location MUST not erase earlier games.

## Key Entities

- Location: one playing field and its address.
- LocationUnavailability: a dated interval when a field cannot host a game.

## Initial Data Model

### locations

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique field ID |
| name, address | TEXT | Required |
| isActive | BOOLEAN | Required |

### locationUnavailability

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique block ID |
| locationId | INTEGER FK | References locations.id |
| startsAt, endsAt | DATETIME | Required; end after start |
| reason | TEXT | Required |

## Gherkin AC

### US-5.1 - Maintain playing locations

#### Scenario: Add a playing field
* **Given** a field is not listed
* **When** the supervisor saves its name and address
* **Then** the field is available for home-field selection and game scheduling

### US-5.2 - Mark a field unavailable

#### Scenario: Block an unavailable time
* **Given** a field is active
* **When** the supervisor saves an unavailable period with a reason
* **Then** the field cannot be used for an overlapping new game

#### Scenario: Flag an existing affected game
* **Given** a game is already scheduled at a field
* **When** the supervisor marks that field unavailable during the game's time
* **Then** the affected game is flagged for rescheduling without silently moving it

### US-5.3 - Update location details

#### Scenario: Prevent unnoticed retirement
* **Given** an active field has a future game
* **When** the supervisor attempts to retire the field
* **Then** the future game is identified and retirement waits until its assignment is resolved

## Screen Requirements

### View: Locations - route locations
Supervisor field list with **Add location**, **Edit location**, and **Retire location**. Show address and active status. Empty: "No playing locations have been added."

### View: Field availability - route field-availability
Display selected field's unavailable intervals and affected games. **Add unavailable time** asks for start, end, and reason. Invalid interval shows "End time must be after start time." Conflicts list the affected date and game.

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

