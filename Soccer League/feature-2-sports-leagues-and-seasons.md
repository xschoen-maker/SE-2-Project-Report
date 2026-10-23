# Feature: Sports, Leagues and Seasons

## Header

**Feature ID:** 2  
**Branch pattern:** feature/2-sports-leagues-and-seasons  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Maintain the sports, leagues, and seasons used to organize teams and games.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md)  
**Needs clarification:** Confirm the season's exact start date, game times, standings points, and tiebreaker order. Other sports are recorded, but their scheduling and scoring rules need separate confirmation.

## User Stories

### US-2.1: Maintain sports
**As a** supervisor  
**I want to** keep a list of sports  
**So that** each league is associated with the right sport

**Priority:** P1  
**Independent test:** Save soccer and flag football as separate sports.  
**Acceptance scenarios:** See US-2.1 under Gherkin AC.

### US-2.2: Maintain leagues
**As a** supervisor  
**I want to** create and update leagues for each sport  
**So that** teams and seasons have a clear league

**Priority:** P1  
**Independent test:** Create a soccer league and view its sport.  
**Acceptance scenarios:** See US-2.2 under Gherkin AC.

### US-2.3: Set up a season
**As a** supervisor  
**I want to** record season dates and regular game days  
**So that** the schedule can use the correct season settings

**Priority:** P1  
**Independent test:** Save a nine-week fall season with Tuesday and Friday game days.  
**Acceptance scenarios:** See US-2.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: A league MUST belong to one sport; a season MUST belong to one league.
- **FR-002**: Sport and league names MUST be required; season names MUST be unique within their league.
- **FR-003**: Seasons MUST store start and end dates and regular game days; the end date MUST not precede the start date.
- **FR-004**: The fall soccer example MUST support nine weeks and Tuesday and Friday game dates without treating those settings as rules for every sport or season.
- **FR-005**: Standings points and tiebreaker rules MUST be recorded only after confirmation; the system MUST identify missing rules before ranking tied teams or selecting tournament qualifiers.
- **FR-006**: Changing season settings MUST NOT silently replace an existing game schedule.
- **FR-007**: Records referenced by teams, games, or results MUST be retained; inactive status MUST be available instead of removing history.

## Key Entities

- Sport: a sport such as soccer, basketball, softball, or flag football.
- League: a competition for one sport.
- Season: a league's dated competition and scheduling settings.

## Initial Data Model

### sports

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique sport ID |
| name | TEXT | Required; unique |
| isActive | BOOLEAN | Required |

### leagues

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique league ID |
| sportId | INTEGER FK | References sports.id |
| name | TEXT | Required |
| isActive | BOOLEAN | Required |

### seasons

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique season ID |
| leagueId | INTEGER FK | References leagues.id |
| name | TEXT | Required; unique within league |
| startDate, endDate | DATE | Required; end not before start |
| gameDays | DAY LIST | Selected regular game days |
| winPoints, drawPoints, lossPoints | DECIMAL | Needs clarification before points-based standings |
| tiebreakerRules | TEXT | Confirmed ordered rules; unresolved ties flagged |
| status | ENUM | draft, active, completed |

## Gherkin AC

### US-2.1 - Maintain sports

#### Scenario: Add a sport
* **Given** soccer is not listed
* **When** the supervisor saves Soccer
* **Then** the sport becomes available when creating a league

### US-2.2 - Maintain leagues

#### Scenario: Create a league
* **Given** Soccer exists
* **When** the supervisor saves a league name and selects Soccer
* **Then** the league appears under Soccer

### US-2.3 - Set up a season

#### Scenario: Save the fall season
* **Given** a soccer league exists
* **When** the supervisor saves a named nine-week season with valid start and end dates and Tuesday and Friday game days
* **Then** the season is linked to that league with those settings

#### Scenario: Reject reversed dates
* **Given** a supervisor is editing a season
* **When** the supervisor enters an end date before the start date
* **Then** the season is not saved and the date error is shown

## Screen Requirements

### View: Sports and leagues - route league-setup
Supervisor-only sport and league lists with **Add sport**, **Add league**, and **Edit**. League form selects sport. Empty: "No leagues have been added."

### View: Seasons - route seasons
Filter by league. **Add season** and **Edit season** show name, dates, game days, status, and confirmed standings rules. Show "End date must be on or after start date." for reversed dates. Empty: "No seasons for this league."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

