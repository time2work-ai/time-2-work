---
title: "Time2Work: Full Application Specification"
description: "Complete functional and non-functional specification for the Time2Work platform"
version: "1.0"
date: "2025-09-02"
type: "Technical Specification"
audience: "Development Team, QA Engineers, Project Managers"
document_number: "04"
input: "03_technical_reference.md"
output: "Complete system requirements ready for implementation"
---

# Time2Work: Full Application Specification

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Core Functional Requirements](#2-core-functional-requirements)
- [3. Non-Functional Requirements](#3-non-functional-requirements)
- [4. Development & Deployment Plan](#4-development--deployment-plan)

## 1. Introduction

### 1.1. Purpose of this Document

This document provides the complete functional and non-functional specification for the Time2Work application. It is intended to serve as the single source of truth for project managers, developers, and QA engineers during the application's development lifecycle. All requirements herein are derived from the source documents.

### 1.2. Application Overview

Time2Work is an intelligent time and attendance platform meticulously designed to navigate the complexities of Polish labor law. It automates and simplifies every aspect of work time evidence and leave management, replacing manual calculations and legal guesswork with guaranteed compliance and peace of mind.

### 1.3. Scope

The scope of this document covers all four planned phases of development, outlining the complete vision for the application from its Minimum Viable Product (MVP) to a full-featured, intelligent platform.

### 1.4. User Roles & Permissions

The system will support two primary user roles:

- **Employee**: Can track their own work time, view their schedules and leave balances, submit leave requests, and make manual time entries for approval. Access is restricted to their own personal data.
- **Administrator (HR/Manager)**: Has full oversight. Can manage all employee profiles, configure contracts and work systems, approve/deny requests, correct data entries, and generate system-wide compliance reports.

## 2. Core Functional Requirements

This section details the application's features, which are broken down into logical modules.

### 2.1. Module: Employee Profile Management

This module is the foundation of the system. The accuracy of all other modules depends on the correct configuration here.

#### 2.1.1. Employee Data Model

Each employee profile must contain all attributes necessary to determine their legal status. Key fields include:

- **Employment Details**: contract_type (employment_contract, service_agreement, specific_work_contract, b2b), hire_date, fte_ratio (FTE/part-time status).
- **Personal Information**: birth_date (for age-related restrictions), sex (male/female - required for pregnancy validation per Art. 178 § 1 Labor Code).
- **Tenure & Education**: education_level, work_experience_years, education_bonus_years (calculated based on education level).
- **Work Configuration**: work_system (basic, equivalent, task_based, flexible, interrupted, continuous, weekend, shortened_week), settlement_period_months.
- **Special Statuses**: is_pregnant (only valid if sex == female), disability_level (none, moderate, severe), has_children_under_4.
- **Parental Status**: Details of children to manage related leave entitlements.

#### 2.1.2. Automated Leave Entitlement Calculation

The system must automatically calculate an employee's annual leave entitlement upon profile creation or update.

**Base Entitlement Logic**: 
- Total qualifying years = work_experience_years + education_bonus_years
- < 10 years tenure = 20 days annual leave
- >= 10 years tenure = 26 days annual leave
- Employees with disability (moderate or severe) = +10 additional days

**Education Bonus Years** (maximum values):
- Basic vocational: 3 years
- Secondary vocational: 5 years
- General secondary: 4 years
- Post-secondary: 6 years
- Higher education: 8 years

*(Ref: BDD Scenarios 1, 2)*

#### 2.1.3. Proportional & First-Year Calculations

- For part-time employees, the leave entitlement must be calculated proportionally to their FTE status, rounded up to the nearest full day. Formula: `ceil(annual_days * fte_ratio)` *(Ref: BDD Scenario 4)*
- For employees in their first-ever job, leave is accrued monthly at a rate of 1/12th of the annual entitlement for each month worked. Example: 20 days annual = 1.67 days per month worked. *(Ref: BDD Scenario 3)*
- For partial year employment: `ceil((annual_days * fte_ratio * months_worked) / 12)`

#### 2.1.4. Special Status Protections

When an employee's status is updated, the system must automatically enforce legal protections per Polish Labor Code.

- **Pregnancy** (Art. 178 § 1 Labor Code): Block assignment of overtime and night work. Only applicable to female employees. *(Ref: BDD Scenario 8)*
- **Disability**: 
  - Moderate disability: Reduce daily work limit to 7 hours
  - Severe disability: Reduce daily work limit to 6 hours
  - All disability levels: Add 10 additional leave days, block mandatory overtime *(Ref: BDD Scenario 7)*
- **Minors** (under 18): Maximum 6 hours daily work, no overtime, no night work
- **Parents of children under 4**: Cannot be assigned night work without written consent

### 2.2. Module: Time & Attendance Tracking

#### 2.2.1. Time Entry

- Employees can clock in/out via the web or mobile interface.
- Employees can submit manual time entries for manager approval. *(Ref: BDD Scenarios 9, 11)*

#### 2.2.2. Overtime & Night Work

- The system must automatically detect and categorize hours worked beyond the employee's daily norm as overtime.
- Overtime compensation must be calculated according to Polish Labor Code:
  - **Regular workdays**: 50% supplement for the first two hours, 100% supplement for remaining hours
  - **Sundays/holidays/nights**: 100% supplement for all overtime hours
  - **Alternative**: Time off at 1:1 ratio if requested by employee, or 1.5:1 ratio if given by employer
- Work performed during the employer-defined 8-hour night window (e.g., 21:00-07:00) must receive a 20% supplement based on minimum wage hourly rate.

*(Ref: BDD Scenarios 10, 13)*

#### 2.2.3. Work System Logic

The system must support all 8 Polish work time systems. The validation engine must apply the correct rules for each:

| System | Key Rules | Polish Name |
|--------|-----------|-------------|
| basic | 8h/day, 40h/week average | podstawowy |
| equivalent | Up to 12h/day (or 16h/24h in specific cases), balanced to 40h/week in settlement period | równoważny |
| task_based | No hour tracking, only presence/absence. No overtime | zadaniowy |
| flexible | Flexible start/end times, often with core hours | ruchomy |
| interrupted | One unpaid break of up to 5 hours per day | przerywany |
| continuous | 43h/week average norm for continuous production/services | ruch ciągły |
| weekend | Work is performed primarily Friday-Sunday | weekendowy |
| shortened_week | 40h week completed in fewer than 5 days (e.g., 4x10h) | skrócony tydzień |

### 2.3. Module: Leave Management

#### 2.3.1. Leave Workflow

All leave requests (except "on-demand") follow an approval workflow. An employee's balance is updated immediately upon final approval. *(Ref: BDD Scenario 14)*

#### 2.3.2. Leave Type Support

The system must manage all legally defined leave types with their specific rules:

- **Annual Leave** (Polish: Urlop Wypoczynkowy): Standard vacation time.
- **Leave on Demand** (Polish: Urlop na Żądanie): 4 days per year, taken from the annual leave pool, can be requested on the day of absence. *(Ref: BDD Scenario 15)*
- **Parental Leaves** (Polish: Urlopy Rodzicielskie): Full support for the chain of Maternity, Paternity, Parental, and Child-rearing leaves with correct durations and pay-out rates. *(Ref: BDD Scenario 17)*
- **Circumstantial Leave** (Polish: Urlopy Okolicznościowe): For events like weddings (2 days), funerals (2 days for immediate family, 1 day for extended), etc.
- **Sick Leave** (Polish: Zwolnienie Lekarskie): Tracking of medical leave with appropriate compensation rates.

#### 2.3.3. Termination & Payout

Upon employee termination, the system must automatically calculate the monetary equivalent (Polish: Ekwiwalent) for all unused annual leave days using the legally mandated yearly coefficient formula. *(Ref: BDD Scenario 19)*

### 2.4. Module: Compliance & Reporting

#### 2.4.1. Real-time Compliance Engine

The system must actively prevent the creation of schedules or approval of time entries that violate the Polish Labor Code.

- **Checks**: Must include the 11-hour daily rest period and the 35-hour weekly rest period.
- **Alerts**: Must warn admins about approaching limits (e.g., annual overtime) or rule violations (e.g., 4th consecutive Sunday worked).

*(Ref: BDD Scenarios 12, 25)*

#### 2.4.2. PIP Audit Report

A feature must exist to generate a complete, audit-ready report for the National Labor Inspectorate (PIP) on demand, covering a specified date range. The system must be capable of generating this report within 5 minutes to comply with "immediate" (natychmiast) access requirements during inspections. *(Ref: BDD Scenario 23, Art. 149 Labor Code)*

The report must include:
- Complete work time records with exact start/end times
- Overtime calculations and compensation details
- Leave records with entitlement basis
- Compliance status for rest periods
- Any protection rule violations

#### 2.4.3. Data Correction & Auditing

All corrections to approved data must require a reason and be stored in an immutable audit log, tracking the user, timestamp, and before/after values. *(Ref: BDD Scenario 20)*

## 3. Non-Functional Requirements

### 3.1. Data Architecture

The system must be built on an event-sourced architecture following these principles:
- **Immutability**: No data is ever overwritten; all changes are recorded as new events
- **Audit Trail**: Complete history of all changes with timestamps, user IDs, and reasons
- **Event Replay**: Ability to reconstruct any historical state from events
- **CQRS Pattern**: Separate read and write models for optimal performance

### 3.2. Data Retention

All employee time-tracking and profile data must be stored for a minimum of 10 years following the calendar year of their termination per Art. 149 Labor Code. *(Ref: BDD Scenario 26)*
- Automatic archival process for terminated employees
- Encrypted long-term storage with integrity verification
- Compliance with GDPR right to erasure after retention period

### 3.3. Security

Access must be governed by a strict Role-Based Access Control (RBAC) model:
- Multi-factor authentication for administrative access
- Encrypted data at rest and in transit
- Session management with automatic timeout
- Comprehensive access logging for audit purposes

### 3.4. Performance

The system must meet these performance requirements:
- Generate PIP compliance reports within 5 minutes for any date range
- Handle 10,000+ concurrent employee clock-ins/outs
- Sub-second response time for all user interactions
- 99.9% uptime with zero-downtime deployments
- Real-time compliance validation without performance impact

### 3.5. Integration Requirements

- RESTful API for third-party integrations
- Webhook support for real-time event notifications
- Bulk data import/export capabilities
- Standard formats: JSON, CSV, XML

## 4. Development & Deployment Plan

The development of Time2Work will follow a phased approach, delivering core value first and building upon it.

### Phase 1: The Compliance Core (MVP)

**Focus**: Essential legal requirements for employment contracts (employment_contract type)
- Core employee profile management with all required fields
- Basic time tracking with clock in/out functionality
- Standard leave entitlement calculations (20/26 days)
- Overtime calculation (50%/100% supplements)
- Rest period validation (11h daily, 35h weekly)
- Basic compliance reports

### Phase 2: Advanced HR & Leave Management

**Focus**: Complete leave management and special protections
- All leave types (annual, on-demand, parental, circumstantial, sick)
- Special employee status protections (pregnancy, disability, minors)
- Leave request/approval workflows
- Data correction with audit trails
- PIP audit report generation (5-minute response)

### Phase 3: Full Operational Flexibility

**Focus**: All work systems and contract types
- Support for all 8 work systems (basic through shortened_week)
- Service agreements and specific work contracts
- Complex settlement period calculations
- Industry-specific rules (e.g., 24h shifts for security)
- Remote work management

### Phase 4: Intelligence & Integration

**Focus**: Advanced features and external connectivity
- Predictive compliance alerts
- Advanced analytics and dashboards
- RESTful API for third-party integrations
- Payroll system integration
- Mobile applications
- Machine learning for pattern detection