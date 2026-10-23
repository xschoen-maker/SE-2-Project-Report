# Feature: League Messages

## Header

**Feature ID:** 11  
**Branch pattern:** feature/11-league-messages  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Send game notices to managers and referees and league announcements to managers or players.  
**Depends on:** [Feature 1 - User Access](feature-1-user-access.md), [Feature 3 - Team Management](feature-3-team-management.md), [Feature 4 - Player Rosters](feature-4-player-rosters.md), [Feature 6 - Season Scheduling](feature-6-season-scheduling.md), [Feature 8 - Referee Management](feature-8-referee-management.md), [Feature 9 - Referee Assignments](feature-9-referee-assignments.md)  
**Needs clarification:** Confirm the email or messaging service, upcoming-game reminder timing, and who handles recipient replies. This draft uses supervisor-reviewed email and does not require manager or referee messaging accounts.

## User Stories

### US-11.1: Send game reminders
**As a** supervisor  
**I want to** email assigned referees and team managers about upcoming games  
**So that** they have the current time and location

**Priority:** P1  
**Independent test:** Preview a game's current officials and manager recipients and send one notice.  
**Acceptance scenarios:** See US-11.1 under Gherkin AC.

### US-11.2: Notify schedule and referee changes
**As a** supervisor  
**I want to** send updated details when a game or official changes  
**So that** affected people are not left with an old assignment

**Priority:** P1  
**Independent test:** Reschedule a game and prepare a change notice.  
**Acceptance scenarios:** See US-11.2 under Gherkin AC.

### US-11.3: Send group announcements
**As a** supervisor  
**I want to** email all managers or all players in a selected league season  
**So that** I can share league information without collecting addresses by hand

**Priority:** P1  
**Independent test:** Select a season's player group and preview the recipient list.  
**Acceptance scenarios:** See US-11.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Only supervisors MUST send league-wide or game-related notices in this draft.
- **FR-002**: Game notices MUST identify the selected game and use its current teams, interval, field, managers, and affected referee assignments.
- **FR-003**: Manager email MUST be stored privately for notices; missing contact addresses MUST be identified before sending.
- **FR-004**: Group messages MUST be scoped to a selected league season and chosen group of managers or roster players; duplicate email addresses MUST be removed.
- **FR-005**: Every outgoing message MUST have a subject, body, recipient preview, and explicit send action; saving a draft MUST NOT send it.
- **FR-006**: Messages MUST protect recipient email addresses from other recipients and MUST not expose the directory publicly.
- **FR-007**: The system MUST record each recipient's sending outcome and sent time; provider acceptance MUST not be presented as proof the person read the message.
- **FR-008**: Failed deliveries MUST be visible and selectable for retry without deliberately resending to successful recipients.
- **FR-009**: Automatic reminder timing and reply handling MUST remain for confirmation; no referee or player editing access is added by messaging.

## Key Entities

- Message: supervisor-written announcement or game notice.
- MessageRecipient: one private destination and its sending outcome.
- Manager contact: private email associated with an existing manager user.

## Initial Data Model

### users (additional field)

| Field | Type | Rules |
|-------|------|-------|
| email | TEXT | Optional private valid contact email for manager notices |

### messages

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique message ID |
| seasonId | INTEGER FK | References seasons.id |
| gameId | INTEGER FK | Optional; references games.id |
| subject, body | TEXT | Required before sending |
| audience | ENUM | game contacts, managers, or players |
| createdById | INTEGER FK | References users.id |
| createdAt | DATETIME | Required |
| status | ENUM | draft, sending, sent, partial failure |

### messageRecipients

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique recipient record |
| messageId | INTEGER FK | References messages.id |
| email | TEXT | Private saved destination; unique per message |
| status | ENUM | pending, accepted by provider, failed |
| sentAt | DATETIME | Set after successful sending |
| errorNote | TEXT | Reason when failed |

## Gherkin AC

### US-11.1 - Send game reminders

#### Scenario: Send an upcoming-game notice
* **Given** a future game has team managers and assigned referees with email addresses
* **When** the supervisor reviews the recipients and sends the game notice
* **Then** the notice includes current teams, date, time, and field and records the sending outcome

### US-11.2 - Notify schedule and referee changes

#### Scenario: Prepare a changed-game notice
* **Given** a published game has a saved time or location change
* **When** the supervisor prepares its change notice
* **Then** the current managers and affected referees are listed with the changed details and the notice remains unsent until reviewed

### US-11.3 - Send group announcements

#### Scenario: Send a season announcement
* **Given** a selected season has players with saved email addresses
* **When** the supervisor reviews and sends an announcement to that season's players
* **Then** each distinct address receives the message without exposing other private recipient addresses

#### Scenario: Show a delivery problem
* **Given** a recipient is missing an email address or delivery fails
* **When** the supervisor reviews message outcomes
* **Then** the missing or failed recipient is identified without being marked successfully delivered

## Screen Requirements

### View: Messages - route messages
Supervisor-only list showing subject, season, audience, and outcome. **New message** selects game contacts, season managers, or season players. Empty: "No messages have been created."

### View: Compose and review - route compose-message
Fields: season, optional game, audience, subject, and body. Show recipient preview, duplicate removal, and missing-contact warnings. Actions **Save draft** and **Send message**; confirm the selected audience before sending.

### View: Message outcomes - route message-outcomes
Display successful and failed recipient outcomes privately. **Retry failed recipients** excludes successful recipients. Label provider acceptance as "Sent to email service", not "Read".

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

