---
title: "Comprehensive Time Tracking Scenarios"
description: "Real-world user stories bridging business rules to technical implementation"
version: "1.0"
date: "2025-09-06"
type: "User Scenarios"
audience: "Product Managers, Development Team, QA Engineers"
document_number: "02"
input: "01_business_rules.md"
output: "User-centric scenarios ready for technical reference"
---

# Comprehensive Time Tracking Scenarios

This document presents real-world user stories that bridge the gap between extracted business rules and technical implementation. Each scenario is written from the user's perspective, showing how Time2Work handles complex Polish labor law requirements seamlessly.

## Table of Contents

1. [Employee Onboarding Stories](#employee-onboarding-stories)
2. [Daily Time Tracking Stories](#daily-time-tracking-stories)
3. [Leave Management Stories](#leave-management-stories)
4. [Special Protections Stories](#special-protections-stories)
5. [Work System Transitions](#work-system-transitions)
6. [Compliance and Reporting Stories](#compliance-and-reporting-stories)
7. [Edge Cases and Complex Scenarios](#edge-cases-and-complex-scenarios)

## Employee Onboarding Stories

### Story 1: Anna's First Job Journey

> **Anna**, a 22-year-old fresh university graduate, starts her first job ever at a tech company in Warsaw on March 1st, 2025.
> 
> **What happens in Time2Work:**
> 1. HR creates Anna's profile with:
>    - Birth date: 2003-01-15
>    - Education: Master's degree (adds 5 years to tenure calculation)
>    - Previous experience: None (first job)
>    - Contract type: Employment contract (umowa o pracę)
> 
> 2. System automatically:
>    - Calculates tenure: 5 years (education only)
>    - Assigns 20 days annual leave (tenure < 10 years)
>    - Flags as "first-time employee"
>    - Sets leave accrual: 1.67 days/month (20 days ÷ 12 months)
> 
> 3. Anna sees on her dashboard:
>    - Current leave balance: 0 days (will accrue monthly)
>    - Next accrual: 1.67 days on March 31st
>    - Projected year-end balance: 16.7 days (10 months × 1.67)

### Story 2: Marek's Senior Position with Complex History

> **Marek**, a 45-year-old engineer, joins a manufacturing company. He has 15 years of documented work experience and a moderate disability certificate.
> 
> **What happens in Time2Work:**
> 1. HR enters Marek's data:
>    - Previous work experience: 15 years
>    - Education: Bachelor's degree (4 years bonus)
>    - Disability: Moderate
> 
> 2. System calculates:
>    - Total tenure: 19 years (15 + 4)
>    - Base leave: 26 days (tenure ≥ 10 years)
>    - Disability bonus: +10 days
>    - Total annual leave: 36 days
>    - Daily work limit: 7 hours (disability accommodation)
> 
> 3. System restrictions automatically applied:
>    - Cannot be assigned overtime
>    - Cannot work night shifts
>    - Maximum 35 hours/week

### Story 3: Katarzyna's Part-Time Arrangement

> **Katarzyna**, mother of two young children, negotiates a 0.6 FTE position (24 hours/week) to balance work and family.
> 
> **What happens in Time2Work:**
> 1. HR configures:
>    - FTE: 0.6
>    - Work pattern: Mon-Wed-Fri, 8 hours each
>    - Has children under 4: Yes
> 
> 2. System calculates:
>    - Base leave entitlement: 26 days (12 years tenure)
>    - Proportional calculation: 26 × 0.6 = 15.6
>    - Rounded up: 16 days annual leave
>    - Remote work priority flag activated

## Daily Time Tracking Stories

### Story 4: Piotr's Unexpected Overtime

> **Piotr**, a customer service representative, usually works 8:00-16:00. On Tuesday, a major client issue keeps him until 19:30.
> 
> **What happens in Time2Work:**
> 1. Piotr clocks in: 08:02
> 2. Takes lunch break: 12:00-12:30 (unpaid)
> 3. Clocks out: 19:32
> 
> 4. System automatically calculates:
>    - Total time at work: 11 hours 30 minutes
>    - Minus break: 30 minutes
>    - Net work time: 11 hours
>    - Regular hours: 8
>    - Overtime: 3 hours
>      - First 2 hours: 150% rate (50% supplement)
>      - 3rd hour: 200% rate (100% supplement)
> 
> 5. Piotr's timesheet shows:
>    - Regular pay: 8 hours
>    - Overtime pay: 3 hours base + 2.5 hours supplement equivalent

### Story 5: Magda's Night Shift Pattern

> **Magda** works in a 24/7 monitoring center on the equivalent work system (system równoważny), with 12-hour shifts.
> 
> **What happens in Time2Work:**
> 1. Magda's schedule for the week:
>    - Monday: 19:00 - 07:00 (night shift)
>    - Tuesday: Off
>    - Wednesday: 19:00 - 07:00 (night shift)
>    - Thursday: Off
>    - Friday-Sunday: Off
> 
> 2. System validates and calculates:
>    - Each shift: 12 hours (allowed in equivalent system)
>    - Night hours: 10 hours per shift (21:00-07:00)
>    - Weekly total: 24 hours (under 40-hour average)
>    - Night work supplement: 20% of minimum wage per night hour
>    - Validates 11-hour rest between shifts: ✓ Compliant

### Story 6: Remote Work Friday Mix

> **Tomasz** has a hybrid arrangement - office Monday-Thursday, remote on Fridays.
> 
> **What happens in Time2Work:**
> 1. Friday morning, Tomasz:
>    - Opens mobile app at home
>    - Starts "remote work" mode at 07:30
>    - Takes a break at 10:00-10:30 (personal time)
>    - Lunch: 13:00-13:30
>    - Ends work: 16:00
> 
> 2. System tracks:
>    - Remote work flag active
>    - Total time: 8.5 hours
>    - Minus breaks: 1 hour
>    - Net work: 7.5 hours
>    - Note: Started earlier than usual (flex time allowed)

## Leave Management Stories

### Story 7: Ewa's Wedding Planning

> **Ewa** is getting married in June and wants to take 2 weeks off plus her special occasion leave.
> 
> **What happens in Time2Work:**
> 1. Ewa requests:
>    - June 15-26: 10 days annual leave
>    - June 27-28: 2 days wedding leave (special occasion)
> 
> 2. System checks:
>    - Current balance: 18 days
>    - Annual leave requested: 10 days ✓
>    - Wedding leave entitlement: 2 days ✓
>    - Total: 12 working days off
> 
> 3. Manager receives notification:
>    - Clear breakdown of leave types
>    - One-click approval
>    - System auto-updates team calendar
> 
> 4. After approval:
>    - Annual leave balance: 8 days remaining
>    - Wedding leave: Used for the year

### Story 8: Emergency Family Situation

> **Robert** receives a call at 10 AM that his child is sick at school. He needs to leave immediately.
> 
> **What happens in Time2Work:**
> 1. Robert uses mobile app:
>    - Selects "Leave on demand" (urlop na żądanie)
>    - Reason: "Family emergency"
>    - Submit at 10:15 AM
> 
> 2. System processes:
>    - Validates: Used 1 of 4 days this year ✓
>    - Deducts from annual leave balance
>    - Sends notification to manager
>    - No approval needed (automatic for on-demand)
> 
> 3. Robert's records show:
>    - March 15: Half day worked (8:00-10:15)
>    - Leave on demand: 0.5 day
>    - Annual balance reduced by 0.5 day

### Story 9: Maternity to Parental Leave Transition

> **Aleksandra** is expecting her first child in April 2025.
> 
> **Timeline in Time2Work:**
> 
> **February 2025** - Pregnancy registration:
> - Updates profile: Pregnant = Yes
> - System blocks: No overtime, no night shifts
> - Remote work priority activated
> 
> **April 15** - Baby arrives:
> 1. HR initiates maternity leave:
>    - Start date: April 15
>    - Duration: 20 weeks
>    - Pay: 100%
>    - End date: September 1
> 
> **August 1** - Planning ahead:
> 1. Aleksandra decides to take parental leave
> 2. Submits request while on maternity leave:
>    - Parental leave: 32 weeks
>    - Reduced pay option: 70%
>    - Partner will take remaining 9 weeks
> 
> **September 2** - Automatic transition:
> - Maternity leave ends
> - Parental leave begins seamlessly
> - No gap in coverage or pay

## Special Protections Stories

### Story 10: Protecting Pregnant Małgorzata

> **Małgorzata**, 3 months pregnant, works as a retail supervisor. Her manager tries to schedule her for inventory night shift.
> 
> **What happens in Time2Work:**
> 1. Manager attempts to assign:
>    - Date: March 20, 22:00-06:00
>    - Type: Inventory count (overtime)
> 
> 2. System blocks with error:
>    ```
>    ❌ Cannot assign shift
>    - Employee has protected status: Pregnant
>    - Night work prohibited (Art. 178 § 1 KP)
>    - Overtime prohibited (Art. 178 § 1 KP)
>    ```
> 
> 3. Alternative suggested:
>    - System shows available employees without restrictions
>    - Suggests day shift for Małgorzata instead

### Story 11: Minor Employee Summer Job

> **Kacper**, 17 years old, gets a summer job at a café.
> 
> **What happens in Time2Work:**
> 1. Profile created with birth date verification
> 2. System auto-configures:
>    - Maximum 6 hours per day
>    - No work between 22:00-06:00
>    - No overtime allowed
>    - Mandatory 30-minute break after 4.5 hours
> 
> 3. When scheduling, manager sees:
>    - Available hours: 06:00-22:00 only
>    - Daily limit indicator: Max 6h
>    - Auto-break insertion after 4.5 hours

## Work System Transitions

### Story 12: Seasonal Business Flexibility

> **Paweł** manages a tax consulting firm that needs different work arrangements during tax season.
> 
> **January-March (Tax Season):**
> 1. Switches team to flexible system (ruchomy):
>    - Core hours: 10:00-14:00
>    - Flexible start: 06:00-10:00
>    - Flexible end: 14:00-20:00
>    - Some Saturdays required
> 
> 2. System validates each schedule:
>    - Daily max: Still 8 hours average
>    - Weekly max: 48 hours (with overtime)
>    - Mandatory rest periods maintained
> 
> **April-December (Normal Period):**
> 1. Reverts to basic system:
>    - Standard hours: 8:00-16:00
>    - Regular 5-day week
>    - Overtime only when necessary

### Story 13: Manufacturing Shift Patterns

> **Janusz** works in automotive manufacturing with rotating shifts.
> 
> **Work Pattern (Continuous System):**
> - Week 1: Morning shift (06:00-14:00)
> - Week 2: Afternoon shift (14:00-22:00)  
> - Week 3: Night shift (22:00-06:00)
> - Week 4: Off (rest week)
> 
> **System Handles:**
> 1. Automatic shift supplements:
>    - Night shift: +20% for hours 22:00-06:00
>    - Sunday work: +100% supplement
>    - Holiday work: +100% supplement + day off in lieu
> 
> 2. Complex rest validation:
>    - Between shift types: Minimum 24 hours
>    - After night shift week: 48 hours before next shift
>    - Ensures one Sunday off per month

## Compliance and Reporting Stories

### Story 14: Surprise PIP Inspection

> **Tuesday, 10:00 AM** - Labor inspector arrives at the company for a routine inspection.
> 
> **HR Manager's Response:**
> 1. Opens Time2Work compliance module
> 2. Selects "PIP Inspection Report"
> 3. Parameters:
>    - Employee: Specific individual
>    - Period: Last 3 months
>    - Click "Generate"
> 
> **10:04 AM - Report Ready:**
> - Complete time records with exact clock in/out
> - All overtime calculated and justified
> - Rest period compliance chart
> - Leave balances and usage
> - Special protection adherence proof
> - Digital signature and timestamp
> 
> **Inspector's Review:**
> - Validates hash integrity
> - Confirms all legal requirements met
> - Inspection passed

### Story 15: Year-End Leave Reconciliation

> **December 15, 2025** - System runs automatic year-end analysis.
> 
> **Notifications sent:**
> 1. **To Employees with unused leave:**
>    > "You have 8 days of leave remaining. Remember to plan your time off by September 30, 2026, or it will expire."
> 
> 2. **To Managers:**
>    > "Team leave summary: 
>    > - 5 employees with >10 days remaining
>    > - Encourage leave planning for Q1 2026
>    > - Avoid year-end bottlenecks"
> 
> 3. **To HR:**
>    > "Annual leave liability report:
>    > - Total days unused: 287
>    > - Financial liability: 426,000 PLN
>    > - Expiry risk assessment included"

## Edge Cases and Complex Scenarios

### Story 16: Multi-Status Employee

> **Barbara** is a unique case:
> - Part-time (0.75 FTE)
> - Moderate disability
> - Mother of 3-year-old
> - 11 years total tenure
> 
> **System Calculations:**
> 1. Base leave: 26 days (tenure ≥ 10)
> 2. Proportional: 26 × 0.75 = 19.5 → 20 days
> 3. Disability bonus: +10 days
> 4. Total: 30 days annual leave
> 5. Daily work limit: 5.25 hours (7h × 0.75)
> 6. Remote work priority
> 7. Flexible hours preference

### Story 17: Contract Type Transition

> **Michał** transitions from service agreement to employment contract mid-year.
> 
> **January-June (Service Agreement):**
> - Time tracking: Allowed
> - Overtime: Calculated and paid
> - Leave: None
> - Benefits: Limited
> 
> **July 1 - Contract Change:**
> 1. System prompts complete re-configuration
> 2. Leave calculation:
>    - Annual entitlement: 20 days
>    - Proportional for 6 months: 10 days
>    - Available immediately
> 3. Historical data:
>    - Time records retained
>    - Overtime history preserved
>    - New compliance rules apply going forward

### Story 18: Retail Sunday Complexity

> **Agnieszka** manages a shopping mall retail store navigating Sunday trading restrictions.
> 
> **2025 Sunday Work Planning:**
> 1. System shows allowed trading Sundays:
>    - January 26
>    - April 13 (before Easter)
>    - April 27
>    - June 29
>    - August 31
>    - December 14 & 21
> 
> 2. For each Sunday:
>    - Staff scheduling allowed
>    - 100% Sunday supplement applies
>    - Voluntary basis only
>    - Ensures each employee gets 1 Sunday off per 4 weeks
> 
> 3. Non-trading Sundays:
>    - System blocks shift creation
>    - Exception only for essential services
>    - Automatic compliance with law

### Story 19: Cross-Border Remote Work

> **Kamil**, a software developer, wants to work remotely from Spain for 3 weeks.
> 
> **System Considerations:**
> 1. Remote work request:
>    - Location: Spain
>    - Duration: 15 working days
>    - Time zone: CET (same as Poland)
> 
> 2. Validation checks:
>    - Company policy: Max 30 days/year abroad ✓
>    - Tax implications: Under 183-day limit ✓
>    - Insurance: Valid in EU ✓
> 
> 3. Time tracking adjustments:
>    - Time zone awareness
>    - Public holidays: Polish calendar applies
>    - Compliance: Polish labor law continues

### Story 20: Historical Correction Discovery

> **March 2026** - During internal audit, HR discovers Tomasz was underpaid overtime in October 2025.
> 
> **Correction Process:**
> 1. HR accesses October 2025 records
> 2. Identifies error:
>    - Worked: 10 hours overtime
>    - Paid: 150% rate for all
>    - Should be: 150% for first 2h, 200% for remaining 8h
> 
> 3. System correction:
>    - Creates correction event
>    - Calculates difference: 8 hours × 50% = 4 hours
>    - Audit log: Who, when, why
>    - Links to original records
> 
> 4. Compliance maintained:
>    - Original records unchanged (immutable)
>    - Correction clearly documented
>    - Payment adjustment processed

## Scenario Categories Summary

### Time-Sensitive Scenarios
- Leave on demand: Same-day processing
- PIP inspection: 5-minute response time
- Rest period validation: Real-time blocking
- Overtime calculations: Immediate on clock-out

### Complex Calculations
- First-year employee: Monthly accrual
- Part-time proportional: Always round up
- Multi-status employees: Layered protections
- Contract transitions: Mid-year adjustments

### Protective Mechanisms
- Pregnancy: Automatic work restrictions
- Disability: Reduced hours enforcement
- Minors: Age-based limitations
- Rest periods: Schedule validation

### Compliance Features
- Audit trail: Every change tracked
- Historical corrections: Immutable records
- Report generation: Instant and accurate
- Legal updates: System adaptability

## Implementation Priority

Based on these scenarios, the development team should prioritize:

1. **Core time tracking with overtime calculation**
2. **Employee protections and validations**
3. **Leave management and accrual**
4. **Work system flexibility**
5. **Compliance reporting**
6. **Edge cases and corrections**

Each scenario demonstrates how Time2Work transforms complex Polish labor law requirements into seamless user experiences, protecting both employees and employers while ensuring complete compliance.