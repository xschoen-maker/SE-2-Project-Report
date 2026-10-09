# Feature: Family Management

## Header

**Feature ID:** 2  
**Branch pattern:** feature/2-family-management  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Group related people into families so staff can find household information together.  
**Depends on:** [Feature 1 - Member Record Management](feature-1-member-record-management.md)

## User Stories

### US-2.1: Create a family
**As a** church secretary  
**I want to** create a named family group  
**So that** relatives can be organized together

**Priority:** P1  
**Independent test:** Create and reopen the Lee family.  
**Acceptance scenarios:** See US-2.1 under Gherkin AC.

### US-2.2: Link people to a family
**As a** church secretary  
**I want to** assign existing people to a family  
**So that** staff can see who belongs to the household

**Priority:** P1  
**Independent test:** Assign Jordan to Lee and view the family list.  
**Acceptance scenarios:** See US-2.2 under Gherkin AC.

### US-2.3: Correct family membership
**As a** church secretary  
**I want to** move or unlink someone from a family  
**So that** family information stays accurate

**Priority:** P1  
**Independent test:** Move Jordan to another family and check both groups.  
**Acceptance scenarios:** See US-2.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Staff MUST be able to create, view, and rename families.
- **FR-002**: Family names MUST not be blank.
- **FR-003**: A person MUST belong to zero or one current family; a family MUST allow multiple people.
- **FR-004**: Assignments MUST reference existing families and people.
- **FR-005**: Staff MUST be able to view a family and its linked people.
- **FR-006**: Moving or unlinking someone MUST keep their person record and attendance history.
- **FR-007**: Deleting a family MUST require confirmation and clear family links without deleting people.

## Initial Data Model

A family groups people who are already in the directory. A person can be listed without a family.

### families

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique family ID |
| name | STRING | Required; spaces trimmed; cannot be blank |
| createdAt, updatedAt | DATETIME | Creation and latest update |

### people (extension of Feature 1)

| Field | Type | Rules |
|-------|------|-------|
| familyId | INTEGER FK | Optional; references families.id; cleared on family deletion |

## Gherkin AC

### US-2.1 - Create a family

#### Scenario: Create a family
* **Given** no Lee family group exists
* **When** I create a family named Lee
* **Then** the family appears with no assigned people

#### Scenario: Reject a blank family name
* **Given** I am creating a family
* **When** I submit a name containing only spaces
* **Then** the system asks for a family name and creates no family

### US-2.2 - Link people to a family

#### Scenario: Link people to a family
* **Given** Jordan and the Lee family exist
* **When** I assign Jordan to Lee
* **Then** Jordan appears in the family with the same person ID

#### Scenario: Reject an unknown family
* **Given** Jordan has a person record
* **When** I assign Jordan to a family that does not exist
* **Then** the system rejects the assignment and leaves the existing family link unchanged

### US-2.3 - Correct family membership

#### Scenario: Correct family membership
* **Given** Jordan belongs to Lee and another family exists
* **When** I move Jordan to the other family
* **Then** only the new family lists Jordan and attendance history is kept

#### Scenario: Delete a family
* **Given** the Lee family contains Jordan and Casey
* **When** I delete the family and confirm
* **Then** the family is removed
* **And** Jordan and Casey remain in the directory without that family link
