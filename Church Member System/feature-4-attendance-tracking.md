# Feature: Attendance Tracking

## Header

**Feature ID:** 4  
**Branch pattern:** feature/4-attendance-tracking  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Record attendance at each in-person class or church service and correct recording mistakes.  
**Depends on:** [Feature 1 - Member Record Management](feature-1-member-record-management.md), [Feature 3 - Church Class Management](feature-3-church-class-management.md)

## User Stories

### US-4.1: Create a meeting
**As a** church secretary  
**I want to** record the date and time of a class meeting or service  
**So that** attendance belongs to a specific gathering

**Priority:** P1  
**Independent test:** Create a class meeting and a service with a date and time.  
**Acceptance scenarios:** See US-4.1 under Gherkin AC.

### US-4.2: Mark people present
**As a** teacher  
**I want to** mark who attended a meeting  
**So that** the church has an accurate headcount

**Priority:** P1  
**Independent test:** Mark Jordan present and reopen the attendance list.  
**Acceptance scenarios:** See US-4.2 under Gherkin AC.

### US-4.3: Correct attendance
**As a** teacher  
**I want to** remove an incorrect present mark  
**So that** the report counts only people who attended

**Priority:** P1  
**Independent test:** Remove Jordan from attendance and check the headcount.  
**Acceptance scenarios:** See US-4.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Meetings MUST have a name, a type of class or service, and a valid date and time.
- **FR-002**: Class meetings MUST reference a class; service meetings MUST NOT require one.
- **FR-003**: Attendance MUST reference an existing person and meeting, including non-members.
- **FR-004**: A person MUST be counted at most once per meeting.
- **FR-005**: Class enrollment MUST NOT automatically mark a person present.
- **FR-006**: Meeting headcount MUST equal the number of distinct people marked present.
- **FR-007**: Staff MUST be able to add or remove present marks and see corrected counts.
- **FR-008**: A missing present mark MUST NOT mean that someone was expected to attend.

## Initial Data Model

A meeting is a class or service on a specific date. Attendance records who was there.

### meetings

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique meeting ID |
| name | STRING | Required; cannot be blank |
| type | ENUM | Required: class or service |
| startsAt | DATETIME | Required meeting date and time |
| classId | INTEGER FK | References classes.id; required for class, null for service |

### attendance

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique present mark |
| meetingId | INTEGER FK | Required; references meetings.id |
| personId | INTEGER FK | Required; references people.id |
| recordedAt | DATETIME | Time recorded |

The pair meetingId and personId is unique. Headcount is calculated from attendance.

## Gherkin AC

### US-4.1 - Create a meeting

#### Scenario: Create a meeting
* **Given** a class exists
* **When** I create a class meeting with a date and time
* **Then** the meeting is available for attendance recording

#### Scenario: Create a service meeting
* **Given** I am recording a church service
* **When** I save its name, date, and time with type service and no class
* **Then** the meeting is created without a class link

### US-4.2 - Mark people present

#### Scenario: Mark people present
* **Given** Jordan has a person record and a meeting has been created
* **When** I mark Jordan present
* **Then** Jordan appears once and the meeting headcount increases by one

#### Scenario: Do not double-count a person
* **Given** Jordan is already marked present
* **When** I mark Jordan present again at the same meeting
* **Then** the headcount stays unchanged and Jordan has one present mark

#### Scenario: Start attendance for an enrolled class
* **Given** a class roster contains Jordan and Casey
* **When** I create a meeting for the class before recording attendance
* **Then** its headcount is zero
* **And** neither enrolled person is automatically marked present

### US-4.3 - Correct attendance

#### Scenario: Correct attendance
* **Given** Jordan was marked present by mistake
* **When** I remove the present mark
* **Then** Jordan is no longer counted for that meeting and the person record remains
