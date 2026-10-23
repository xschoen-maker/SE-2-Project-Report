# Feature: Referee Management

## Header

**Feature ID:** 8  
**Branch pattern:** feature/8-referee-management  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Keep referee contacts and sport-specific qualifications for suitable game assignments.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 2 - Sports, Leagues and Seasons](feature-2-sports-leagues-and-seasons.md)  
**Needs clarification:** Confirm required officiating positions and qualification labels for each sport. Soccer needs center and assistant referee (AR) qualifications; the number of each per game is not specified.

## User Stories

### US-8.1: Maintain referee contacts
**As a** supervisor  
**I want to** add and update referee names and contact email addresses  
**So that** I can find officials and contact them about games

**Priority:** P1  
**Independent test:** Save a referee and locate that record.  
**Acceptance scenarios:** See US-8.1 under Gherkin AC.

### US-8.2: Record referee qualifications
**As a** supervisor  
**I want to** record which sports and officiating roles a referee can cover  
**So that** games can be assigned to qualified officials

**Priority:** P1  
**Independent test:** Mark one referee as qualified for soccer AR but not center.  
**Acceptance scenarios:** See US-8.2 under Gherkin AC.

### US-8.3: Maintain referee availability
**As a** supervisor  
**I want to** record unavailable times and inactive referees  
**So that** I do not assign someone who cannot attend

**Priority:** P1  
**Independent test:** Save an unavailable interval and check it during assignment.  
**Acceptance scenarios:** See US-8.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Referee records MUST have a required name, valid contact email, and active status; contact details MUST remain private.
- **FR-002**: Each qualification MUST identify referee, sport, and role; duplicate referee/sport/role qualifications MUST be rejected.
- **FR-003**: Soccer qualifications MUST distinguish center and AR; an AR qualification MUST NOT imply center qualification.
- **FR-004**: Unavailable periods MUST have a start before the end and a reason.
- **FR-005**: Inactive or unavailable referees MUST not receive new conflicting assignments.
- **FR-006**: Retiring a referee MUST retain earlier game assignments and identify future assignments needing replacement.
- **FR-007**: Referee records MUST NOT create editing accounts or allow referees to change league data.

## Key Entities

- Referee: an official with private contact details.
- RefereeQualification: a sport and role an official can perform.
- RefereeUnavailability: a period when an official cannot attend.

## Initial Data Model

### referees

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique referee ID |
| name, email | TEXT | Required; email private |
| isActive | BOOLEAN | Required |

### refereeQualifications

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique qualification |
| refereeId, sportId | INTEGER FK | Reference referees.id and sports.id |
| role | TEXT | Confirmed role; soccer center or AR; unique referee/sport/role |

### refereeUnavailability

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique interval |
| refereeId | INTEGER FK | References referees.id |
| startsAt, endsAt | DATETIME | End after start |
| reason | TEXT | Required |

## Gherkin AC

### US-8.1 - Maintain referee contacts

#### Scenario: Add a referee
* **Given** a referee is not listed
* **When** the supervisor saves the referee's name and valid email address
* **Then** the referee appears in the private referee directory

### US-8.2 - Record referee qualifications

#### Scenario: Record an AR qualification
* **Given** a referee is in the directory
* **When** the supervisor adds a soccer assistant referee qualification
* **Then** the referee is eligible for soccer AR assignments but not center assignments

### US-8.3 - Maintain referee availability

#### Scenario: Record an unavailable period
* **Given** a referee cannot work on a selected evening
* **When** the supervisor saves the start, end, and reason
* **Then** that interval is shown during referee assignment

#### Scenario: Reject a reversed availability interval
* **Given** the supervisor is editing referee availability
* **When** an end before the start is saved
* **Then** the period is rejected without changing availability

## Screen Requirements

### View: Referee directory - route referees
Supervisor-only list with name, email, qualifications, and status. **Add referee**, **Edit referee**, and **Mark inactive**. Empty: "No referees have been added."

### View: Referee details - route referee-details
Manage sport/role qualifications and unavailable periods; show future assignments needing attention. Invalid email shows "Enter a valid email address." Invalid interval shows "End time must be after start time."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

