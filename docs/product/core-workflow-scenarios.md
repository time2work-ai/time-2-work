## The 80/20 Core Workflow Scenarios

  ### 1. Foundational Setup: Onboarding a Standard Employee
  (This scenario validates the critical initial setup and the automatic calculation of leave, which affects
  the employee for their entire lifecycle.)

   1 Feature: Employee Profile Management
   2
   3   Scenario: Standard Employee Onboarding (Over 10 Years Tenure)
   4     Given I am logged in as an Admin
   5     When I create a new employee with a "University Degree" and "3 years" of prior work
     experience
   6     Then the system should calculate their total work tenure as "11 years"
   7     And their annual leave entitlement should be "26 days"

  ---
 ### 2. Daily Core Action: Tracking Work and Calculating Overtime
  (This scenario tests the most common and complex daily workflow: logging hours and ensuring the system can
   correctly differentiate standard time from overtime and apply the legally required pay supplements.)

   1 Feature: Daily Time and Compliance Tracking
   2
   3   Scenario: Logging Overtime
   4     Given I am logged in as an Employee on a "podstawowy" work system
   5     When I log my work from "09:00" to "18:30" with a "30 minute" unpaid break
   6     Then the system should categorize "8 hours" as standard time
   7     And the system should categorize "1 hour" as overtime to be compensated at "150%"

  ---
 ### 3. Common HR Process: Requesting Annual Leave
  (This scenario covers the most frequent employee-initiated HR action, testing the entire leave request and
   approval workflow, including balance validation.)

   1 Feature: Leave Management
   2
   3   Scenario: Requesting Annual Leave
   4     Given I am logged in as an Employee with "15 days" of remaining annual leave
   5     When I request "5 days" of annual leave for next month
   6     Then the request should be sent to my manager for approval
   7     And my available leave balance should show "10 days" with "5 days pending"

  ---
###  4. Critical System Value: Automated Compliance Violation Prevention
  (This scenario demonstrates the application's core value proposition: actively protecting the business by
  automatically enforcing complex legal rules, not just passively tracking data.)

   1 Feature: Daily Time and Compliance Tracking
   2
   3   Scenario: Mandatory Rest Period Violation
   4     Given I am logged in as an Admin
   5     And an employee has a shift ending at "22:00" on Monday
   6     When I attempt to schedule a new shift for that employee starting at "07:00" on Tuesday
   7     Then the system must reject the schedule
   8     And the system must display a critical error: "Violation of the 11-hour mandatory daily rest
     period."

  ---
 ### 5. Ultimate Business Outcome: Generating an Audit-Ready Report
  (This scenario validates the end-to-end integrity of the system. It proves that all correctly entered data
   can be aggregated and presented in a format that satisfies the highest level of scrutiny from a
  government body like the PIP.)

    1 Feature: Compliance, Reporting, and Legal Edge Cases
    2
    3   Scenario: Generating a PIP Audit Report
    4     Given I am logged in as an Admin
    5     When I request a "PIP Audit Report" for an employee for the last "3 years"
    6     Then the system must generate a single, comprehensive report containing:
    7       | Field                               | Included |
    8       | Exact start/end times for every day | Yes      |
    9       | Log of all overtime and night work  | Yes      |
   10       | Full history of all leave taken     | Yes      |
   11       | Proof of respected rest periods     | Yes      |
