# Feature: User Access

## Header

**Feature ID:** 1  
**Branch pattern:** feature/1-user-access  
**Assignment branch:** soccer-league  
**Status:** Draft  
**Created:** 2026-10-22  
**Input:** Identify supervisors and team managers and limit who can change league records.  
**Needs clarification:** Confirm who creates the first supervisor account and the password reset process.

## User Stories

### US-1.1: Maintain user accounts
**As a** supervisor  
**I want to** create, update, and deactivate supervisor and manager accounts  
**So that** only authorized people can change league data

**Priority:** P1  
**Independent test:** Create a manager account and assign it to a team.  
**Acceptance scenarios:** See US-1.1 under Gherkin AC.

### US-1.2: Sign in and sign out
**As a** team manager  
**I want to** sign in and sign out  
**So that** my updates are associated with my account

**Priority:** P1  
**Independent test:** Sign in, open an allowed screen, sign out, and try it again.  
**Acceptance scenarios:** See US-1.2 under Gherkin AC.

### US-1.3: Protect league records
**As a** supervisor  
**I want to** limit managers to their own teams and keep public visitors read-only  
**So that** league information cannot be changed by unauthorized people

**Priority:** P1  
**Independent test:** Attempt to edit another manager's roster and attempt an anonymous update.  
**Acceptance scenarios:** See US-1.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Only active supervisors and team managers MUST be able to add or edit league data.
- **FR-002**: Supervisors MUST be able to maintain user accounts and manage all league records.
- **FR-003**: Manager access MUST be checked against current team assignments, including direct requests, rather than only hiding screen controls.
- **FR-004**: Accounts MUST have a name, unique username, password hash, role, and active status; passwords MUST NOT be stored or displayed in plain text.
- **FR-005**: Signing in MUST require valid credentials and an active account; failed sign-in MUST not create a session.
- **FR-006**: Signing out or deactivating an account MUST end its editing access.
- **FR-007**: Public viewing MUST not require an account; a referee or player contact record MUST NOT itself grant editing access.

## Key Entities

- User: a supervisor or manager account; managers are linked to season teams in Feature 3.
- Session: a signed-in user's current access.

## Initial Data Model

### users

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique account ID |
| name, username | TEXT | Required; username unique |
| passwordHash | TEXT | Required; password hash only |
| role | ENUM | supervisor or manager |
| isActive | BOOLEAN | Required |

### sessions

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique session ID |
| userId | INTEGER FK | References users.id |
| tokenHash | TEXT | Required; unique |
| expiresAt | DATETIME | Required |
| endedAt | DATETIME | Optional; set when ended |

## Gherkin AC

### US-1.1 - Maintain user accounts

#### Scenario: Create a manager account
* **Given** a new team manager needs access
* **When** the supervisor saves a name, unique username, password, and manager role
* **Then** the account is created with manager access

#### Scenario: Reject a duplicate username
* **Given** an account uses the username manager1
* **When** the supervisor saves another account with manager1
* **Then** the duplicate is rejected and the first account is unchanged

### US-1.2 - Sign in and sign out

#### Scenario: Sign in with valid credentials
* **Given** an active manager account exists
* **When** the manager signs in with the correct password
* **Then** the manager can open permitted management screens

#### Scenario: End a session
* **Given** a manager is signed in
* **When** the manager selects Sign out
* **Then** the session ends and editing requires another sign-in

### US-1.3 - Protect league records

#### Scenario: Block another team's roster change
* **Given** a manager is assigned to Team A only
* **When** the manager attempts to change Team B's roster
* **Then** access is denied and the roster is unchanged

#### Scenario: Block public editing
* **Given** a visitor is not signed in
* **When** the visitor attempts to save a league record directly
* **Then** access is denied and no record changes

## Screen Requirements

### View: Sign in - route sign-in
Username and password fields; primary action **Sign in**. Show "Enter your username and password." when required fields are missing and "Unable to sign in." for invalid credentials.

### View: Users - route users
Supervisor-only list with name, username, role, and status. **Add user**, **Edit user**, and **Deactivate user** actions; confirm deactivation. Empty: "No user accounts found."

### View: Signed-in navigation - route management-home
Show league management links allowed for the current account, the signed-in name, and **Sign out**. Unauthorized editing shows "You do not have permission to change this record."

While a screen loads, show a progress indicator. If loading or saving fails, show a clear error, keep entered values, and provide a retry action. Give icon-only actions accessible labels.

