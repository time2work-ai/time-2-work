 Feature: Employee Profile and Contract Management
  This feature covers the creation and configuration of employee profiles, ensuring all calculations are based
   on their specific, legally-defined attributes.

    1 Feature: Employee Profile and Contract Management
    2
    3   Background:
    4     Given I am logged in as an Admin
    5
    6   Scenario: 1 - Standard Employee Onboarding (Over 10 Years Tenure)
    7     When I create a new employee with a "University Degree" and "3 years" of prior work
      experience
    8     Then the system should calculate their total work tenure as "11 years"
    9     And their annual leave entitlement should be "26 days"
   10
   11   Scenario: 2 - Junior Employee Onboarding (Under 10 Years Tenure)
   12     When I create a new employee who graduated from "Secondary School" with "0 years" of prior
      work experience
   13     Then the system should calculate their total work tenure as "4 years"
   14     And their annual leave entitlement should be "20 days"
   15
   16   Scenario: 3 - First-Time Employee Onboarding
   17     When I create a new employee and mark them as a "first-time employee"
   18     Then their leave entitlement should be calculated as "1/12th of their annual limit" for each
      month worked in the first year
   19
   20   Scenario: 4 - Part-Time Employee Configuration
   21     Given an employee has a total work tenure of "11 years"
   22     When I configure their contract as "Umowa o pracę" on a "0.5" (half-time) basis
   23     Then their annual leave entitlement should be proportionally calculated as "13 days"
   24
   25   Scenario: 5 - Configuring a Task-Based Work System (Zadaniowy)
   26     When I assign an employee to the "zadaniowy" (task-based) work system
   27     Then their time tracking interface should be simplified, disabling start/end hour logging
   28     And overtime calculations for this employee should be disabled
   29
   30   Scenario: 6 - Configuring an Equivalent Work System (Równoważny)
   31     When I assign an employee to the "równoważny" (equivalent) work system with a "3 month"
      settlement period
   32     Then the system must validate that their scheduled hours average no more than "40" per week
      over the period
   33     And the system should allow scheduling daily work up to "12 hours"
   34
   35   Scenario: 7 - Configuring a Disabled Employee
   36     When I update an employee's profile to a "moderate" degree of disability
   37     Then their daily work norm must be reduced to "7 hours"
   38     And "10 extra days" must be added to their annual leave entitlement
   39     And the system must block assigning them overtime or night work
   40
   41   Scenario: 8 - Configuring a Pregnant Employee
   42     When I update an employee's profile status to "pregnant"
   43     Then the system must immediately block any assignment of overtime or night work
   44     And the system must flag them as having priority for remote work requests

  Feature: Daily Time and Compliance Tracking
  This feature covers how employees and admins interact with the system for daily time recording and how the
   system enforces compliance rules.

    1 Feature: Daily Time and Compliance Tracking
    2
    3   Scenario: 9 - Standard Clock-In / Clock-Out
    4     Given I am logged in as an Employee on a "podstawowy" work system
    5     When I clock in at "08:02" and clock out at "16:05"
    6     Then the system should record my total work time as "8 hours and 3 minutes"
    7
    8   Scenario: 10 - Logging Overtime
    9     Given I am logged in as an Employee on a "podstawowy" work system
   10     When I log my work from "09:00" to "18:30" with a "30 minute" unpaid break
   11     Then the system should categorize "8 hours" as standard time
   12     And the system should categorize "1 hour" as overtime to be compensated at "150%"
   13
   14   Scenario: 11 - Manual Timesheet Entry
   15     Given I am logged in as an Employee
   16     When I manually submit a timesheet for yesterday with hours "08:00" to "16:00"
   17     Then the entry should be flagged as "Pending Approval" for my manager
   18     And the entry should be clearly marked as a "Manual Entry" in the audit log
   19
   20   Scenario: 12 - Mandatory Rest Period Violation
   21     Given I am logged in as an Admin
   22     And an employee has a shift ending at "22:00" on Monday
   23     When I attempt to schedule a new shift for that employee starting at "07:00" on Tuesday
   24     Then the system must reject the schedule
   25     And the system must display a critical error: "Violation of the 11-hour mandatory daily rest
      period."
   26
   27   Scenario: 13 - Night Work Logging and Compensation
   28     Given I am logged in as an Admin
   29     When I approve a timesheet for an employee who worked from "22:00" to "06:00"
   30     Then the system must identify all "8 hours" as night work
   31     And the system must automatically calculate the required pay supplement for those hours

  Feature: Leave Management
  This feature covers the entire lifecycle of leave requests, from application to approval and final pay-out
   on termination.

    1 Feature: Leave Management
    2
    3   Scenario: 14 - Requesting Annual Leave
    4     Given I am logged in as an Employee with "15 days" of remaining annual leave
    5     When I request "5 days" of annual leave for next month
    6     Then the request should be sent to my manager for approval
    7     And my available leave balance should show "10 days" with "5 days pending"
    8
    9   Scenario: 15 - Requesting "Leave on Demand"
   10     Given I am logged in as an Employee
   11     And I have used "1" day of "Leave on Demand" this year
   12     When I submit a request for "Leave on Demand" for today
   13     Then the system should automatically approve the request
   14     And my annual leave balance should be reduced by "1 day"
   15     And my "Leave on Demand" counter should update to "2"
   16
   17   Scenario: 16 - Insufficient Leave Balance
   18     Given I am logged in as an Employee with "3 days" of remaining annual leave
   19     When I attempt to request "5 days" of annual leave
   20     Then the system must reject the request
   21     And display an error: "Insufficient leave balance. Available: 3, Requested: 5."
   22
   23   Scenario: 17 - Parental Leave Chain Management
   24     Given I am logged in as an Admin for a new mother
   25     When I initiate the parental leave process
   26     Then the system must guide me to first apply "20 weeks" of "urlop macierzyński"
   27     And then allow transitioning to "41 weeks" of "urlop rodzicielski"
   28     And the system must track the "9 non-transferable weeks" for the other parent separately
   29
   30   Scenario: 18 - Past-Due Leave Alert
   31     Given I am logged in as an Admin
   32     And the current date is "August 1, 2026"
   33     When I view the compliance dashboard
   34     Then the system must display a critical alert listing all employees with unused leave from
      "2025"
   35     And the alert must state the final deadline is "September 30, 2026"
   36
   37   Scenario: 19 - Employee Termination and Leave Pay-out
   38     Given I am logged in as an Admin
   39     And an employee with "10 days" of unused annual leave is being terminated
   40     When I execute the final payroll process for this employee
   41     Then the system must automatically calculate the monetary pay-out for the "10 days"
   42     And the calculation must use the official "ekwiwalent" coefficient, not a simple daily rate

  Feature: Data Corrections and System Auditing
  This feature ensures that all data can be corrected with a full audit trail.

    1 Feature: Data Corrections and System Auditing
    2
    3   Background:
    4     Given I am logged in as an Admin
    5
    6   Scenario: 20 - Correcting a Timesheet Entry
    7     Given an employee's approved timesheet from last week shows "8 hours" worked
    8     When I correct the entry to "9 hours" and provide "Forgot to log overtime" as the reason
    9     Then the system must recalculate "1 hour" of overtime for that day
   10     And the change must be recorded in a permanent audit log with the reason and my user ID
   11
   12   Scenario: 21 - Cancelling an Approved Leave Request
   13     Given an employee has an approved leave request for "5 days" next week
   14     When I cancel the leave request on behalf of the employee
   15     Then the "5 days" must be immediately returned to their annual leave balance
   16
   17   Scenario: 22 - Historical Data Import
   18     When I import a set of historical work records for an employee from 2 years ago
   19     Then the system must update the employee's total work tenure calculation
   20     But the system must not alter any recent, approved timesheets

  Feature: Compliance, Reporting, and Legal Edge Cases
  This feature ensures the system can produce compliant reports and handles complex legal edge cases
  correctly.

    1 Feature: Compliance, Reporting, and Legal Edge Cases
    2
    3   Background:
    4     Given I am logged in as an Admin
    5
    6   Scenario: 23 - Generating a PIP Audit Report
    7     When I request a "PIP Audit Report" for an employee for the last "3 years"
    8     Then the system must generate a single, comprehensive report containing:
    9       | Field                               | Included |
   10       | Exact start/end times for every day | Yes      |
   11       | Log of all overtime and night work  | Yes      |
   12       | Full history of all leave taken     | Yes      |
   13       | Proof of respected rest periods     | Yes      |
   14
   15   Scenario: 24 - Invalid Contract Type Operation
   16     Given an employee is hired under an "Umowa o dzieło"
   17     When I attempt to assign overtime hours to this employee
   18     Then the system must block the action
   19     And display an error: "Time tracking and overtime are not applicable for Umowa o dzieło."
   20
   21   Scenario: 25 - Sunday Work Rule Violation
   22     Given an employee has already worked for the last "3 consecutive Sundays"
   23     When I attempt to schedule them for a shift on the upcoming Sunday
   24     Then the system must raise a compliance alert: "Violation: Employee must have at least one
      Sunday off in every 4-week period."
   25
   26   Scenario: 26 - Data Retention Policy Verification
   27     When I view the system's data management policies
   28     Then I must see that the data retention policy is set to "10 years" post-termination
   29     And the system must prevent the purging of any employee records before this period expires
