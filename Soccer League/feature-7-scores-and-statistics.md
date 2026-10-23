# Feature: Scores and Statistics

## Header

**Feature ID:** 7  
**Branch pattern:** feature/7-scores-and-statistics  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Collect game scores and player statistics from team managers and keep corrections traceable.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 4 - Player Rosters](feature-4-player-rosters.md), [Feature 6 - Season Scheduling](feature-6-season-scheduling.md)  
**Needs clarification:** Confirm the soccer stat categories, submission deadlines, and result-confirmation policy. This draft keeps conflicting reports out of official standings until the supervisor resolves them.

## User Stories

### US-7.1: Report a game score
**As a** team manager  
**I want to** enter scores for a game involving my team  
**So that** the league gets results in one place instead of late texts and emails

**Priority:** P1  
**Independent test:** Save a score report for a completed game involving the assigned team.  
**Acceptance scenarios:** See US-7.1 under Gherkin AC.

### US-7.2: Enter player statistics
**As a** team manager  
**I want to** record statistics for players on my game roster  
**So that** player records can be kept current

**Priority:** P1  
**Independent test:** Save a nonnegative value for a configured soccer stat and a roster player.  
**Acceptance scenarios:** See US-7.2 under Gherkin AC.

### US-7.3: Resolve and correct results
**As a** supervisor  
**I want to** review conflicting reports and retain earlier values when correcting them  
**So that** public results and standings use one official result

**Priority:** P1  
**Independent test:** Resolve two disagreeing score reports and check the published result.  
**Acceptance scenarios:** See US-7.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Only a supervisor or a manager assigned to a participating season team MUST submit game reports; a manager MUST update only that team's player statistics.
- **FR-002**: Reports MUST link a game, submitting user, submission time, and nonnegative whole-number home and away scores.
- **FR-003**: Player statistics MUST reference an eligible roster entry from the reported game's participating team and a confirmed stat category for that sport.
- **FR-004**: Count statistics MUST be nonnegative whole numbers; repeated saves MUST update the same game/player/category entry rather than add its value twice.
- **FR-005**: Conflicting score reports MUST be flagged; unresolved reports MUST NOT contribute an official result to public standings.
- **FR-006**: The supervisor MUST be able to confirm or correct an official result; a correction MUST record previous values, new values, actor, time, and reason.
- **FR-007**: The league MUST distinguish games awaiting a report from games with official results; no automatic penalty or reporting deadline is assumed.
- **FR-008**: Official player totals and team records MUST use only games with a confirmed official result, without double counting replaced values.

## Key Entities

- ScoreReport: a manager's submitted account of one game.
- OfficialResult: the single current confirmed score for one game.
- StatCategory: a supervisor-confirmed statistic for one sport.
- PlayerGameStat: a roster player's statistic in one game.
- ResultCorrection: retained values and explanation for an official result change.

## Initial Data Model

### scoreReports

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique report ID |
| gameId | INTEGER FK | References games.id |
| submittedById | INTEGER FK | References users.id |
| homeScore, awayScore | INTEGER | Nonnegative |
| submittedAt | DATETIME | Required |

### officialResults

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique official result |
| gameId | INTEGER FK | References games.id; unique |
| homeScore, awayScore | INTEGER | Nonnegative |
| confirmedById | INTEGER FK | References users.id |
| confirmedAt | DATETIME | Required |

### statCategories

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique category ID |
| sportId | INTEGER FK | References sports.id |
| name | TEXT | Confirmed category; unique within sport |

### playerGameStats

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique stat record |
| gameId, rosterEntryId, categoryId | INTEGER FK | Reference games.id, rosterEntries.id, statCategories.id; unique combination |
| value | INTEGER | Nonnegative count |
| updatedById | INTEGER FK | References users.id |
| updatedAt | DATETIME | Required |

### resultCorrections

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique correction |
| gameId | INTEGER FK | References games.id |
| previousValues, newValues, reason | TEXT | Required |
| changedById | INTEGER FK | References users.id |
| changedAt | DATETIME | Required |

## Gherkin AC

### US-7.1 - Report a game score

#### Scenario: Submit a score report
* **Given** a manager's team played a scheduled game
* **When** the manager saves home score 2 and away score 1
* **Then** the report is linked to the game and submitting manager

#### Scenario: Block an unrelated game's report
* **Given** a manager controls neither team in a game
* **When** the manager attempts to submit its score
* **Then** the submission is denied and the game is unchanged

### US-7.2 - Enter player statistics

#### Scenario: Record a player's statistic
* **Given** a roster player belongs to the manager's participating team and a soccer stat category exists
* **When** the manager records a value of 2 for that category in the game
* **Then** the player's game statistic is saved once for that category

#### Scenario: Reject an invalid statistic
* **Given** a manager is entering a player count statistic
* **When** the manager saves a negative value
* **Then** the value is rejected and existing statistics remain unchanged

### US-7.3 - Resolve and correct results

#### Scenario: Resolve conflicting score reports
* **Given** the two teams submitted different scores for the same game
* **When** the supervisor confirms the correct score with a reason
* **Then** one official result is used, the earlier reports remain available, and standings use the corrected result only once

## Screen Requirements

### View: Game reporting - route game-reporting
List assigned team's games with **Enter score** and **Enter player stats**. Show missing reports and conflicting scores. Score form labels **Home score** and **Away score**. Empty: "No games to report."

### View: Player game stats - route player-game-stats
Show only the selected team's roster and confirmed sport categories. Save updates without duplicating totals. Invalid counts show "Enter a whole number of zero or more."

### View: Result review - route result-review
Supervisor view shows both team reports, current official result, and correction history. **Confirm result** and **Correct result** require a reason when resolving a conflict or changing confirmed values.

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

