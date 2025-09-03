---
title: "Polish Labor Law Technical Reference"
description: "Agent-friendly technical reference for implementing Polish labor law compliance in Time2Work"
version: "1.0"
date: "2025-09-02"
type: "Technical Reference"
audience: "AI Agents, Development Team"
priority: "CRITICAL"
---

# Polish Labor Law Technical Reference

## Core Implementation Requirements

### 1. Overtime Calculation Engine

```python
def calculate_overtime_pay(base_hourly_rate: float, overtime_hours: float, work_context: str) -> dict:
    """
    Base pay + supplement structure per Art. 151⁴ KP
    """
    base_pay = base_hourly_rate * overtime_hours
    
    supplement_rates = {
        'regular': 0.5,      # 50% supplement for regular overtime
        'sunday': 1.0,       # 100% supplement for Sunday work
        'holiday': 1.0,      # 100% supplement for holiday work
        'rest_day': 1.0      # 100% supplement for scheduled rest day
    }
    
    supplement_rate = supplement_rates.get(work_context, 0.5)
    supplement_amount = base_hourly_rate * overtime_hours * supplement_rate
    
    return {
        'base_pay': base_pay,
        'supplement_amount': supplement_amount,
        'total_pay': base_pay + supplement_amount,
        'legal_basis': f'Art. 151⁴ KP - {supplement_rate * 100}% supplement'
    }
```

### 2. Work Time System Validation

```python
WORK_SYSTEMS = {
    'podstawowy': {
        'daily_max': 8,
        'overtime_daily_max': 4,
        'weekly_average': 40,
        'annual_overtime_limit': 150
    },
    'równoważny': {
        'daily_max_by_industry': {
            'standard': 12,
            'monitoring_equipment': 16,
            'security_guards': 24
        },
        'weekly_average': 40,
        'settlement_period_months': {'standard': 1, 'special': 3}
    },
    'zadaniowy': {
        'time_tracking': False,
        'task_tracking': True,
        'overtime_applicable': False,
        'leave_tracking': True
    }
}

def validate_work_schedule(employee_id: int, schedule: dict) -> list:
    """Validate schedule against work system rules"""
    employee = get_employee(employee_id)
    system_rules = WORK_SYSTEMS[employee.work_system]
    violations = []
    
    for day in schedule['work_days']:
        if day.hours > system_rules['daily_max']:
            violations.append(f"Daily limit exceeded: {day.hours}h > {system_rules['daily_max']}h")
    
    return violations
```

### 3. Employee Protection Enforcement

```python
def apply_employee_protections(employee_id: int, proposed_action: dict) -> dict:
    """Real-time protection validation before any work assignment"""
    employee = get_employee_with_protections(employee_id)
    violations = []
    
    # Pregnancy protections
    if employee.is_pregnant:
        if proposed_action.get('overtime') or proposed_action.get('night_work'):
            violations.append("Overtime and night work prohibited for pregnant employees")
    
    # Disability accommodations
    if employee.disability_level:
        system_limits = {
            'moderate': {'daily_max': 7, 'weekly_max': 35},
            'severe': {'daily_max': 6, 'weekly_max': 30}
        }
        limits = system_limits.get(employee.disability_level)
        if limits and proposed_action.get('daily_hours', 0) > limits['daily_max']:
            violations.append(f"Daily hours exceed disability limit: {limits['daily_max']}h")
    
    # Minor worker protections
    if calculate_age(employee.birth_date) < 18:
        if proposed_action.get('daily_hours', 0) > 6:
            violations.append("Minor workers limited to 6 hours daily")
    
    return {
        'action_permitted': len(violations) == 0,
        'violations': violations,
        'required_accommodations': generate_accommodations(employee) if violations else None
    }
```

### 4. Contract Type Validation

```python
CONTRACT_RULES = {
    'umowa_o_prace': {
        'time_tracking': True,
        'overtime_applicable': True,
        'leave_entitlement': True,
        'protection_obligations': True
    },
    'umowa_zlecenie': {
        'time_tracking': True,
        'overtime_applicable': True,
        'leave_entitlement': False,
        'protection_obligations': True
    },
    'umowa_o_dzielo': {
        'time_tracking': False,  # PROHIBITED - result-based only
        'overtime_applicable': False,
        'leave_entitlement': False,
        'protection_obligations': False,
        'tracking_error': 'B2B contracts cannot have time tracking'
    }
}

def validate_contract_tracking(employee_id: int, tracking_type: str) -> bool:
    """Prevent illegal tracking for B2B contracts"""
    employee = get_employee(employee_id)
    contract_rules = CONTRACT_RULES[employee.contract_type]
    
    if tracking_type == 'time' and not contract_rules['time_tracking']:
        raise ComplianceError(f"Time tracking prohibited for {employee.contract_type}")
    
    return contract_rules.get(tracking_type, False)
```

### 5. Data Retention Implementation

```python
def setup_data_retention_system():
    """10-year retention system per Art. 149 KP"""
    return {
        'retention_period': {
            'calculation': 'termination_year + 10 years + 1 calendar year',
            'example': 'Terminated 2024-06-15 → Retention expires 2035-01-01'
        },
        'required_data': [
            'exact_start_end_times',
            'overtime_breakdown_detailed',
            'leave_calculations_with_basis',
            'on_call_hours_with_locations',  # Location required
            'special_protection_compliance',
            'audit_trail_immutable'
        ],
        'pip_export_capability': {
            'response_time': '5 minutes for any employee data',
            'format_options': ['PDF', 'Excel', 'CSV'],
            'data_integrity': 'Cryptographic verification included'
        }
    }
```

