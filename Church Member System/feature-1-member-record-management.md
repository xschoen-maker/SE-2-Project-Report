# Feature: Member Record Management

## Header

**Feature ID:** 1  
**Branch pattern:** feature/1-member-record-management  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Replace the church card system with records for members and non-members.  

## User Stories

### US-1.1: Add a person
**As a** church secretary  
**I want to** record names, birthdays, gender, addresses, and membership status  
**So that** member and visitor information is kept together

**Priority:** P1  
**Independent test:** Save a visitor record and reopen it.  
**Acceptance scenarios:** See US-1.1 under Gherkin AC.

### US-1.2: Find and update a person
**As a** church secretary  
**I want to** find people and correct their details  
**So that** the directory stays current

**Priority:** P1  
**Independent test:** Search for Jordan, change the address, and reopen the record.  
**Acceptance scenarios:** See US-1.2 under Gherkin AC.

### US-1.3: Update membership status
**As a** church secretary  
**I want to** record membership changes and inactive people  
**So that** the church can keep an accurate member count

**Priority:** P1  
**Independent test:** Change a visitor to a member and verify the saved status.  
**Acceptance scenarios:** See US-1.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: The system MUST keep separate person records for members and non-members.
- **FR-002**: First name, last name, and membership status MUST be required; blank names MUST be rejected.
- **FR-003**: Birthday, gender, and address MUST be supported but may be left unknown.
- **FR-004**: Birthdays MUST be valid dates that are not in the future.
- **FR-005**: Staff MUST be able to search by name and filter by membership and active status.
- **FR-006**: Staff MUST be able to edit details and change membership or active status without replacing the person ID.
- **FR-007**: People with the same name MUST be allowed.
- **FR-008**: Inactive person records MUST keep family links and attendance history.

## Initial Data Model

A person record holds information about a member or visitor, including children. Having a record does not require a login account.

### people

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique person ID |
| firstName, lastName | STRING | Required; spaces trimmed; cannot be blank |
| birthday | DATE | Optional; not in the future |
| gender | STRING | Optional; unknown values allowed |
| address | STRING | Optional current address |
| membershipStatus | ENUM | Required: member or non-member |
| isActive | BOOLEAN | Required; defaults to true |
| createdAt, updatedAt | DATETIME | Creation and latest update |

## Gherkin AC

### US-1.1 - Add a person

#### Scenario: Add a person
* **Given** Jordan Lee is not in the directory
* **When** I save Jordan with a birthday, gender, address, and non-member status
* **Then** Jordan appears once as a non-member with the saved details

#### Scenario: Reject a missing name
* **Given** I am adding a person
* **When** I save a record with a blank last name
* **Then** the system identifies the missing name and creates no record

#### Scenario: Reject a future birthday
* **Given** I am adding a person
* **When** I enter a birthday after today's date
* **Then** the system asks for a valid birthday and does not save the record

### US-1.2 - Find and update a person

#### Scenario: Find and update a person
* **Given** Jordan has a person record
* **When** I find Jordan by name and save a corrected address
* **Then** the new address is shown on the same person record

### US-1.3 - Update membership status

#### Scenario: Update membership status
* **Given** Jordan is an active non-member
* **When** I change Jordan to a member
* **Then** Jordan is an active member without another person record

#### Scenario: Mark a member inactive
* **Given** Jordan is a member with saved attendance and a family link
* **When** I mark Jordan inactive
* **Then** Jordan no longer appears in the active member list
* **And** Jordan's attendance and family link remain
