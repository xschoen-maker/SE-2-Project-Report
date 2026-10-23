# Feature: Team Management

## Header

**Feature ID:** 3  
**Branch pattern:** feature/3-team-management  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Maintain teams, enroll them in seasons, assign managers and home fields, and record team check payments.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 2 - Sports, Leagues and Seasons](feature-2-sports-leagues-and-seasons.md), [Feature 5 - Location Management](feature-5-location-management.md)  
**Needs clarification:** Confirm team fee amounts and whether payment status affects participation; do not block a team solely for an unpaid check without an agreed rule.

## User Stories

### US-3.1: Maintain a team
**As a** supervisor  
**I want to** add and update team information  
**So that** the league has an accurate team list

**Priority:** P1  
**Independent test:** Save a team with its name and sport.  
**Acceptance scenarios:** See US-3.1 under Gherkin AC.

### US-3.2: Enroll a team for a season
**As a** supervisor  
**I want to** assign a team, manager, and home field to a season  
**So that** scheduling and manager access use the correct records

**Priority:** P1  
**Independent test:** Enroll a soccer team in a soccer season and assign an active manager and field.  
**Acceptance scenarios:** See US-3.2 under Gherkin AC.

### US-3.3: Record a team payment
**As a** supervisor  
**I want to** record a check received for a season team  
**So that** I can see which team fees have been received

**Priority:** P2  
**Independent test:** Save a check amount and received date for one season team.  
**Acceptance scenarios:** See US-3.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Teams MUST have a required name, sport, and active status.
- **FR-002**: Each season enrollment MUST link one team, one season, an active manager account, and a home location.
- **FR-003**: A team MUST not be enrolled twice in the same season or in a season for a different sport.
- **FR-004**: Only supervisors MUST change season enrollment, manager assignments, and home locations.
- **FR-005**: A manager MUST gain access only to currently assigned teams; changing an assignment MUST update access without changing roster history.
- **FR-006**: Check records MUST include season team, positive amount, received date, and reference.
- **FR-007**: Team payments MUST be recordkeeping only; card processing, check processing, and automatic eligibility restrictions are not specified.
- **FR-008**: Removing a team or enrollment referenced by games or results MUST be blocked; history MUST remain available.

## Key Entities

- Team: a named team participating in a sport.
- SeasonTeam: a team's entry in a season, linked to its manager and home location.
- TeamPayment: a received check recorded against one season team.

## Initial Data Model

### teams

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique team ID |
| sportId | INTEGER FK | References sports.id |
| name | TEXT | Required |
| isActive | BOOLEAN | Required |

### seasonTeams

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique enrollment ID |
| seasonId | INTEGER FK | References seasons.id |
| teamId | INTEGER FK | References teams.id; unique pair with seasonId |
| managerId | INTEGER FK | References users.id; active manager |
| homeLocationId | INTEGER FK | References locations.id |
| isActive | BOOLEAN | Required |

### teamPayments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique payment ID |
| seasonTeamId | INTEGER FK | References seasonTeams.id |
| amount | DECIMAL(12,2) | Positive check amount |
| checkReference | TEXT | Required |
| receivedDate | DATE | Required |

## Gherkin AC

### US-3.1 - Maintain a team

#### Scenario: Add a team
* **Given** a sport exists
* **When** the supervisor saves a new team name and sport
* **Then** the team appears in the team list

### US-3.2 - Enroll a team for a season

#### Scenario: Enroll a season team
* **Given** a soccer season, soccer team, manager, and location exist
* **When** the supervisor enrolls the team with that manager and home field
* **Then** the team appears once in the season and its manager can manage its roster

#### Scenario: Reject duplicate enrollment
* **Given** a team is already enrolled in a season
* **When** the supervisor enrolls the same team again
* **Then** the duplicate is rejected without adding another entry

### US-3.3 - Record a team payment

#### Scenario: Record a received check
* **Given** a team is enrolled in a season
* **When** the supervisor records a positive check amount, check reference, and received date
* **Then** the payment appears against that season team without initiating an online payment

## Screen Requirements

### View: Teams - route teams
Supervisor view with sport filter and **Add team**, **Edit team**, and active status. Empty: "No teams have been added."

### View: Season teams - route season-teams
Select league and season; show enrolled teams, manager, home field, and check records. **Enroll team**, **Edit enrollment**, and **Record check** are supervisor-only. Payment dialog shows amount, check reference, and date. Empty: "No teams enrolled in this season."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

