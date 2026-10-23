# Feature: Season Tournament

## Header

**Feature ID:** 12  
**Branch pattern:** feature/12-season-tournament  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Organize an end-of-season tournament for the top four teams.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 2 - Sports, Leagues and Seasons](feature-2-sports-leagues-and-seasons.md), [Feature 3 - Team Management](feature-3-team-management.md), [Feature 6 - Season Scheduling](feature-6-season-scheduling.md), [Feature 7 - Scores and Statistics](feature-7-scores-and-statistics.md)  
**Needs clarification:** Confirm standings points, tied qualification rules, seeding and semifinal pairings, and how drawn knockout games are decided. No automatic coin toss, penalty procedure, or third-place game is assumed.

## User Stories

### US-12.1: Select the top four teams
**As a** supervisor  
**I want to** review the regular-season standings and confirm four qualifiers  
**So that** the tournament includes the correct teams

**Priority:** P1  
**Independent test:** Select four qualifiers from confirmed final standings.  
**Acceptance scenarios:** See US-12.1 under Gherkin AC.

### US-12.2: Schedule tournament games
**As a** supervisor  
**I want to** set semifinal matchups, dates, and locations  
**So that** the four teams know where and when to play

**Priority:** P1  
**Independent test:** Schedule two semifinals for the four confirmed teams.  
**Acceptance scenarios:** See US-12.2 under Gherkin AC.

### US-12.3: Advance winners to the final
**As a** supervisor  
**I want to** record semifinal winners and the final result  
**So that** the tournament has a clear champion

**Priority:** P1  
**Independent test:** Confirm both semifinal winners and place them in a final.  
**Acceptance scenarios:** See US-12.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: The tournament MUST belong to one season and contain exactly four distinct qualifying season teams.
- **FR-002**: Qualification MUST use official regular-season results and agreed standings rules; unresolved ties MUST require an agreed supervisor decision before confirming qualifiers.
- **FR-003**: Semifinal pairings MUST be entered or confirmed by the supervisor; unprovided seeding rules MUST not be assumed.
- **FR-004**: Tournament games MUST be stored as tournament-phase games and MUST use the same field, team, time, and referee checks as other games.
- **FR-005**: Each qualifier MUST appear in one semifinal; only confirmed semifinal winners MUST enter the final.
- **FR-006**: A drawn knockout score MUST not determine an advancing team without an agreed deciding result; that decision MUST be recorded.
- **FR-007**: The final's confirmed winner MUST identify the champion; tournament results MUST not alter regular-season standings.
- **FR-008**: Changing an earlier confirmed winner MUST flag dependent games for supervisor review rather than silently changing completed games.

## Key Entities

- Tournament: the season's four-team event.
- TournamentEntry: one qualifying team and its confirmed qualification.
- TournamentMatch: semifinal or final linked to a game and its confirmed advancing team.

## Initial Data Model

### tournaments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique tournament ID |
| seasonId | INTEGER FK | References seasons.id; one tournament per season |
| status | ENUM | draft, confirmed, completed |

### tournamentEntries

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique qualifier ID |
| tournamentId | INTEGER FK | References tournaments.id |
| seasonTeamId | INTEGER FK | References seasonTeams.id; unique within tournament |
| qualificationNote | TEXT | Confirmed standing or agreed tie decision |

### tournamentMatches

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique tournament match |
| tournamentId | INTEGER FK | References tournaments.id |
| gameId | INTEGER FK | References games.id; unique; tournament phase |
| round | ENUM | semifinal or final |
| advancingSeasonTeamId | INTEGER FK | Optional; references a participating seasonTeams.id |
| decisionNote | TEXT | Required when a deciding procedure is needed |

## Gherkin AC

### US-12.1 - Select the top four teams

#### Scenario: Confirm four qualifiers
* **Given** the regular season is complete and agreed standings rules identify four qualifying teams
* **When** the supervisor reviews and confirms those teams
* **Then** the tournament records exactly those four season teams

#### Scenario: Flag an unresolved qualification tie
* **Given** two teams are tied for the fourth place and no agreed tiebreaker resolves them
* **When** the supervisor prepares the qualifiers
* **Then** the tie is shown and qualifiers cannot be finalized using an invented order

### US-12.2 - Schedule tournament games

#### Scenario: Schedule the semifinals
* **Given** four qualifiers and approved pairings are recorded
* **When** the supervisor saves two semifinal games in available slots
* **Then** each qualifier appears in one semifinal and the games use the regular conflict checks

### US-12.3 - Advance winners to the final

#### Scenario: Create the final from semifinal winners
* **Given** both semifinals have confirmed advancing teams
* **When** the supervisor schedules the final
* **Then** only those two winners are entered in the final

#### Scenario: Do not advance a tied score automatically
* **Given** a semifinal's reported score is tied and its winner has not been confirmed under an agreed rule
* **When** the supervisor attempts to advance a team
* **Then** advancement is blocked until the winner and decision are recorded

## Screen Requirements

### View: Tournament setup - route tournament-setup
Supervisor view shows season standings, unresolved ties, four qualifiers, and approved pairings. **Confirm qualifiers** is unavailable until qualification is resolved. Empty: "No tournament has been set up."

### View: Tournament bracket - route tournament-bracket
Show two semifinals and one final with teams, date, field, officials, official score, and advancing team. Supervisor actions **Schedule match** and **Confirm winner**; public visitors have read-only access to published matches. Pending: "Winner not confirmed."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

