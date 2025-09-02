Time2Work is a streamlined compliance platform that automates the most critical aspects of Polish labor
  law. It focuses on the essential workflows that businesses face every day, ensuring accuracy and legal
  safety by mastering the fundamentals of time, leave, and compliance.

  This overview describes the core end-to-end journey that represents the heart of the Time2Work experience.

  ---

 ## The Essential Application Workflow

  The entire lifecycle of an employee's time and attendance is handled in a simple, automated, and compliant
   flow.

###  1. It Starts with a Smart Setup
  The process begins when an administrator onboards a new employee. Time2Work instantly analyzes their work
  history and education to automatically assign the correct annual leave entitlement—20 or 26 days. This
  eliminates manual calculation and ensures the foundational element of their employment is legally
  compliant from day one.
  (Based on Scenario: Standard Employee Onboarding)

###  2. Daily Work is Captured Intelligently
  An employee simply tracks their hours. If they work late, Time2Work automatically detects the extra time,
  calculates the overtime, and applies the correct 150% or 200% pay supplement. There is no need for manual
  timesheet adjustments or complex calculations; the system handles it correctly every time.
  (Based on Scenario: Logging Overtime)

###  3. Requesting Time Off is Simple
  When an employee needs time off, they request vacation directly in the app. The system checks their
  available balance and sends the request to their manager for a simple, one-click approval. This
  streamlines a common HR process for both employees and managers.
  (Based on Scenario: Requesting Annual Leave)

###  4. The System Acts as an Intelligent Guardian
  Time2Work doesn't just record data; it actively protects the business. If a manager attempts to create a
  schedule that violates the mandatory 11-hour rest period between shifts, the system will actively block
  the action and explain the compliance issue, preventing costly legal mistakes before they can happen.
  (Based on Scenario: Mandatory Rest Period Violation)

###  5. Everything Leads to Being Instantly Audit-Ready
  Ultimately, all of this meticulous, automated record-keeping culminates in one powerful feature: the
  one-click PIP Audit Report. At any moment, an administrator can generate a perfectly formatted, legally
  compliant report that proves adherence to labor laws, providing complete peace of mind and readiness for
  any government inspection.
  (Based on Scenario: Generating an Audit-Ready Report)

  ---

 ## Core Value Proposition

  By focusing on these five critical scenarios, Time2Work delivers:

   * Automated & Accurate Setup: Guarantees correct leave entitlements from the start.
   * Simplified Daily Tracking: Captures work and calculates complex overtime automatically.
   * Built-in Compliance Guardrails: Actively prevents illegal scheduling and work patterns.
   * Instant, Audit-Ready Reporting: Provides proof of compliance on demand.

  By mastering these core workflows, Time2Work makes guaranteed compliance and operational efficiency
  accessible to every business.


