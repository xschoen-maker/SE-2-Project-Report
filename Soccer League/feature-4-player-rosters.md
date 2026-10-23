# Feature: Player Rosters

## Header

**Feature ID:** 4  
**Branch pattern:** feature/4-player-rosters  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Let team managers maintain their own season rosters and the player details shown to the public.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 3 - Team Management](feature-3-team-management.md)  
**Needs clarification:** Confirm the player information allowed on public pages and any roster eligibility, transfer, or roster-size rules; no age or registration restrictions were provided.

## User Stories

### US-4.1: Add players to my team
**As a** team manager  
**I want to** add players to my assigned season team  
**So that** the public roster stays current without sending every update to the supervisor

**Priority:** P1  
**Independent test:** Add a named player to an assigned team.  
**Acceptance scenarios:** See US-4.1 under Gherkin AC.

### US-4.2: Update a roster
**As a** team manager  
**I want to** correct player details and mark a player inactive  
**So that** the roster reflects who is on the team

**Priority:** P1  
**Independent test:** Correct a player name while preserving earlier game statistics.  
**Acceptance scenarios:** See US-4.2 under Gherkin AC.

### US-4.3: Keep contact information private
**As a** team manager  
**I want to** save player email addresses separately from public roster details  
**So that** league messages can reach players without exposing their addresses

**Priority:** P1  
**Independent test:** Save an email address and check the public roster.  
**Acceptance scenarios:** See US-4.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Supervisors MUST manage all rosters; managers MUST add or update only players on their assigned season teams.
- **FR-002**: A roster entry MUST link an identified player to one season team; duplicate entries on that roster MUST be rejected.
- **FR-003**: Player records MUST include a required display name and optional contact email.
- **FR-004**: A supplied email address MUST have a valid email format; absence of an email MUST not prevent roster entry.
- **FR-005**: Public pages MUST expose only approved roster details and MUST NOT expose player contact email or account information.
- **FR-006**: Inactivating a roster entry MUST retain its earlier game statistics and season association.
- **FR-007**: Player eligibility and transfer rules MUST remain marked for clarification rather than adding unprovided limits.

## Key Entities

- Player: a named person with an optional private contact email.
- RosterEntry: a player's membership in one season team.

## Initial Data Model

### players

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique player ID |
| displayName | TEXT | Required; public display subject to agreed policy |
| contactEmail | TEXT | Optional; private |
| isActive | BOOLEAN | Required |

### rosterEntries

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique roster ID |
| seasonTeamId | INTEGER FK | References seasonTeams.id |
| playerId | INTEGER FK | References players.id; unique pair with seasonTeamId |
| isActive | BOOLEAN | Required; preserve historical stats |

## Gherkin AC

### US-4.1 - Add players to my team

#### Scenario: Add a player to an assigned roster
* **Given** a manager is assigned to a season team
* **When** the manager saves a player's name on that roster
* **Then** the player appears on the team's season roster

#### Scenario: Reject a duplicate roster entry
* **Given** a player is already on a team's season roster
* **When** the manager adds the same player to that roster again
* **Then** the duplicate is rejected

### US-4.2 - Update a roster

#### Scenario: Correct a player name
* **Given** a player has recorded game statistics
* **When** the assigned manager corrects the player's display name
* **Then** the name is updated and the earlier statistics remain linked to the same player

### US-4.3 - Keep contact information private

#### Scenario: Hide private player contact details
* **Given** a roster player has a saved contact email
* **When** a visitor opens the public team roster
* **Then** the approved public name appears and the contact email is not returned or displayed

## Screen Requirements

### View: Team roster - route team-roster
Show season, team, and player names. Assigned managers and supervisors see **Add player**, **Edit player**, and **Mark inactive**. Form separates display name from private contact email. Empty: "No players on this roster."

### View: Player details - route player-details
Management view shows roster membership and saved contact details; public view shows only approved name, team, and published statistics. Invalid email shows "Enter a valid email address."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

