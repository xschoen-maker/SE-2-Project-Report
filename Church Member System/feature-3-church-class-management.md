# Feature: Church Class Management

## Header

**Feature ID:** 3  
**Branch pattern:** feature/3-church-class-management  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Organize in-person church classes, meeting details, and participant rosters.  
**Depends on:** [Feature 1 - Member Record Management](feature-1-member-record-management.md)

## User Stories

### US-3.1: Create a class
**As a** class coordinator  
**I want to** save the class name, physical location, and schedule  
**So that** people know where and when to meet

**Priority:** P1  
**Independent test:** Create a class and reopen its meeting details.  
**Acceptance scenarios:** See US-3.1 under Gherkin AC.

### US-3.2: Manage a class roster
**As a** class coordinator  
**I want to** add or remove existing people from a roster  
**So that** teachers have an accurate participant list

**Priority:** P1  
**Independent test:** Enroll Jordan and check the roster.  
**Acceptance scenarios:** See US-3.2 under Gherkin AC.

### US-3.3: Update class details
**As a** class coordinator  
**I want to** change meeting details and mark classes inactive  
**So that** people have the current class details

**Priority:** P1  
**Independent test:** Change the room and mark the class inactive.  
**Acceptance scenarios:** See US-3.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Each class MUST store its name, physical location, schedule, and active status.
- **FR-002**: Name, location, and schedule MUST not be blank.
- **FR-003**: All classes MUST be in person.
- **FR-004**: Roster entries MUST reference existing people and classes.
- **FR-005**: People MUST be allowed in multiple classes but MUST NOT appear twice in one roster.
- **FR-006**: Removing an enrollment MUST NOT delete a person or past attendance.
- **FR-007**: Staff MUST be able to edit class details and mark classes inactive without removing history.

## Initial Data Model

A class meets in person. Its roster lists enrolled people; attendance is recorded for each meeting.

### classes

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique class ID |
| name, location, schedule | STRING | Required; cannot be blank |
| isActive | BOOLEAN | Defaults to true |
| createdAt, updatedAt | DATETIME | Creation and latest update |

### classEnrollments

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique enrollment ID |
| classId | INTEGER FK | Required; references classes.id |
| personId | INTEGER FK | Required; references people.id |
| enrolledAt | DATETIME | Enrollment time |

The pair classId and personId is unique.

## Gherkin AC

### US-3.1 - Create a class

#### Scenario: Create a class
* **Given** I am organizing a new class
* **When** I save its name, church room, and meeting schedule
* **Then** the class appears with its in-person meeting details

#### Scenario: Reject a class without a meeting location
* **Given** I am adding an in-person class
* **When** I save its name and schedule without a physical location
* **Then** the system asks for the location and does not create the class

### US-3.2 - Manage a class roster

#### Scenario: Manage a class roster
* **Given** Jordan and a class exist
* **When** I enroll Jordan
* **Then** Jordan appears once on the class roster

#### Scenario: Avoid duplicate enrollment
* **Given** Jordan is already on a class roster
* **When** I enroll Jordan in that class again
* **Then** the roster still has only one entry for Jordan

#### Scenario: Remove someone from a class roster
* **Given** Jordan is enrolled in a class and has attended a previous meeting
* **When** I remove Jordan from the roster
* **Then** Jordan no longer appears on the current roster
* **And** Jordan's person record and previous attendance remain

### US-3.3 - Update class details

#### Scenario: Update class details
* **Given** a class has a roster and past attendance
* **When** I update the room and mark the class inactive
* **Then** the changes are saved and past attendance is kept
