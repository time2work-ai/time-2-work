---
title: "Time2Work: Acceptance Criteria"
description: "Comprehensive user story scenarios for Time2Work validation"
version: "1.0"
date: "2025-09-02"
type: "User Story Scenarios"
audience: "Product Managers, QA Engineers, Stakeholders"
---

# Time2Work: Acceptance Criteria

## Purpose

Define ALL possible real-life work time and leave management scenarios from both Admin and Employee perspectives for the validation of the Work Time Evidence system, ensuring full compliance with the Polish Labor Code as detailed in the source document.

## Format

Story-driven scenarios with expected system behavior.

## Table of Contents

- [1. Employee & Contract Configuration](#1-employee--contract-configuration)
- [2. Daily Time Tracking](#2-daily-time-tracking)
- [3. Leave Management](#3-leave-management)
- [4. Corrections & Error Handling](#4-corrections--error-handling)
- [5. Compliance, Reporting & Edge Cases](#5-compliance-reporting--edge-cases)

## 1. Employee & Contract Configuration (Admin Perspective)

This section covers the initial setup of employee profiles, which is critical for all subsequent calculations.

### Scenario 1: Standard Employee Onboarding (Over 10 Years Tenure)

> **As an Admin**, I need to create a new employee hired under a standard Umowa o pracę (contract of employment) with a university degree (adds 8 years to tenure) and 3 years of previous work experience. I expect the system to automatically calculate their total work tenure as 11 years and assign them an annual leave entitlement of 26 days.

### Scenario 2: Junior Employee Onboarding (Under 10 Years Tenure)

> **As an Admin**, I need to create a new employee hired under a standard Umowa o pracę who just graduated from a secondary school (adds 4 years to tenure) and has no prior work experience. I expect the system to calculate their tenure as 4 years and assign them an annual leave entitlement of 20 days.

### Scenario 3: First-Time Employee Onboarding

> **As an Admin**, I need to onboard an employee for their first-ever job. I expect the system to flag them as a "first-time employee" and calculate their leave entitlement as 1/12th of their annual limit for each month worked during their first calendar year.

### Scenario 4: Part-Time Employee Configuration

> **As an Admin**, I need to configure an employee on a 0.5 (half-time) Umowa o pracę with over 10 years of tenure. I expect the system to calculate their annual leave entitlement proportionally as 13 days (26 × 0.5), rounded up.

### Scenario 5: Configuring a Task-Based Work System (Zadaniowy)

> **As an Admin**, I need to assign an employee (e.g., a Field Salesperson) to the zadaniowy (task-based) work system. I expect the system to automatically switch their time tracking interface to an "uproszczona" (simplified) view, disabling the need to log specific start/end hours and removing overtime calculations.

### Scenario 6: Configuring an Equivalent Work System (Równoważny)

> **As an Admin**, I need to assign a warehouse worker to the równoważny (equivalent) work system, allowing them to work up to 12 hours a day. I expect the system to validate their schedule to ensure their average weekly work time does not exceed 40 hours within the defined settlement period (e.g., 3 months).

### Scenario 7: Configuring a Disabled Employee

> **As an Admin**, I need to update an employee's profile to reflect a 'moderate' degree of disability. I expect the system to automatically:
>
> 1. Reduce their daily work norm to 7 hours and weekly to 35 hours.
> 2. Add 10 extra days to their annual leave entitlement.
> 3. Block the ability to assign them overtime or night work.

### Scenario 8: Configuring a Pregnant Employee

> **As an Admin**, I need to update an employee's profile to 'pregnant'. I expect the system to immediately and automatically:
>
> 1. Block any possibility of assigning them overtime or night work.
> 2. Flag them as having priority for remote work requests.
> 3. Protect their profile from termination actions.

## 2. Daily Time Tracking (Employee & Admin Perspective)

### Scenario 9: Standard Clock-In / Clock-Out (Employee)

> **As an Employee** on a podstawowy (basic) 8-hour system, I need to clock in at 08:02 and clock out at 16:05. I expect the system to record my total time and show exactly 8 hours of standard work time.

### Scenario 10: Logging Overtime (Employee)

> **As an Employee** on a podstawowy system, I need to clock in at 09:00 and clock out at 18:30, taking a 30-minute unpaid break. I expect the system to record 9 hours of work, automatically categorizing 8 hours as standard time and 1 hour as overtime to be compensated at 150%.

### Scenario 11: Manual Timesheet Entry (Employee)

> **As an Employee** who forgot to clock in/out, I need to manually enter my work time for yesterday as 08:00 to 16:00. I expect the entry to be submitted for my manager's approval and clearly marked as a "manual entry".

### Scenario 12: Mandatory Rest Period Violation (Admin)

> **As an Admin**, I need to attempt to schedule an employee for a shift ending at 22:00 and a new shift starting at 07:00 the next day. I expect the system to reject the schedule with a critical error: "Violation of the 11-hour mandatory daily rest period."

### Scenario 13: Night Work Logging (Admin)

> **As an Admin**, I need to approve a timesheet for an employee who worked from 22:00 to 06:00. I expect the system to automatically identify all 8 hours as praca nocna (night work) and calculate the appropriate pay supplement (20% of minimum wage per hour).

## 3. Leave Management (Employee & Admin Perspective)

### Scenario 14: Requesting Annual Leave (Employee)

> **As an Employee**, I need to request 5 days of urlop wypoczynkowy (annual leave) for next month. I expect the system to show my remaining leave balance, submit the request to my manager for approval, and put a temporary hold on those 5 days from my balance.

### Scenario 15: Requesting "Leave on Demand" (Employee)

> **As an Employee**, I need to report an absence for today using urlop na żądanie (leave on demand). I expect the system to let me submit this request on the same day, automatically deduct 1 day from my annual leave pool, and increment my "on-demand" leave counter (max 4 per year).

### Scenario 16: Leave Request for Insufficient Balance (Employee)

> **As an Employee** with 3 days of leave left, I need to attempt to request 5 days of leave. I expect the system to reject the request with an error: "Insufficient leave balance. Available: 3, Requested: 5."

### Scenario 17: Parental Leave Chain (Admin)

> **As an Admin**, I need to process leave for a new mother. I expect the system to guide me through the correct sequence:
>
> 1. Start with 20 weeks of urlop macierzyński (maternity leave) at 100% pay.
> 2. Allow transitioning to 41 weeks of urlop rodzicielski (parental leave) at 70% pay (or 81.5% if requested early).
> 3. Ensure the non-transferable 9 weeks for the other parent are correctly tracked.

### Scenario 18: Forgetting to Use Past-Due Leave (Admin)

> **As an Admin**, I need to run a report on August 1st, 2026. I expect the system to generate a critical alert for all employees who have not used their remaining leave from the year 2025, reminding me that the deadline is September 30th, 2026.

### Scenario 19: Employee Termination & Leave Pay-out (Admin)

> **As an Admin**, I need to process the termination of an employee who has 10 unused vacation days. I expect the system to automatically calculate the ekwiwalent za niewykorzystany urlop (monetary equivalent for unused leave) using the special yearly coefficient, not by simply dividing monthly salary by 30.

## 4. Corrections & Error Handling (Admin Perspective)

### Scenario 20: Correcting a Timesheet Entry

> **As an Admin**, I need to correct an employee's approved timesheet from last week; they worked 9 hours but it was recorded as 8. I expect to be able to edit the entry, provide a reason for the change, and have the system recalculate the overtime and store a full audit log of the correction.

### Scenario 21: Cancelling an Approved Leave Request

> **As an Admin**, I need to cancel an employee's approved leave for next week at their request. I expect the system to allow the cancellation and immediately return the days to their leave balance.

### Scenario 22: Historical Data Import

> **As an Admin**, I need to import time tracking data from a previous system for an employee, dated 2 years ago. I expect the system to accept the historical data and correctly update the employee's total work tenure without affecting recent timesheet calculations.

## 5. Compliance, Reporting & Edge Cases

### Scenario 23: Generating a PIP Audit Report (Admin)

> **As an Admin**, I need to respond to a Państwowa Inspekcja Pracy (National Labor Inspectorate) audit. I expect to be able to instantly generate a complete and compliant report for a specific employee covering the last 3 years, including:
>
> 1. Exact start/end times for every workday.
> 2. A log of all overtime, night work, and dyżur (on-call time).
> 3. A full history of all leave taken.
> 4. Proof of respected daily and weekly rest periods.

### Scenario 24: Contract Type Mismatch (Admin)

> **As an Admin**, I need to attempt to assign overtime to an employee on an Umowa o dzieło (contract for specific work). I expect the system to block this action with a clear error: "Time tracking and overtime are not applicable for Umowa o dzieło."

### Scenario 25: Sunday Work Rule (Admin)

> **As an Admin**, I need to schedule an employee for their 4th consecutive Sunday of work. I expect the system to raise a compliance alert: "Violation: Employee must have at least one Sunday off in every 4-week period."

### Scenario 26: Data Retention Policy (Admin)

> **As an Admin**, I need to verify that the system is configured to store all employee time-tracking records for at least 10 years from the end of the calendar year of their termination, and that this data cannot be purged before then.