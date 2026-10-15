# Feature: Equipment and Maintenance

## Header

**Feature ID:** 16  
**Branch pattern:** feature/16-equipment-and-maintenance  
**Assignment branch:** req-draft  
**Status:** Draft  
**Created:** 2026-10-09  
**Input:** Record equipment problems, maintenance work, and monthly repair spending.  
**Depends on:** [Feature 1 - Worker Accounts and Access](feature-1-worker-accounts-and-access.md), [Feature 2 - Company and Warehouse Management](feature-2-company-and-warehouse-management.md)  
**Sources:** Interview: forklifts, damage, repair costs, maintenance and cleaning crew, and a 5,000-dollar monthly maintenance budget.  
**Draft questions:** Confirm whether cleaning work is included and whether spending over the budget requires approval. The draft reports the overage without automatically blocking a repair.

## User Stories

### US-16.1: Report an equipment problem
**As a** warehouse worker  
**I want to** report a broken forklift with a description  
**So that** maintenance knows what needs attention

**Priority:** P2  
**Independent test:** Report a problem linked to a recorded forklift.  
**Acceptance scenarios:** See US-16.1 under Gherkin AC.

### US-16.2: Record maintenance work
**As a** maintenance worker  
**I want to** record the work completed and its actual cost  
**So that** the manager can see what was repaired

**Priority:** P2  
**Independent test:** Complete a task and record a 250-dollar repair.  
**Acceptance scenarios:** See US-16.2 under Gherkin AC.

### US-16.3: Compare spending with budget
**As a** manager  
**I want to** see maintenance costs against the month's budget  
**So that** I know how much has been spent

**Priority:** P2  
**Independent test:** Record costs of 5,200 against a monthly budget of 5,000.  
**Acceptance scenarios:** See US-16.3 under Gherkin AC.

## Functional Requirements

- **FR-001**: Equipment records MUST identify the asset, warehouse, active status, and service condition.
- **FR-002**: Maintenance tasks MUST include an asset or warehouse location, problem description, reporter, and report time.
- **FR-003**: Tasks MUST support open, in progress, and completed status and an assigned maintenance worker.
- **FR-004**: Completion MUST include a work note and completion time.
- **FR-005**: Repair costs MUST be recorded as nonnegative exact currency amounts with a cost date and task link.
- **FR-006**: The monthly maintenance budget MUST be configurable; the interview's 5000-dollar value MUST be treated as the initial draft amount, not a permanent hardcoded limit.
- **FR-007**: Monthly spending MUST use dated maintenance costs and MUST exclude supplier inventory purchases.
- **FR-008**: Budget reporting MUST show budget, spending, remaining amount, and overage when spending exceeds budget.
- **FR-009**: This draft MUST NOT block repairs solely because of an overage until the approval policy is confirmed.
- **FR-010**: Equipment and maintenance history MUST remain available after an asset is retired.

## Key Entities

- **Equipment**: forklift or other maintained asset.
- **MaintenanceTask**: reported problem or assigned work.
- **MaintenanceCost**: expense linked to a task.
- **MaintenanceBudget**: warehouse budget for a month.

## Initial Data Model

### equipment

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique equipment ID |
| assetNumber | STRING | Required; unique |
| warehouseId | INTEGER FK | References warehouses.id |
| description | STRING | Required |
| condition | ENUM | in service or out of service |
| isActive | BOOLEAN | Defaults to true |

### maintenanceTasks

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique task ID |
| equipmentId | INTEGER FK | Optional equipment link |
| locationId | INTEGER FK | Optional location link; at least one target required |
| description | TEXT | Required |
| reportedById, assignedWorkerId | INTEGER FK | Account and worker links |
| status | ENUM | open, in progress, completed |
| reportedAt, completedAt | DATETIME | Report time; completion time when closed |
| workNote | TEXT | Required on completion |

### maintenanceCosts

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique cost ID |
| taskId | INTEGER FK | References maintenanceTasks.id |
| amount | DECIMAL(12,2) | Nonnegative |
| costDate | DATE | Required |

### maintenanceBudgets

| Field | Type | Rules |
|-------|------|-------|
| id | INTEGER PK | Unique monthly budget |
| warehouseId | INTEGER FK | References warehouses.id |
| month | DATE | First day of month; unique per warehouse |
| amount | DECIMAL(12,2) | Nonnegative; draft initial value 5000 |

## Gherkin AC

### US-16.1 - Report an equipment problem

#### Scenario: Report an equipment problem
* **Given** forklift F1 is in the equipment list
* **When** the worker saves a problem description for F1
* **Then** an open maintenance task is linked to F1

### US-16.2 - Record maintenance work

#### Scenario: Record maintenance work
* **Given** an open maintenance task exists
* **When** the maintenance worker records the repair, completion time, and cost of 250
* **Then** the task is completed and 250 is recorded as maintenance spending

#### Scenario: Reject a negative repair cost
* **Given** a maintenance task exists
* **When** a worker records a cost of minus 25
* **Then** the cost is rejected and spending does not change

### US-16.3 - Compare spending with budget

#### Scenario: Compare spending with budget
* **Given** the monthly budget is 5000 and recorded costs total 5200
* **When** the manager opens the month's spending summary
* **Then** the summary shows spending of 5200 and an overage of 200