## Time2Work: Core (80/20) Application Specification

  Version: 0.8 (MVP)
  Date: September 2, 2025

  ## 1. Introduction

  1.1. Purpose of this Document
  This document provides the functional and non-functional specifications for the core version of the
  Time2Work application. It is designed to guide the development of a Minimum Viable Product (MVP) that
  delivers on the 80/20 principle: implementing the 20% of features that solve 80% of the most critical user
   needs for time and attendance compliance in Poland.

  1.2. Application Overview
  Time2Work is a streamlined compliance platform that automates the most critical aspects of Polish labor
  law. It focuses on the essential workflows that businesses face every day, ensuring accuracy and legal
  safety from day one by mastering the fundamentals of time, leave, and compliance.

  1.3. Scope
  The scope of this specification is strictly limited to the functionality required to fulfill the 5 core
  workflow scenarios.

  ### In Scope:
   * Onboarding employees under a standard contract (Umowa o pracę).
   * Calculating leave entitlement based on tenure.
   * Daily time tracking for the podstawowy (basic) work system.
   * Automatic overtime calculation.
   * Requesting and approving standard annual leave.
   * Validation of the mandatory daily rest period.
   * Generation of a core PIP audit report.

  ### Out of Scope (for this version):
   * The other 7 work systems (e.g., równoważny, zadaniowy).
   * Advanced leave types (parental, on-demand, circumstantial).
   * Special employee statuses (disability, pregnancy, minors).
   * Part-time or other non-standard contract configurations.
   * Advanced compliance checks (weekly rest, Sunday work rules).
   * Data corrections, timesheet approvals, and advanced integrations.

  ---

  ## 2. User Roles & Permissions (Core)

   * Employee: Can track their daily work time and request annual leave. Can view their own timesheets and
     leave balance.
   * Administrator: Can create and manage employee profiles. Can view all employee timesheets and approve
     annual leave requests. Can generate the PIP Audit Report.

  ---

  ## 3. Core Functional Requirements (MVP)

  3.1. Module: Employee Profile Management

   * 3.1.1. Core Data Model: An employee profile must contain the following essential fields:
       * Employment Details: typ_umowy (fixed to 'Umowa o pracę' for MVP).
       * Tenure & Education: poziom_wyksztalcenia, lata_stazu_pracy.

   * 3.1.2. Automated Leave Entitlement Calculation: Upon profile creation, the system must automatically
     calculate the employee's annual leave entitlement based on their total tenure.
       * Logic: Tenure is calculated from lata_stazu_pracy + years granted for poziom_wyksztalcenia.
           * < 10 years tenure = 20 days
           * >= 10 years tenure = 26 days
       * (Ref: BDD Scenario "Standard Employee Onboarding")

  3.2. Module: Time & Attendance Tracking

   * 3.2.1. Time Entry: Employees must be able to log their start and end times for each workday.
   * 3.2.2. Work System Support: This version of the system will exclusively support the `podstawowy` (basic)
     work system, defined as an 8-hour workday and a 40-hour average work week.
   * 3.2.3. Overtime Calculation: The system must automatically detect and calculate overtime for hours worked
      beyond the 8-hour daily norm.
       * Logic: The system will apply a 150% pay supplement for the first two hours of overtime on a weekday
         and 200% for subsequent hours.
       * (Ref: BDD Scenario "Logging Overtime")

  3.3. Module: Leave Management

   * 3.3.1. Leave Request Workflow:
       * An employee can submit a request for Urlop Wypoczynkowy (Annual Leave).
       * The request must be sent to an Administrator for approval.
       * The employee's leave balance must be checked to prevent requesting more days than available.
       * (Ref: BDD Scenario "Requesting Annual Leave")

   * 3.3.2. Leave Type Support: This version will exclusively support Annual Leave. All other leave types are
     out of scope.

  3.4. Module: Compliance & Reporting

   * 3.4.1. Real-time Compliance Validation: The system must include a validation engine that actively
     prevents scheduling that violates core labor laws.
       * Logic: The system must block any attempt to schedule an employee for a shift that does not allow for
         an uninterrupted 11-hour rest period since their last shift ended.
       * (Ref: BDD Scenario "Mandatory Rest Period Violation")

   * 3.4.2. PIP Audit Report: The system must be able to generate a report for a single employee over a
     specified date range that is compliant with PIP requirements.
       * Content: The report must include a chronological log of:
           1. Workdays with exact start and end times.
           2. All calculated overtime hours.
           3. All approved annual leave days taken.
           4. Data sufficient to prove the 11-hour rest period was respected.
       * (Ref: BDD Scenario "Generating an Audit-Ready Report")

  ---

  ## 4. Non-Functional Requirements (Core)

   * 4.1. Data Integrity: All recorded events (time entries, leave requests) must be stored accurately and
     reliably.
   * 4.2. Data Retention: All time and attendance data must be stored for a minimum of 10 years
     post-employment.
   * 4.3. Security: A simple Role-Based Access Control must be in place: Employees can only access their own
     data; Administrators can access all data.

  ---
  ## 5. Appendix

  5.1. Referenced Core Scenarios
   1. Standard Employee Onboarding (Over 10 Years Tenure)
   2. Logging Overtime
   3. Requesting Annual Leave
   4. Mandatory Rest Period Violation
   5. Generating a PIP Audit Report
