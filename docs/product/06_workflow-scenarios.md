---
title: "Time2Work: Core Workflow Scenarios (80/20)"
description: "Priority scenarios representing 80% of the most critical user workflows"
version: "1.0"
date: "2025-09-02"
type: "Core Scenarios"
audience: "Product Managers, Developers, QA Engineers"
---

# Time2Work: Core Workflow Scenarios (80/20)

## Overview

These scenarios represent the 20% of features that solve 80% of the most critical user needs for time and attendance compliance in Poland.

## Table of Contents

- [1. Foundational Setup](#1-foundational-setup)
- [2. Daily Core Action](#2-daily-core-action)
- [3. Common HR Process](#3-common-hr-process)
- [4. Critical System Value](#4-critical-system-value)
- [5. Ultimate Business Outcome](#5-ultimate-business-outcome)

## 1. Foundational Setup

### Onboarding a Standard Employee

This scenario validates the critical initial setup and the automatic calculation of leave, which affects the employee for their entire lifecycle.

```gherkin
Feature: Employee Profile Management

  Scenario: Standard Employee Onboarding (Over 10 Years Tenure)
    Given I am logged in as an Admin
    When I create a new employee with a "University Degree" and "3 years" of prior work experience
    Then the system should calculate their total work tenure as "11 years"
    And their annual leave entitlement should be "26 days"
```

## 2. Daily Core Action

### Tracking Work and Calculating Overtime

This scenario tests the most common and complex daily workflow: logging hours and ensuring the system can correctly differentiate standard time from overtime and apply the legally required pay supplements.

```gherkin
Feature: Daily Time and Compliance Tracking

  Scenario: Logging Overtime
    Given I am logged in as an Employee on a "podstawowy" work system
    When I log my work from "09:00" to "18:30" with a "30 minute" unpaid break
    Then the system should categorize "8 hours" as standard time
    And the system should categorize "1 hour" as overtime to be compensated at "150%"
```

## 3. Common HR Process

### Requesting Annual Leave

This scenario covers the most frequent employee-initiated HR action, testing the entire leave request and approval workflow, including balance validation.

```gherkin
Feature: Leave Management

  Scenario: Requesting Annual Leave
    Given I am logged in as an Employee with "15 days" of remaining annual leave
    When I request "5 days" of annual leave for next month
    Then the request should be sent to my manager for approval
    And my available leave balance should show "10 days" with "5 days pending"
```

## 4. Critical System Value

### Automated Compliance Violation Prevention

This scenario demonstrates the application's core value proposition: actively protecting the business by automatically enforcing complex legal rules, not just passively tracking data.

```gherkin
Feature: Daily Time and Compliance Tracking

  Scenario: Mandatory Rest Period Violation
    Given I am logged in as an Admin
    And an employee has a shift ending at "22:00" on Monday
    When I attempt to schedule a new shift for that employee starting at "07:00" on Tuesday
    Then the system must reject the schedule
    And the system must display a critical error: "Violation of the 11-hour mandatory daily rest period."
```

## 5. Ultimate Business Outcome

### Generating an Audit-Ready Report

This scenario validates the end-to-end integrity of the system. It proves that all correctly entered data can be aggregated and presented in a format that satisfies the highest level of scrutiny from a government body like the PIP.

```gherkin
Feature: Compliance, Reporting, and Legal Edge Cases

  Scenario: Generating a PIP Audit Report
    Given I am logged in as an Admin
    When I request a "PIP Audit Report" for an employee for the last "3 years"
    Then the system must generate a single, comprehensive report containing:
      | Field                               | Included |
      | Exact start/end times for every day | Yes      |
      | Log of all overtime and night work  | Yes      |
      | Full history of all leave taken     | Yes      |
      | Proof of respected rest periods     | Yes      |
```

## Core Value Proposition

By focusing on these five critical scenarios, Time2Work delivers:

- **Automated & Accurate Setup**: Guarantees correct leave entitlements from the start
- **Simplified Daily Tracking**: Captures work and calculates complex overtime automatically
- **Built-in Compliance Guardrails**: Actively prevents illegal scheduling and work patterns
- **Instant, Audit-Ready Reporting**: Provides proof of compliance on demand

By mastering these core workflows, Time2Work makes guaranteed compliance and operational efficiency accessible to every business.