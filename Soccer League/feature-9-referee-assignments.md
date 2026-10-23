# Feature: Referee Assignments

## Header

**Feature ID:** 9  
**Branch pattern:** feature/9-referee-assignments  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Assign qualified, available officials to games and track replacements when someone cannot attend.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 6 - Season Scheduling](feature-6-season-scheduling.md), [Feature 8 - Referee Management](feature-8-referee-management.md)  
**Needs clarification:** Confirm the number of center and AR positions required for a soccer game and how referee acceptance is reported. Referee replies can be recorded by the supervisor; a referee login is not assumed.

## User Stories

### US-9.1: Assign game officials
**As a** supervisor  
**I want to** choose a qualified referee for each required game position  
**So that** games have suitable officials

**Priority:** P1  
**Independent test:** Assign a center-qualified referee to a center position.  
**Acceptance scenarios:** See US-9.1 under Gherkin AC.

### US-9.2: Check assignment conflicts
**As a** supervisor  
**I want to** see when a referee is already booked or unavailable  
**So that** one person is not assigned to overlapping games

**Priority:** P1  
**Independent test:** Assign an official to an overlapping second game.  
**Acceptance scenarios:** See US-9.2 under Gherkin AC.

### US-9.3: Replace an unavailable referee
**As a** supervisor  
**I want to** record a declined assignment and select a replacement  
**So that** an uncovered game is visible until a qualified replacement is found

**Priority:** P1  
**Independent test:** Record a decline and fill the reopened role.  
**Acceptance scenarios:** See US-9.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Only supervisors MUST create or replace referee assignments.
- **FR-002**: Each assignment MUST link one game, required role, referee, and assignment status.
- **FR-003**: Assignment MUST check sport-specific qualification, active status, unavailable periods, and overlapping game intervals.
- **FR-004**: One referee MUST not fill two simultaneous positions in the same game; each required position MUST have at most one current referee.
- **FR-005**: Games MUST show unfilled or declined positions until an eligible replacement is assigned.
- **FR-006**: Declined and replaced assignments MUST be retained with a reason and time rather than overwritten.
- **FR-007**: Rescheduling a game MUST recheck its referees and flag affected assignments for review and notification.
- **FR-008**: Required position counts MUST use confirmed season rules rather than assuming a fixed crew for all sports.

## Key Entities

- RequiredGameRole: one officiating position needed for a game.
- RefereeAssignment: current or earlier assignment of an official to a required position.

## Initial Data Model

### requiredGameRoles

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique position ID |
| gameId | INTEGER FK | References games.id |
| role | TEXT | Confirmed sport role |
| positionNumber | INTEGER | Positive; unique game/role/position |

### refereeAssignments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique assignment ID |
| requiredRoleId | INTEGER FK | References requiredGameRoles.id |
| refereeId | INTEGER FK | References referees.id |
| status | ENUM | assigned, accepted, declined, replaced |
| assignedAt | DATETIME | Required |
| responseAt | DATETIME | Optional |
| reason | TEXT | Required for decline or replacement |

## Gherkin AC

### US-9.1 - Assign game officials

#### Scenario: Assign a qualified center referee
* **Given** a published soccer game needs a center and a center-qualified referee is free
* **When** the supervisor assigns that referee
* **Then** the center assignment is saved for the game

#### Scenario: Reject an unqualified official
* **Given** a referee is qualified only as a soccer AR
* **When** the supervisor assigns the referee as center
* **Then** the assignment is rejected

### US-9.2 - Check assignment conflicts

#### Scenario: Reject overlapping referee assignments
* **Given** a referee is assigned from 18:00 to 19:30
* **When** the supervisor assigns the same referee to another game from 19:00 to 20:30
* **Then** the conflicting assignment is rejected

### US-9.3 - Replace an unavailable referee

#### Scenario: Replace a declined assignment
* **Given** an assigned referee reports that they cannot attend
* **When** the supervisor records the decline and assigns an available qualified replacement
* **Then** the earlier assignment remains in history, the replacement is current, and the game is marked for a notification

## Screen Requirements

### View: Game officials - route game-officials
Show game teams, field, interval, required positions, and assignment status. **Assign referee** filters by sport qualification and availability. Empty positions show "Referee needed."

### View: Replace referee - route replace-referee
Supervisor form records reason, available candidates, and replacement. Reject conflicts with "This referee is unavailable for the selected game." Show a notice-needed indicator after replacement or rescheduling.

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

