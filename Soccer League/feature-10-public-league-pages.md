# Feature: Public League Pages

## Header

**Feature ID:** 10  
**Branch pattern:** feature/10-public-league-pages  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Publish league, season, team, player, schedule, result, and statistics information for public viewing.  
**Depends on:** [Feature 2 - Sports, Leagues and Seasons](feature-2-sports-leagues-and-seasons.md), [Feature 3 - Team Management](feature-3-team-management.md), [Feature 4 - Player Rosters](feature-4-player-rosters.md), [Feature 6 - Season Scheduling](feature-6-season-scheduling.md), [Feature 7 - Scores and Statistics](feature-7-scores-and-statistics.md)  
**Needs clarification:** Confirm the standings points and tiebreakers before displaying a ranked table. Only agreed public player details are shown; contact details and account information stay private.

## User Stories

### US-10.1: Browse the league
**As a** public visitor  
**I want to** choose a sport, league, and season  
**So that** I can find the competition my family follows

**Priority:** P1  
**Independent test:** Open a season without signing in.  
**Acceptance scenarios:** See US-10.1 under Gherkin AC.

### US-10.2: View teams and players
**As a** public visitor  
**I want to** see team rosters and published player statistics  
**So that** I can follow players and their teams

**Priority:** P1  
**Independent test:** Open a team roster and a player's confirmed statistics.  
**Acceptance scenarios:** See US-10.2 under Gherkin AC.

### US-10.3: View schedules and records
**As a** public visitor  
**I want to** see game details, official scores, and team records  
**So that** I know when games occur and how teams are doing

**Priority:** P1  
**Independent test:** View a published reschedule and an official completed result.  
**Acceptance scenarios:** See US-10.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Public viewing MUST not require sign-in and MUST offer navigation from sport to league, season, team, and player.
- **FR-002**: Public schedules MUST show published game date, time, location and address, teams, and current status; unpublished previews MUST remain private.
- **FR-003**: Team pages MUST show season roster and approved player details; player pages MUST show confirmed game statistics and totals.
- **FR-004**: Official team records MUST use confirmed regular-season results; tournament games MUST remain separate.
- **FR-005**: Ranked standings MUST use confirmed season points and tiebreakers; unresolved rules or ties MUST be identified instead of inventing a rank.
- **FR-006**: Public pages MUST NOT expose user credentials, player email, referee contact email, private messages, or payment records.
- **FR-007**: All public screens and direct data requests MUST be read-only; managers MUST sign in to access editing.
- **FR-008**: Empty or pending results MUST be labeled clearly rather than displayed as a played 0-0 game.

## Key Entities

- PublicLeagueView: read-only display derived from league and season records.
- TeamRecord: calculated wins, draws, losses, and configured points from official regular-season results.
- PlayerTotals: calculated confirmed stats for a selected season; not duplicate manually maintained values.

## Initial Data Model

### publicSeasonSummary (calculated)

| Field | Type | Rules |
|-------|------|-------|
| seasonId | INTEGER | References existing season |
| teamId, playerId | INTEGER | Existing team/player identifiers where applicable |
| played, won, drawn, lost | INTEGER | Derived from official regular-season results |
| points, rank | NUMBER | Only when confirmed season rules allow calculation |
| statCategory, total | TEXT / INTEGER | Category name and sum of confirmed player statistics |

## Gherkin AC

### US-10.1 - Browse the league

#### Scenario: Browse without signing in
* **Given** a league has a published season schedule
* **When** a visitor selects that league and season without an account
* **Then** the public season page is shown with no editing actions

### US-10.2 - View teams and players

#### Scenario: View a public roster
* **Given** a season team has roster players with private contact email addresses
* **When** a visitor opens the team page
* **Then** the approved player details appear without contact emails or management controls

### US-10.3 - View schedules and records

#### Scenario: Show current published game details
* **Given** a published game's field has been changed and another game has an official result
* **When** a visitor opens the season schedule
* **Then** the updated field is displayed and the completed game's confirmed score is shown

#### Scenario: Exclude unresolved results
* **Given** a game has conflicting reports but no official result
* **When** a visitor opens season records
* **Then** the game is identified as awaiting confirmation and is excluded from official totals

## Screen Requirements

### View: League and season directory - route public-leagues
Selectors for sport, league, and season; links to schedules and teams. Empty: "No league information is available." No management actions.

### View: Team and player pages - route public-team / public-player
Team name, season roster, official team record, and links to player stats. Player page lists confirmed categories and totals. Empty: "No confirmed statistics yet."

### View: Schedule and results - route public-schedule
Filter by team and date. Show time, home/away teams, field address, status, and official scores. Pending: "Result awaiting confirmation." Empty: "No published games for this selection."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

