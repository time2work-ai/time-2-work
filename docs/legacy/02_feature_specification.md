---
title: "Time2Work: Feature Specification (BDD Format)"
description: "Detailed BDD scenarios for all Time2Work features"
version: "1.0"
date: "2025-09-02"
type: "BDD Specification"
audience: "QA Engineers, Developers, Product Managers"
---

# Time2Work: Feature Specification (BDD Format)

## Table of Contents

- [Employee Profile and Contract Management](#employee-profile-and-contract-management)
- [Daily Time and Compliance Tracking](#daily-time-and-compliance-tracking)
- [Leave Management](#leave-management)
- [Data Corrections and System Auditing](#data-corrections-and-system-auditing)
- [Compliance, Reporting, and Legal Edge Cases](#compliance-reporting-and-legal-edge-cases)

## Employee Profile and Contract Management

This feature covers the creation and configuration of employee profiles, ensuring all calculations are based on their specific, legally-defined attributes.

```gherkin
Feature: Employee Profile and Contract Management

  Background:
    Given I am logged in as an Admin

  Scenario: Standard Employee Onboarding (Over 10 Years Tenure)
    When I create a new employee with a "University Degree" and "3 years" of prior work experience
    Then the system should calculate their total work tenure as "11 years"
    And their annual leave entitlement should be "26 days"

  Scenario: Junior Employee Onboarding (Under 10 Years Tenure)
    When I create a new employee who graduated from "Secondary School" with "0 years" of prior work experience
    Then the system should calculate their total work tenure as "4 years"
    And their annual leave entitlement should be "20 days"

  Scenario: First-Time Employee Onboarding
    When I create a new employee and mark them as a "first-time employee"
    Then their leave entitlement should be calculated as "1/12th of their annual limit" for each month worked in the first year

  Scenario: Part-Time Employee Configuration
    Given an employee has a total work tenure of "11 years"
    When I configure their contract as "Umowa o pracę" on a "0.5" (half-time) basis
    Then their annual leave entitlement should be proportionally calculated as "13 days"

  Scenario: Configuring a Task-Based Work System (Zadaniowy)
    When I assign an employee to the "zadaniowy" (task-based) work system
    Then their time tracking interface should be simplified, disabling start/end hour logging
    And overtime calculations for this employee should be disabled

  Scenario: Configuring an Equivalent Work System (Równoważny)
    When I assign an employee to the "równoważny" (equivalent) work system with a "3 month" settlement period
    Then the system must validate that their scheduled hours average no more than "40" per week over the period
    And the system should allow scheduling daily work up to "12 hours"

  Scenario: Configuring a Disabled Employee
    When I update an employee's profile to a "moderate" degree of disability
    Then their daily work norm must be reduced to "7 hours"
    And "10 extra days" must be added to their annual leave entitlement
    And the system must block assigning them overtime or night work

  Scenario: Configuring a Pregnant Employee
    When I update an employee's profile status to "pregnant"
    Then the system must immediately block any assignment of overtime or night work
    And the system must flag them as having priority for remote work requests
```

## Daily Time and Compliance Tracking

This feature covers how employees and admins interact with the system for daily time recording and how the system enforces compliance rules.

```gherkin
Feature: Daily Time and Compliance Tracking

  Scenario: Standard Clock-In / Clock-Out
    Given I am logged in as an Employee on a "podstawowy" work system
    When I clock in at "08:02" and clock out at "16:05"
    Then the system should record my total work time as "8 hours and 3 minutes"

  Scenario: Logging Overtime
    Given I am logged in as an Employee on a "podstawowy" work system
    When I log my work from "09:00" to "18:30" with a "30 minute" unpaid break
    Then the system should categorize "8 hours" as standard time
    And the system should categorize "1 hour" as overtime to be compensated at "150%"

  Scenario: Manual Timesheet Entry
    Given I am logged in as an Employee
    When I manually submit a timesheet for yesterday with hours "08:00" to "16:00"
    Then the entry should be flagged as "Pending Approval" for my manager
    And the entry should be clearly marked as a "Manual Entry" in the audit log

  Scenario: Mandatory Rest Period Violation
    Given I am logged in as an Admin
    And an employee has a shift ending at "22:00" on Monday
    When I attempt to schedule a new shift for that employee starting at "07:00" on Tuesday
    Then the system must reject the schedule
    And the system must display a critical error: "Violation of the 11-hour mandatory daily rest period."

  Scenario: Night Work Logging and Compensation
    Given I am logged in as an Admin
    When I approve a timesheet for an employee who worked from "22:00" to "06:00"
    Then the system must identify all "8 hours" as night work
    And the system must automatically calculate the required pay supplement for those hours
```

## Leave Management

This feature covers the entire lifecycle of leave requests, from application to approval and final pay-out on termination.

```gherkin
Feature: Leave Management

  Scenario: Requesting Annual Leave
    Given I am logged in as an Employee with "15 days" of remaining annual leave
    When I request "5 days" of annual leave for next month
    Then the request should be sent to my manager for approval
    And my available leave balance should show "10 days" with "5 days pending"

  Scenario: Requesting "Leave on Demand"
    Given I am logged in as an Employee
    And I have used "1" day of "Leave on Demand" this year
    When I submit a request for "Leave on Demand" for today
    Then the system should automatically approve the request
    And my annual leave balance should be reduced by "1 day"
    And my "Leave on Demand" counter should update to "2"

  Scenario: Insufficient Leave Balance
    Given I am logged in as an Employee with "3 days" of remaining annual leave
    When I attempt to request "5 days" of annual leave
    Then the system must reject the request
    And display an error: "Insufficient leave balance. Available: 3, Requested: 5."

  Scenario: Parental Leave Chain Management
    Given I am logged in as an Admin for a new mother
    When I initiate the parental leave process
    Then the system must guide me to first apply "20 weeks" of "urlop macierzyński"
    And then allow transitioning to "41 weeks" of "urlop rodzicielski"
    And the system must track the "9 non-transferable weeks" for the other parent separately

  Scenario: Past-Due Leave Alert
    Given I am logged in as an Admin
    And the current date is "August 1, 2026"
    When I view the compliance dashboard
    Then the system must display a critical alert listing all employees with unused leave from "2025"
    And the alert must state the final deadline is "September 30, 2026"

  Scenario: Employee Termination and Leave Pay-out
    Given I am logged in as an Admin
    And an employee with "10 days" of unused annual leave is being terminated
    When I execute the final payroll process for this employee
    Then the system must automatically calculate the monetary pay-out for the "10 days"
    And the calculation must use the official "ekwiwalent" coefficient, not a simple daily rate
```

## Data Corrections and System Auditing

This feature ensures that all data can be corrected with a full audit trail.

```gherkin
Feature: Data Corrections and System Auditing

  Background:
    Given I am logged in as an Admin

  Scenario: Correcting a Timesheet Entry
    Given an employee's approved timesheet from last week shows "8 hours" worked
    When I correct the entry to "9 hours" and provide "Forgot to log overtime" as the reason
    Then the system must recalculate "1 hour" of overtime for that day
    And the change must be recorded in a permanent audit log with the reason and my user ID

  Scenario: Cancelling an Approved Leave Request
    Given an employee has an approved leave request for "5 days" next week
    When I cancel the leave request on behalf of the employee
    Then the "5 days" must be immediately returned to their annual leave balance

  Scenario: Historical Data Import
    When I import a set of historical work records for an employee from 2 years ago
    Then the system must update the employee's total work tenure calculation
    But the system must not alter any recent, approved timesheets
```

## Compliance, Reporting, and Legal Edge Cases

This feature ensures the system can produce compliant reports and handles complex legal edge cases correctly.

```gherkin
Feature: Compliance, Reporting, and Legal Edge Cases

  Background:
    Given I am logged in as an Admin

  Scenario: Generating a PIP Audit Report
    When I request a "PIP Audit Report" for an employee for the last "3 years"
    Then the system must generate a single, comprehensive report containing:
      | Field                               | Included |
      | Exact start/end times for every day | Yes      |
      | Log of all overtime and night work  | Yes      |
      | Full history of all leave taken     | Yes      |
      | Proof of respected rest periods     | Yes      |

  Scenario: Invalid Contract Type Operation
    Given an employee is hired under an "Umowa o dzieło"
    When I attempt to assign overtime hours to this employee
    Then the system must block the action
    And display an error: "Time tracking and overtime are not applicable for Umowa o dzieło."

  Scenario: Sunday Work Rule Violation
    Given an employee has already worked for the last "3 consecutive Sundays"
    When I attempt to schedule them for a shift on the upcoming Sunday
    Then the system must raise a compliance alert: "Violation: Employee must have at least one Sunday off in every 4-week period."

  Scenario: Data Retention Policy Verification
    When I view the system's data management policies
    Then I must see that the data retention policy is set to "10 years" post-termination
    And the system must prevent the purging of any employee records before this period expires
```