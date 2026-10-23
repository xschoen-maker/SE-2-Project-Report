# Feature: Season Scheduling

## Header

**Feature ID:** 6  
**Branch pattern:** feature/6-season-scheduling  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Generate and maintain a home-and-home soccer schedule using season dates and field availability.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 2 - Sports, Leagues and Seasons](feature-2-sports-leagues-and-seasons.md), [Feature 3 - Team Management](feature-3-team-management.md), [Feature 5 - Location Management](feature-5-location-management.md)  
**Needs clarification:** Confirm kickoff times, game lengths, timezone, and any holidays before publishing. The ten-team fall example supplies Tuesday and Friday dates over nine weeks.

## User Stories

### US-6.1: Generate regular-season games
**As a** supervisor  
**I want to** generate a home-and-home schedule  
**So that** I do not have to build every matchup manually

**Priority:** P1  
**Independent test:** Generate a ten-team season and count games and home/away pairings.  
**Acceptance scenarios:** See US-6.1 under Gherkin AC.

### US-6.2: Review and publish a schedule
**As a** supervisor  
**I want to** review proposed games before making them public  
**So that** teams see an agreed schedule

**Priority:** P1  
**Independent test:** Publish a valid reviewed preview and view it publicly.  
**Acceptance scenarios:** See US-6.2 under Gherkin AC.

### US-6.3: Reschedule a game
**As a** supervisor  
**I want to** change a game's time or field when necessary  
**So that** teams can use an available field and see the updated details

**Priority:** P1  
**Independent test:** Move an unplayed game to an available alternative field.  
**Acceptance scenarios:** See US-6.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Regular-season generation MUST create two games per unordered pair of enrolled soccer teams with reversed home and away roles.
- **FR-002**: For ten teams, a complete schedule MUST contain ninety games and eighteen games per team: nine home and nine away.
- **FR-003**: Games MUST use the selected season dates, regular game days, and confirmed time slots; each team MUST have at most one regular-season game on each selected game date.
- **FR-004**: A home game MUST default to the home team's recorded field; a supervisor MUST be able to save an available alternative field with a reason.
- **FR-005**: Game times MUST have an end after the start; teams and locations MUST not have overlapping games or conflicts with field unavailability.
- **FR-006**: Generation MUST produce a reviewable preview, identify unsatisfied constraints, and require a separate publish action.
- **FR-007**: Regeneration MUST NOT duplicate saved games, erase results, or silently replace a published schedule.
- **FR-008**: Only supervisors MUST create, publish, postpone, or reschedule games; changes MUST record who changed them, when, and the reason.
- **FR-009**: A game change MUST identify affected managers and referee assignments for the notification process in Feature 11.

## Key Entities

- Game: a season matchup with home and away season teams, location, interval, phase, and status.
- GameChange: a record of changed schedule details and the reason.
- SchedulePreview: proposed games checked before publication; not a second official schedule.

## Initial Data Model

### games

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique game ID |
| seasonId | INTEGER FK | References seasons.id |
| homeSeasonTeamId, awaySeasonTeamId | INTEGER FK | References seasonTeams.id; different teams in the same season |
| locationId | INTEGER FK | References locations.id |
| startsAt, endsAt | DATETIME | Required before publication |
| phase | ENUM | regular or tournament |
| status | ENUM | draft, published, postponed, completed, cancelled |

### gameChanges

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique change ID |
| gameId | INTEGER FK | References games.id |
| previousDetails, newDetails | TEXT | Saved time, field, and status changes |
| reason | TEXT | Required |
| changedById | INTEGER FK | References users.id |
| changedAt | DATETIME | Required |

## Gherkin AC

### US-6.1 - Generate regular-season games

#### Scenario: Generate the ten-team fall schedule
* **Given** ten teams are enrolled with home fields and eighteen usable Tuesday and Friday dates over nine weeks, with sufficient game time slots
* **When** the supervisor generates a schedule preview
* **Then** the preview has ninety games, eighteen per team, with every pair meeting once at each team's home field

#### Scenario: Report an unschedulable season
* **Given** the selected dates or available fields cannot fit all required games
* **When** the supervisor generates a preview
* **Then** the conflicts are shown and no incomplete schedule is published

### US-6.2 - Review and publish a schedule

#### Scenario: Publish a valid schedule
* **Given** a complete preview passes team, date, and field checks
* **When** the supervisor selects Publish schedule
* **Then** the games become visible on the public schedule

#### Scenario: Prevent duplicate generation
* **Given** a season already has saved regular-season games
* **When** the supervisor requests another schedule
* **Then** the existing games are shown and a second duplicate set is not silently added

### US-6.3 - Reschedule a game

#### Scenario: Change a game's field
* **Given** an unplayed game is at an unavailable home field and an alternative field is free
* **When** the supervisor saves the alternative location and reason
* **Then** the home and away teams stay the same and the updated location is shown

#### Scenario: Reject a field conflict
* **Given** one game occupies a field from 18:00 to 19:30
* **When** the supervisor moves another game there from 19:00 to 20:30
* **Then** the overlapping change is rejected and the original game is unchanged

## Screen Requirements

### View: Season schedule - route season-schedule
Select league and season. Show date, time, home team, away team, field, and status. Supervisor actions: **Generate schedule**, **Publish schedule**, and **Edit game**. Empty: "No games have been scheduled."

### View: Schedule preview - route schedule-preview
Show the season dates and game days, confirm kickoff times and game lengths, then show proposed games, total count, per-team counts, and conflict list. Publish is unavailable while conflicts remain. Failure: "A complete schedule could not be created. Review the conflicts."

### View: Edit game - route edit-game
Select game time, end time, location, status, and change reason. Show team, field, and referee conflicts before saving. Changed assignments identify who needs a notice.

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

