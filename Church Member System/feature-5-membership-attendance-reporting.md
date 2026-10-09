# Feature: Membership and Attendance Reporting

## Header

**Feature ID:** 5  
**Branch pattern:** feature/5-membership-attendance-reporting  
**Status:** Draft  
**Created:** 2026-10-08  
**Input:** Show member headcounts, demographic summaries, and attendance reports for church planning.  
**Depends on:** [Feature 1 - Member Record Management](feature-1-member-record-management.md), [Feature 4 - Attendance Tracking](feature-4-attendance-tracking.md)

## User Stories

### US-5.1: View current membership counts
**As a** church leader  
**I want to** see separate totals for members and non-members  
**So that** I know how many people are in the congregation

**Priority:** P1  
**Independent test:** Compare totals with known member, visitor, and inactive records.  
**Acceptance scenarios:** See US-5.1 under Gherkin AC.

### US-5.2: View demographic information
**As a** church leader  
**I want to** see age and gender summaries with missing values identified  
**So that** I can plan classes and services around the congregation

**Priority:** P1  
**Independent test:** Check a summary with known and unknown demographic information.  
**Acceptance scenarios:** See US-5.2 under Gherkin AC.

### US-5.3: View attendance reports
**As a** church leader  
**I want to** filter attendance by date, class, or service  
**So that** I can compare participation across meetings

**Priority:** P1  
**Independent test:** Filter two meetings attended by Jordan and check visits and unique people.  
**Acceptance scenarios:** See US-5.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Directory summaries MUST distinguish active members, active non-members, and inactive people.
- **FR-002**: Demographic summaries MUST show which people are included and the date used to calculate age.
- **FR-003**: Age MUST be calculated from birthday; missing demographic data MUST be grouped as unknown.
- **FR-004**: Attendance reports MUST support an inclusive date range and class or service filters.
- **FR-005**: A start date after the end date MUST be rejected.
- **FR-006**: Reports MUST show each meeting headcount and distinguish attendance visits from unique attendees.
- **FR-007**: Empty attendance results MUST show zero counts and a no-attendance message.
- **FR-008**: Reports MUST reflect saved corrections and MUST NOT count enrollment as attendance.

## Initial Data Model

Reports read people, meetings, classes, and attendance from earlier features. Reports use the existing records and do not need a separate table.

### calculated report values

| Field | Type | Rules |
|-------|------|-------|
| activeMemberCount | INTEGER | Active people with member status |
| activeNonMemberCount | INTEGER | Active people with non-member status |
| inactiveCount | INTEGER | People with isActive set to false |
| age | INTEGER or unknown | Age in whole years on the report date |
| demographicGroupCount | INTEGER | Count of people included in the summary, including unknown values |
| meetingHeadcount | INTEGER | Distinct people marked present for one meeting |
| totalAttendanceVisits | INTEGER | Sum of selected meeting headcounts |
| uniqueAttendeeCount | INTEGER | Distinct people across selected meetings |

## Gherkin AC

### US-5.1 - View current membership counts

#### Scenario: View current membership counts
* **Given** there are 150 active members, 8 active non-members, and 2 inactive members
* **When** I view the directory summary
* **Then** it shows 150 active members and 8 active non-members with inactive people counted separately

### US-5.2 - View demographic information

#### Scenario: View demographic information
* **Given** some birthdays and genders are unknown
* **When** I view a demographic summary
* **Then** missing values appear as unknown and the summary shows which people are included and the date used

### US-5.3 - View attendance reports

#### Scenario: View attendance reports
* **Given** Jordan attended two meetings in a date range
* **When** I view the attendance report for that range
* **Then** Jordan counts once per meeting and the report distinguishes two visits from one unique attendee

#### Scenario: Reject reversed report dates
* **Given** I am choosing report dates
* **When** I choose an end date before the start date
* **Then** the system asks me to correct the range and does not run the report

#### Scenario: No attendance matches the filters
* **Given** no attendance matches the chosen date range and class
* **When** I run the report
* **Then** attendance visits and unique attendees both show zero
* **And** the report says no attendance was found

#### Scenario: Corrected attendance changes a report
* **Given** Jordan is the only person marked present at a meeting
* **And** the report shows one visit and one unique attendee for that meeting
* **When** the present mark is removed and I run the report again
* **Then** both counts are zero
