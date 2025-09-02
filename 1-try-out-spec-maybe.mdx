 ## Time2Work: Full Application Specification

  Version: 1.0
  Date: September 2, 2025

 ### 1. Introduction

  1.1. Purpose of this Document
  This document provides the complete functional and non-functional specification for the Time2Work
  application. It is intended to serve as the single source of truth for project managers, developers, and
  QA engineers during the application's development lifecycle. All requirements herein are derived from the
  source documents.

  1.2. Application Overview
  Time2Work is an intelligent time and attendance platform meticulously designed to navigate the
  complexities of Polish labor law. It automates and simplifies every aspect of work time evidence and leave
   management, replacing manual calculations and legal guesswork with guaranteed compliance and peace of
  mind.

  1.3. Scope
  The scope of this document covers all four planned phases of development, outlining the complete vision
  for the application from its Minimum Viable Product (MVP) to a full-featured, intelligent platform.

  1.4. User Roles & Permissions
  The system will support two primary user roles:

   * Employee: Can track their own work time, view their schedules and leave balances, submit leave requests,
     and make manual timesheet entries for approval. Access is restricted to their own personal data.
   * Administrator (HR/Manager): Has full oversight. Can manage all employee profiles, configure contracts and
      work systems, approve/deny requests, correct data entries, and generate system-wide compliance reports.

  ---

 ### 2. Core Functional Requirements

  This section details the application's features, which are broken down into logical modules.

  2.1. Module: Employee Profile Management

  This module is the foundation of the system. The accuracy of all other modules depends on the correct
  configuration here.

   * 2.1.1. Employee Data Model: Each employee profile must contain all attributes necessary to determine
     their legal status, as defined in the ProfilPracownika model. Key fields include:
       * Employment Details: typ_umowy (contract type), data_zatrudnienia (hire date), wymiar_etatu
         (FTE/part-time status).
       * Tenure & Education: poziom_wyksztalcenia (education level), lata_stazu_pracy (years of work
         experience).
       * Work Configuration: system_czasu_pracy (work system), okres_rozliczeniowy (settlement period).
       * Special Statuses: czy_w_ciazy (is pregnant), stopien_niepelnosprawnosci (disability degree),
         data_urodzenia (for age-related restrictions).
       * Parental Status: Details of children to manage related leave entitlements.

   * 2.1.2. Automated Leave Entitlement Calculation: The system must automatically calculate an employee's
     annual leave entitlement upon profile creation or update.
       * Logic: The calculation is based on total tenure (work years + years for education).
           * < 10 years tenure = 20 days
           * >= 10 years tenure = 26 days
       * (Ref: BDD Scenarios 1, 2)

   * 2.1.3. Proportional & First-Year Calculations:
       * For part-time employees, the leave entitlement must be calculated proportionally to their FTE status,
          rounded up to the nearest full day. (Ref: BDD Scenario 4)
       * For employees in their first-ever job, leave is accrued at a rate of 1/12th of the annual entitlement
          for each month worked. (Ref: BDD Scenario 3)

   * 2.1.4. Special Status Protections: When an employee's status is updated, the system must automatically
     enforce legal protections.
       * Pregnancy: Block assignment of overtime and night work. (Ref: BDD Scenario 8)
       * Disability: Reduce work norms (to 7h/day), add 10 days of leave, and block overtime. (Ref: BDD
         Scenario 7)

 ### 2.2. Module: Time & Attendance Tracking

   * 2.2.1. Time Entry:
       * Employees can clock in/out via the web or mobile interface.
       * Employees can submit manual timesheets for manager approval. (Ref: BDD Scenarios 9, 11)

   * 2.2.2. Overtime & Night Work:
       * The system must automatically detect and categorize hours worked beyond the employee's daily norm as
         overtime.
       * Overtime pay supplements must be calculated at 150% for the first two hours and 200% thereafter on
         weekdays.
       * Work performed during the employer-defined 8-hour night window (e.g., 22:00-06:00) must receive a 20%
          supplement.
       * (Ref: BDD Scenarios 10, 13)

   * 2.2.3. Work System Logic: The system must support all 8 Polish work time systems. The validation engine
     must apply the correct rules for each:


  ┌────────────────┬─────────────────────────────────────────────────────────────────────────────────┐
  │ System         │ Key Rules                                                                       │
  ├────────────────┼─────────────────────────────────────────────────────────────────────────────────┤
  │ Podstawowy     │ 8h/day, 40h/week average.                                                       │
  │ Równoważny     │ Up to 12h/day (or 16h/24h in specific cases), balanced to 40h/week in settle... │
  │ Zadaniowy      │ No hour tracking, only presence/absence. No overtime.                           │
  │ Ruchomy        │ Flexible start/end times, often with core hours.                                │
  │ Przerywany     │ One unpaid break of up to 5 hours per day.                                      │
  │ Ruch Ciągły    │ 43h/week average norm for continuous production/services.                       │
  │ Weekendowy     │ Work is performed primarily Friday-Sunday.                                      │
  │ **Skrócony Ty. │ 40h week completed in fewer than 5 days (e.g., 4x10h).                          │
  └────────────────┴─────────────────────────────────────────────────────────────────────────────────┘

 ### 2.3. Module: Leave Management

   * 2.3.1. Leave Workflow: All leave requests (except "on-demand") follow an approval workflow. An employee's
      balance is updated immediately upon final approval. (Ref: BDD Scenario 14)

   * 2.3.2. Leave Type Support: The system must manage all legally defined leave types with their specific
     rules:
       * Urlop Wypoczynkowy (Annual Leave): Standard vacation time.
       * Urlop na Żądanie (Leave on Demand): 4 days per year, taken from the annual leave pool, can be
         requested on the day of absence. (Ref: BDD Scenario 15)
       * Urlopy Rodzicielskie (Parental Leaves): Full support for the chain of Maternity, Paternity, Parental,
          and Child-rearing leaves with correct durations and pay-out rates. (Ref: BDD Scenario 17)
       * Urlopy Okolicznościowe (Circumstantial Leave): For events like weddings, funerals, etc.
       * Zwolnienie Lekarskie (Sick Leave): Tracking of medical leave.

   * 2.3.3. Termination & Payout (`Ekwiwalent`): Upon employee termination, the system must automatically
     calculate the monetary equivalent for all unused annual leave days using the legally mandated yearly
     coefficient. (Ref: BDD Scenario 19)

  2.4. Module: Compliance & Reporting

   * 2.4.1. Real-time Compliance Engine: The system must actively prevent the creation of schedules or
     approval of timesheets that violate the Polish Labor Code.
       * Checks: Must include the 11-hour daily rest period and the 35-hour weekly rest period.
       * Alerts: Must warn admins about approaching limits (e.g., annual overtime) or rule violations (e.g.,
         4th consecutive Sunday worked).
       * (Ref: BDD Scenarios 12, 25)

   * 2.4.2. PIP Audit Report: A feature must exist to generate a complete, audit-ready report for the National
      Labor Inspectorate (PIP) on demand, covering a specified date range. (Ref: BDD Scenario 23)

   * 2.4.3. Data Correction & Auditing: All corrections to approved data must require a reason and be stored
     in an immutable audit log, tracking the user, timestamp, and before/after values. (Ref: BDD Scenario 20)

  ---

 ### 3. Non-Functional Requirements

   * 3.1. Data Integrity: The system must be built on an event-sourced architecture, ensuring that no data is
     ever overwritten and a full history of changes is always preserved.
   * 3.2. Data Retention: All employee time-tracking and profile data must be stored for a minimum of 10 years
      following the calendar year of their termination. (Ref: BDD Scenario 26)
   * 3.3. Security: Access must be governed by a strict Role-Based Access Control (RBAC) model.
   * 3.4. Performance: The system must be architected to handle thousands of employees and daily transactions
     without a degradation in performance.

  ---

 ### 4. Development & Deployment Plan

  The development of Time2Work will follow a phased approach, delivering core value first and building upon
  it.

   * Phase 1: The Compliance Core (MVP): Focuses on the essential legal requirements for Umowa o pracę,
     including standard time/leave tracking, basic overtime, and rest period validation.
   * Phase 2: Advanced HR & Leave Management: Introduces the full suite of leave types, special employee
     status handling, and robust correction/approval workflows.
   * Phase 3: Full Operational Flexibility: Expands support to all 8 work systems and other contract types,
     making the platform universally applicable.
   * Phase 4: Intelligence & Integration: Delivers advanced analytics, predictive alerts, and a public API for
      integration with third-party Payroll and HR systems.