### 6. PIP Inspection Response System

```python
def pip_emergency_response(employee_id: int, date_range: str) -> dict:
    """5-minute response to PIP inspection requests"""
    
    response_data = {
        'employee_profile': get_employee_complete_profile(employee_id),
        'work_time_records': get_daily_records_with_exact_times(employee_id, date_range),
        'overtime_calculations': get_overtime_breakdown_with_legal_basis(employee_id, date_range),
        'leave_history': get_leave_history_with_entitlement_calculations(employee_id, date_range),
        'compliance_validation': validate_all_compliance_rules(employee_id, date_range),
        'audit_trail': get_complete_audit_trail(employee_id, date_range)
    }
    
    # Generate legally certified export
    return {
        'export_data': response_data,
        'generation_time': datetime.utcnow(),
        'legal_certification': 'Compliant with Art. 149 KP',
        'data_integrity_hash': calculate_integrity_hash(response_data),
        'digital_signature': generate_legal_signature(response_data)
    }
```

### 7. Weekend Work Validation

```python
RETAIL_SUNDAY_CALENDAR_2024 = [
    '2024-01-28', '2024-03-24', '2024-04-28', 
    '2024-06-30', '2024-08-25', '2024-11-24', '2024-12-22'
]

def validate_weekend_work(work_date: str, business_type: str, employee_id: int) -> dict:
    """Validate Sunday work against retail restrictions"""
    violations = []
    
    if work_date.weekday() == 6 and business_type == 'retail':  # Sunday
        if work_date not in RETAIL_SUNDAY_CALENDAR_2024:
            violations.append({
                'type': 'illegal_sunday_retail',
                'severity': 'CRITICAL',
                'fine_range': (5000, 50000),  # PLN
                'message': f'Retail work prohibited on {work_date}'
            })
    
    return {'violations': violations, 'action_permitted': len(violations) == 0}
```

### 8. Database Schema Requirements

```sql
-- Core time tracking table
CREATE TABLE ewidencja_czasu_pracy (
    id BIGSERIAL PRIMARY KEY,
    pracownik_id INTEGER NOT NULL,
    data_pracy DATE NOT NULL,
    
    -- Exact times (REQUIRED)
    czas_rozpoczecia TIMESTAMP,
    czas_zakonczenia TIMESTAMP,
    godziny_przepracowane DECIMAL(4,2),
    
    -- Overtime breakdown (DETAILED)
    nadgodziny_50_procent DECIMAL(4,2),
    nadgodziny_100_procent DECIMAL(4,2),
    
    -- On-call (LOCATION REQUIRED)
    dyzur_godziny DECIMAL(4,2),
    dyzur_lokalizacja VARCHAR(200),
    
    -- Audit trail (IMMUTABLE)
    utworzono_timestamp TIMESTAMP DEFAULT NOW(),
    utworzono_przez_user_id INTEGER NOT NULL,
    hash_integralnosci VARCHAR(64),
    
    CONSTRAINT required_location_for_oncall CHECK (
        (dyzur_godziny IS NULL) OR 
        (dyzur_godziny IS NOT NULL AND dyzur_lokalizacja IS NOT NULL)
    )
);
```

## Critical Implementation Rules

### 1. Calculation Rules
- **Overtime**: Base pay + supplement (NOT percentage of base)
- **Leave entitlement**: Consider education levels, work experience
- **On-call work**: Location tracking mandatory

### 2. Protection Rules
- **Pregnant employees**: Block overtime, night work, hazardous tasks
- **Disabled employees**: Enforce hour limits, provide accommodations
- **Minor workers**: Strict hour limits, no night/overtime work

### 3. Data Rules
- **Retention**: 10 years from end of termination calendar year
- **Audit trail**: Immutable, complete change tracking
- **PIP readiness**: 5-minute response capability required

### 4. Contract Rules
- **Umowa o pracę**: Full tracking and protections
- **Umowa zlecenie**: Time tracking, no leave entitlement
- **Umowa o dzieło**: NO time tracking (result-based only)

### 5. Work System Rules
- **Podstawowy**: 8h daily, 40h weekly average
- **Równoważny**: Industry-specific daily limits (12h-24h)
- **Zadaniowy**: Task tracking only, no time tracking

## Implementation Priorities

### Phase 1 (Critical - Legal Compliance)
1. Fix overtime calculation algorithm
2. Implement protection enforcement engine
3. Setup 10-year data retention system
4. Create PIP emergency response capability

### Phase 2 (High - Operational)
1. Implement all 8 work time systems correctly
2. Create contract type validation engine
3. Setup employee self-service portal
4. Implement real-time violation prevention

### Phase 3 (Medium - Enhancement)
1. PIP inspector portal
2. Compliance monitoring dashboard
3. Legal documentation automation
4. System transition management

## Legal Validation Checklist

- [ ] Polish labor law expert review of all algorithms
- [ ] PIP consultation on inspection requirements
- [ ] Mathematical verification of calculation accuracy
- [ ] Employee protection legal validation
- [ ] Data retention legal compliance verification

**This reference provides the essential technical specifications for implementing legally compliant Polish labor law features in Time2Work. All implementations must follow these exact specifications to avoid legal violations.**