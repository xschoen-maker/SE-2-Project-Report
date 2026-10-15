# Feature: Worker Accounts and Access

## Header

**Feature ID:** 1  
**Branch pattern:** feature/1-worker-accounts-and-access  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Identify each worker and limit access to the work they are allowed to do.  
**Sources:** Manager interview: office, receiving, put-away, picking, shipping, drivers, managers, and maintenance workers.  
**Draft questions:** Confirm the permission list for each role, who may create customer accounts, and session timeout before implementation.

## User Stories

### US-1.1: Create a worker account
**As a** manager  
**I want to** create an account for a worker and assign their role  
**So that** workers use their own accounts

**Priority:** P1  
**Independent test:** Create a receiving worker and check the saved role.  
**Acceptance scenarios:** See US-1.1 under Gherkin AC.

### US-1.2: Sign in and sign out
**As a** warehouse worker  
**I want to** sign in on my phone or an office computer and sign out when finished  
**So that** my work is recorded under my account

**Priority:** P1  
**Independent test:** Sign in, open an allowed page, sign out, and try that page again.  
**Acceptance scenarios:** See US-1.2 under Gherkin AC.

### US-1.3: Restrict access
**As a** manager  
**I want to** give each role only the access it needs  
**So that** customers and unauthorized people cannot change warehouse records

**Priority:** P1  
**Independent test:** Use a customer account to attempt an inventory adjustment.  
**Acceptance scenarios:** See US-1.3 under Gherkin AC.

### US-1.4: Deactivate a worker
**As a** manager  
**I want to** disable an account when someone leaves  
**So that** former workers cannot keep using the system

**Priority:** P1  
**Independent test:** Deactivate a worker and try an existing session.  
**Acceptance scenarios:** See US-1.4 under Gherkin AC.

## Functional Requirements

- **FR-001**: Managers MUST be able to create, edit, and deactivate worker accounts.
- **FR-002**: Each account MUST have a unique username, a name, and at least one assigned role.
- **FR-003**: Passwords MUST be stored as salted hashes and MUST NOT be displayed or stored as plain text.
- **FR-004**: Sign-in MUST require a valid password and an active account; failed attempts MUST return a general sign-in error.
- **FR-005**: Every protected operation MUST check the current account and its permission, including direct requests.
- **FR-006**: Draft role groups MUST cover manager, office clerk, receiving worker, put-away worker, picker, shipping clerk, driver, maintenance worker, and customer.
- **FR-007**: Customers MUST only see their own orders and the customer-facing catalog; supplier costs and other customers' records MUST remain private.
- **FR-008**: Signing out or deactivating an account MUST invalidate its sessions.
- **FR-009**: Changes to stock and order processing MUST record the worker account and time.
- **FR-010**: Assigned warehouse tasks MUST be usable on a phone, and managers MUST be able to view task assignments and progress.

## Key Entities

- **Worker**: employee identity and name; linked to an account.
- **UserAccount**: sign-in credentials and active status; may belong to a worker or a customer.
- **Role and Permission**: allowed tasks assigned through roles.
- **Session**: current sign-in for an account.

## Initial Data Model

### workers

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique worker ID |
| name | STRING | Required |
| isActive | BOOLEAN | Defaults to true |

### userAccounts

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique account ID |
| username | STRING | Required; unique |
| passwordHash | STRING | Required; salted hash only |
| workerId | INTEGER FK | Optional; references workers.id |
| customerId | INTEGER FK | Optional; references customers.id from Feature 4 |
| isActive | BOOLEAN | Defaults to true |

### roles

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique role ID |
| name | STRING | Required; unique |

### permissions

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique permission ID |
| operation | STRING | Required; unique task name |

### userRoles

| Field | Type | Rules |
|-------|------|-------|
| userId | INTEGER FK | References userAccounts.id |
| roleId | INTEGER FK | References roles.id |

### rolePermissions

| Field | Type | Rules |
|-------|------|-------|
| roleId | INTEGER FK | References roles.id |
| permissionId | INTEGER FK | References permissions.id |

### sessions

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique session ID |
| userId | INTEGER FK | References userAccounts.id |
| tokenHash | STRING | Required; unique |
| expiresAt | DATETIME | Required expiration |
| revokedAt | DATETIME | Optional; set on sign-out or deactivation |

## Gherkin AC

### US-1.1 - Create a worker account

#### Scenario: Create a worker account
* **Given** a new receiving worker needs access
* **When** the manager saves the worker's name, unique username, initial password, and receiving role
* **Then** the account is created with receiving access

#### Scenario: Reject a duplicate username
* **Given** a worker account named receiver1 exists
* **When** the manager creates another account named receiver1
* **Then** the duplicate is rejected and the original account remains

### US-1.2 - Sign in and sign out

#### Scenario: Sign in and sign out
* **Given** an active worker account exists
* **When** the worker signs in with valid credentials
* **Then** the worker can open the pages allowed by their role

#### Scenario: Sign out
* **Given** a worker is signed in
* **When** the worker signs out
* **Then** protected warehouse pages cannot be opened using that session

#### Scenario: Reject an invalid password
* **Given** an active account exists
* **When** someone signs in with an incorrect password
* **Then** no session is created

### US-1.3 - Restrict access

#### Scenario: Restrict access
* **Given** a customer is signed in
* **When** the customer attempts to adjust warehouse stock
* **Then** access is denied and no stock record changes

### US-1.4 - Deactivate a worker

#### Scenario: Deactivate a worker
* **Given** a worker has an active session
* **When** the manager deactivates the account
* **Then** the existing session stops working and new sign-in attempts are denied
