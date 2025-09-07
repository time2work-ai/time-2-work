---
title: "Work Time Systems Implementation"
description: "Technical implementation for Polish work time organization systems"
version: "1.0"
date: "2025-09-02"
type: "Technical Reference"
audience: "Development Team"
priority: "HIGH"
document_number: "03"
input: "02_user_scenarios.md"
output: "Technical specifications for implementation"
---

# Work Time Systems Implementation

## System Definitions

### 1. System Podstawowy (Basic System) - Art. 128-129 KP

```python
def podstawowy_system_validation():
    """
    Basic work time system implementation
    """
    return {
        'daily_limits': {
            'standard_work': 8,  # hours
            'maximum_with_overtime': 12,  # 8 standard + 4 overtime max
            'absolute_daily_maximum': 16  # Emergency/exceptional cases only
        },
        'weekly_limits': {
            'standard_average': 40,  # hours over settlement period
            'maximum_with_overtime': 48,  # Art. 131 KP
            'settlement_period_max': 4  # months
        },
        'rest_requirements': {
            'daily_rest': 11,  # continuous hours
            'weekly_rest': 35,  # continuous hours
            'between_shifts': 11  # minimum hours
        },
        'overtime_rules': {
            'annual_limit': 150,  # hours (can be increased by collective agreement)
            'daily_calculation': 'hours over 8',
            'weekly_calculation': 'average hours over 40 in settlement period'
        },
        'validation_functions': [
            'validate_daily_rest_periods',
            'validate_weekly_rest_periods', 
            'validate_annual_overtime_limits',
            'validate_settlement_period_averaging'
        ]
    }
```

### 2. System Równoważny (Equivalent System) - Art. 135-137 KP

```python
def równoważny_system_implementation():
    """
    Industry-specific daily hour limits
    """
    
    industry_specific_rules = {
        'standard_industry': {
            'daily_maximum': 12,  # hours
            'weekly_average': 40,  # over settlement period
            'settlement_period': 1,  # month (standard)
            'justification_required': False
        },
        
        'monitoring_equipment': {
            'daily_maximum': 16,  # hours
            'weekly_average': 40,
            'settlement_period': 3,  # months
            'justification_required': True,
            'special_conditions': 'Remote monitoring, automated systems'
        },
        
        'security_guards': {
            'daily_maximum': 24,  # hours
            'weekly_average': 40,
            'settlement_period': 1,  # month
            'continuous_presence': True,
            'special_rest_after_24h': 24  # hours mandatory rest
        },
        
        'healthcare_emergency': {
            'daily_maximum': 12,
            'weekly_average': 40,
            'settlement_period': 3,  # months
            'emergency_extension': 'up to 16h in critical situations'
        },
        
        'seasonal_work': {
            'daily_maximum': 12,
            'weekly_average': 40,
            'settlement_period': 4,  # months (seasonal variation)
            'seasonal_justification': True
        }
    }
    
    def validate_równoważny_schedule(employee, proposed_schedule):
        """Validate równoważny schedule compliance"""
        
        industry_rules = industry_specific_rules[employee.industry_type]
        violations = []
        
        # Daily limit validation
        for day in proposed_schedule:
            if day.total_hours > industry_rules['daily_maximum']:
                violations.append(f"Daily limit exceeded: {day.total_hours}h > {industry_rules['daily_maximum']}h")
        
        # Settlement period averaging
        settlement_average = calculate_settlement_period_average(proposed_schedule, industry_rules['settlement_period'])
        if settlement_average > industry_rules['weekly_average']:
            violations.append(f"Settlement period average exceeded: {settlement_average}h > {industry_rules['weekly_average']}h")
        
        # Balancing requirement - longer days must be balanced by shorter/free days
        if not validate_balancing_requirement(proposed_schedule):
            violations.append("Równoważny system requires balancing longer days with shorter/free days")
        
        return violations
    
    return {
        'industry_rules': industry_specific_rules,
        'validation_engine': validate_równoważny_schedule,
        'legal_basis': 'Art. 135-137 KP + Industry regulations'
    }
```

### 3. Weekend Work System - Art. 151¹-151¹¹ KP

```python
def weekend_work_system():
    """
    Weekend work validation including retail restrictions
    """
    
    def get_retail_sunday_calendar_2025():
        """Only 7 Sundays allowed for retail trade in 2025"""
        return [
            date(2025, 1, 26),   # Last Sunday of January
            date(2025, 4, 13),   # Sunday before Easter
            date(2025, 4, 27),   # Last Sunday of April  
            date(2025, 6, 29),   # Last Sunday of June
            date(2025, 8, 31),   # Last Sunday of August
            date(2025, 12, 14),  # First Sunday before Christmas
            date(2025, 12, 21)   # Second Sunday before Christmas
        ]
    
    def validate_weekend_work(work_date, employee, business_type):
        """Comprehensive weekend work validation"""
        
        violations = []
        
        # Sunday work validation
        if work_date.weekday() == 6:  # Sunday
            
            # Retail-specific restrictions
            if business_type == 'retail':
                approved_sundays = get_retail_sunday_calendar_2025()
                if work_date not in approved_sundays:
                    violations.append({
                        'type': 'illegal_sunday_retail_work',
                        'severity': 'CRITICAL',
                        'fine_range': (5000, 50000),  # PLN
                        'message': f'Retail work prohibited on {work_date}. Only 7 Sundays allowed in 2025.'
                    })
            
            # General Sunday work rules
            sundays_worked_last_4_weeks = count_sundays_worked_last_4_weeks(employee.id)
            if sundays_worked_last_4_weeks >= 3:
                violations.append({
                    'type': 'excessive_sunday_work',
                    'severity': 'HIGH',
                    'message': 'Employee must have at least 1 Sunday off every 4 weeks'
                })
        
        return violations
    
    def calculate_weekend_supplements(base_rate, hours, work_date, employee_schedule):
        """Weekend work supplement calculations"""
        
        supplements = {}
        
        if work_date.weekday() == 6:  # Sunday
            if not is_normal_work_day(work_date, employee_schedule):
                supplements['sunday_work'] = {
                    'hours': hours,
                    'supplement_rate': 1.0,  # 100% supplement
                    'supplement_amount': base_rate * hours * 1.0,
                    'alternative': 'day_off_in_lieu_within_6_days'
                }
        
        if is_public_holiday(work_date):
            supplements['holiday_work'] = {
                'hours': hours,
                'supplement_rate': 1.0,  # 100% supplement  
                'supplement_amount': base_rate * hours * 1.0,
                'alternative': 'day_off_in_lieu'
            }
        
        return supplements
    
    return {
        'retail_calendar': get_retail_sunday_calendar_2025(),
        'validation_engine': validate_weekend_work,
        'supplement_calculator': calculate_weekend_supplements,
        'compliance_monitoring': 'Real-time Sunday work limit tracking'
    }
```

### 4. Task-Based System (Zadaniowy) - Art. 140 KP

```python
def zadaniowy_system_implementation():
    """
    Task-based system implementation
    """
    
    return {
        'tracking_requirements': {
            'time_tracking': False,     # No start/end times
            'task_completion': True,    # Track deliverables
            'presence_days': True,      # Work days (not hours)
            'leave_tracking': True,     # Full leave management still required
            'absence_tracking': True    # With reasons
        },
        
        'task_equivalency_validation': {
            'daily_task_load': 'Must correspond to 8h standard work',
            'weekly_task_load': 'Must correspond to 40h average',
            'measurement_method': 'Objective task completion metrics',
            'performance_standards': 'Must be achievable in standard work time'
        },
        
        'overtime_rules': {
            'overtime_applicable': False,  # No overtime concept in task-based
            'additional_tasks': 'Separate agreement required',
            'performance_bonuses': 'Allowed as completion incentives'
        },
        
        'compliance_considerations': {
            'rest_periods': 'Still required between work days',
            'annual_leave': 'Full entitlement applies',
            'special_protections': 'Pregnancy/disability rules still apply',
            'health_safety': 'Employer responsibility unchanged'
        },
        
        'suitable_positions': [
            'Senior developers/architects',
            'Sales representatives', 
            'Field consultants',
            'Research personnel',
            'Creative professionals'
        ],
        
        'legal_basis': 'Art. 140 KP'
    }
```

### 5. Flexible Work Time (Ruchomy) - Art. 140¹ KP

```python
def ruchomy_system_implementation():
    """
    Flexible work time system implementation
    """
    
    flexibility_variants = {
        'variant_1': {
            'name': 'Different start times each day',
            'implementation': 'Employer sets different start times for each workday',
            'employee_choice': False,
            'core_hours': 'Optional - can be defined by employer'
        },
        
        'variant_2': {
            'name': 'Employee choice within time frame',
            'implementation': 'Employee chooses start time within employer-defined window',
            'employee_choice': True,
            'time_window': 'Defined by employer (e.g., 7:00-10:00 start window)',
            'core_hours': 'Often required (e.g., 10:00-15:00 mandatory presence)'
        }
    }
    
    def validate_ruchomy_schedule(employee, schedule_config):
        """Validate flexible work time schedule"""
        
        violations = []
        
        # Daily work time must still be 8 hours
        for day in schedule_config['work_days']:
            if day.total_work_hours != 8:
                violations.append(f"Daily work time must be 8 hours, got {day.total_work_hours}")
        
        # Core hours compliance (if defined)
        if schedule_config.get('core_hours'):
            core_start = schedule_config['core_hours']['start']
            core_end = schedule_config['core_hours']['end']
            
            for day in schedule_config['work_days']:
                if not (day.start_time <= core_start and day.end_time >= core_end):
                    violations.append(f"Core hours violation on {day.date}")
        
        # Weekly average still 40 hours
        weekly_average = sum(day.total_work_hours for day in schedule_config['work_days']) / len(schedule_config['work_days']) * 5
        if weekly_average != 40:
            violations.append(f"Weekly average must be 40h, got {weekly_average}h")
        
        return violations
    
    return {
        'variants': flexibility_variants,
        'validation_engine': validate_ruchomy_schedule,
        'overtime_calculation': 'Standard rules apply for hours over 8/day or 40/week average',
        'legal_basis': 'Art. 140¹ KP'
    }
```

### 6. Interrupted Work Time (Przerywany) - Art. 139 KP

```python
def przerywany_system_implementation():
    """
    Interrupted work time system implementation
    """
    
    return {
        'interruption_rules': {
            'max_interruptions_per_day': 1,
            'max_interruption_duration': 5,  # hours
            'paid_interruption': False,  # Unless specifically agreed
            'total_presence_time': 'work_time + interruption_time'
        },
        
        'typical_use_cases': [
            'School employees (teaching + administrative break)',
            'Healthcare (patient care + administrative break)', 
            'Security (active monitoring + standby break)',
            'Transportation (driving + mandatory rest break)'
        ],
        
        'scheduling_validation': {
            'interruption_timing': 'Must be predictable and planned',
            'work_continuity': 'Work after interruption must be related to work before',
            'location_requirements': 'Employee availability during interruption',
            'compensation': 'Only work time compensated unless agreed otherwise'
        },
        
        'compliance_considerations': {
            'rest_periods': 'Daily 11h rest still required after total presence time',
            'overtime_calculation': 'Based on actual work hours (excluding interruption)',
            'special_protections': 'Pregnancy/disability rules apply to total presence time'
        },
        
        'implementation_example': {
            'teacher_schedule': {
                'morning_classes': '08:00-12:00',  # 4h work
                'lunch_break': '12:00-14:00',     # 2h unpaid interruption
                'afternoon_classes': '14:00-18:00', # 4h work
                'total_work': 8,  # hours
                'total_presence': 10,  # hours
                'compensation': 'Only 8 hours paid'
            }
        },
        
        'legal_basis': 'Art. 139 KP'
    }
```

### 7. Continuous Operation System (Ruch Ciągły) - Art. 138 KP

```python
def ruch_ciagly_system_implementation():
    """
    Continuous operation system implementation
    """
    
    return {
        'operation_requirements': {
            'business_justification': 'Production/service cannot be interrupted',
            'typical_industries': ['Manufacturing', 'Healthcare', 'Energy', 'Transportation'],
            'shift_pattern': 'Usually 3-shift or 4-shift rotation'
        },
        
        'time_limits': {
            'daily_maximum': 12,  # hours per shift
            'weekly_average': 43,  # hours (higher than standard 40h)
            'settlement_period': 4,  # weeks maximum
            'annual_working_time': 'Cannot exceed yearly norm'
        },
        
        'shift_management': {
            'shift_rotation': 'Must be planned in advance',
            'shift_change_timing': 'Smooth transition required',
            'coverage_requirements': 'No operational gaps allowed',
            'emergency_coverage': 'Procedures for unexpected absences'
        },
        
        'enhanced_rest_requirements': {
            'between_shifts': 11,  # hours minimum
            'after_night_shift': 14,  # hours recommended
            'weekly_rest': 35,  # continuous hours
            'shift_pattern_health': 'Regular health monitoring required'
        },
        
        'compliance_monitoring': {
            'weekly_hour_averaging': 'Must not exceed 43h average over 4 weeks',
            'annual_limit_tracking': 'Total annual hours within legal limits',
            'health_impact_assessment': 'Regular evaluation of shift work effects',
            'emergency_work_documentation': 'Justification for extended hours'
        },
        
        'legal_basis': 'Art. 138 KP + Industry-specific regulations'
    }
```

### 8. Shortened Work Week System - Art. 143 KP

```python
def skrócony_tydzień_implementation():
    """
    Shortened work week implementation
    """
    
    def calculate_compressed_schedule(total_weekly_hours: int, work_days: int):
        """Calculate daily hours for compressed schedule"""
        daily_hours = total_weekly_hours / work_days
        
        if daily_hours > 12:
            raise ComplianceError(f"Daily hours {daily_hours} exceed 12h maximum for shortened week")
        
        return {
            'daily_hours': daily_hours,
            'work_days': work_days,
            'weekly_total': total_weekly_hours,
            'rest_days': 7 - work_days
        }
    
    common_patterns = {
        '4x10_schedule': {
            'work_days': 4,
            'daily_hours': 10,
            'weekly_total': 40,
            'typical_pattern': 'Monday-Thursday',
            'benefits': ['3-day weekend', 'Reduced commuting', 'Better work-life balance'],
            'challenges': ['Longer daily fatigue', 'Client coordination', 'Meeting scheduling']
        },
        
        '3x12_schedule': {
            'work_days': 3,
            'daily_hours': 12,  # Maximum allowed
            'weekly_total': 36,  # Below 40h average
            'compensation_required': 'Additional day off every 4 weeks to reach 40h average',
            'typical_industries': ['Manufacturing', 'Healthcare', 'Security']
        },
        
        'flexible_4_day': {
            'work_days': 4,
            'daily_hours': 10,
            'flexible_day': 'Different day off each week (Monday/Friday rotation)',
            'business_continuity': 'Staggered team schedules for coverage'
        }
    }
    
    def validate_shortened_week(schedule_pattern, industry_type):
        """Validation for shortened week compliance"""
        
        violations = []
        
        # Daily hour limits
        if schedule_pattern['daily_hours'] > 12:
            violations.append("Daily hours cannot exceed 12 in shortened week system")
        
        # Weekly average requirement
        effective_weekly_average = calculate_effective_weekly_average(schedule_pattern)
        if effective_weekly_average != 40:
            violations.append(f"Effective weekly average must be 40h, got {effective_weekly_average}h")
        
        # Industry suitability
        if not is_suitable_for_industry(schedule_pattern, industry_type):
            violations.append(f"Shortened week pattern not suitable for {industry_type}")
        
        return violations
    
    return {
        'common_patterns': common_patterns,
        'calculation_engine': calculate_compressed_schedule,
        'validation_engine': validate_shortened_week,
        'implementation_guidance': 'Requires careful planning and employee consultation'
    }
```

### 9. Work System Selection Engine

```python
def work_system_selection_engine():
    """
    Intelligent system for recommending appropriate work time systems
    Missing entirely from your current specs
    """
    
    def recommend_work_system(business_type: str, position_type: str, operational_needs: dict) -> dict:
        """Recommend optimal work system based on business requirements"""
        
        recommendations = []
        
        # Basic system (default)
        if operational_needs.get('standard_hours') and not operational_needs.get('flexibility_required'):
            recommendations.append({
                'system': 'podstawowy',
                'suitability': 90,
                'reasoning': 'Standard office work with regular hours',
                'compliance_complexity': 'LOW'
            })
        
        # Task-based for knowledge workers
        if position_type in ['developer', 'consultant', 'researcher', 'sales']:
            recommendations.append({
                'system': 'zadaniowy', 
                'suitability': 85,
                'reasoning': 'Results-focused work suitable for task-based system',
                'compliance_complexity': 'MEDIUM',
                'benefits': ['Simplified tracking', 'Employee autonomy', 'Performance focus']
            })
        
        # Equivalent system for operational flexibility
        if operational_needs.get('seasonal_variation') or operational_needs.get('customer_demand_variation'):
            recommendations.append({
                'system': 'równoważny',
                'suitability': 75,
                'reasoning': 'Business needs require flexible daily hours',
                'compliance_complexity': 'HIGH',
                'warning': 'Requires careful settlement period management'
            })
        
        # Shortened week for work-life balance
        if operational_needs.get('employee_retention_focus') and business_type in ['IT', 'creative', 'consulting']:
            recommendations.append({
                'system': 'skrócony_tydzień',
                'suitability': 70,
                'reasoning': 'Competitive benefit for talent retention',
                'compliance_complexity': 'MEDIUM',
                'implementation_note': 'Requires 3-6 month pilot period'
            })
        
        # Sort by suitability
        recommendations.sort(key=lambda x: x['suitability'], reverse=True)
        
        return {
            'primary_recommendation': recommendations[0],
            'alternative_options': recommendations[1:],
            'legal_consultation_required': any(r['compliance_complexity'] == 'HIGH' for r in recommendations)
        }
    
    return {
        'recommendation_engine': recommend_work_system,
        'implementation_guidance': get_implementation_guidance_by_system(),
        'legal_review_requirements': get_legal_review_requirements()
    }
```

### 10. System Transition and Migration

```python
def work_system_transition_management():
    """
    Managing transitions between work systems
    Missing entirely from your current specs but legally critical
    """
    
    def plan_system_transition(employee_id: int, current_system: str, target_system: str):
        """Plan legal transition between work systems"""
        
        transition_requirements = {
            'legal_consultation': True,  # Required for most transitions
            'employee_consent': True,    # Required for employee protection
            'notice_period': calculate_notice_period(current_system, target_system),
            'contract_amendment': True,  # Formal contract modification required
            'transition_date': 'Must align with settlement period end'
        }
        
        # Special transition rules
        if current_system == 'podstawowy' and target_system == 'zadaniowy':
            transition_requirements.update({
                'performance_baseline': 'Establish current productivity metrics',
                'task_definition': 'Clear definition of equivalent task load',
                'trial_period': '3-6 months recommended',
                'rollback_plan': 'Procedure for reverting if unsuccessful'
            })
        
        if current_system in ['równoważny', 'ruch_ciągły'] and target_system == 'podstawowy':
            transition_requirements.update({
                'settlement_completion': 'Complete current settlement period',
                'overtime_payout': 'Settle any accumulated overtime',
                'schedule_adjustment': 'Gradual transition to standard hours'
            })
        
        return transition_requirements
    
    def validate_transition_legality(current_system: str, target_system: str, employee_data: dict):
        """Ensure transition doesn't violate employee rights"""
        
        violations = []
        
        # Cannot transition to worse conditions without consent
        if is_less_favorable_system(target_system, current_system):
            if not employee_data.get('explicit_consent'):
                violations.append("Transition to less favorable system requires explicit employee consent")
        
        # Special status restrictions
        if employee_data.get('pregnant') and target_system in ['równoważny', 'ruch_ciągły']:
            violations.append("Cannot transition pregnant employee to system allowing extended hours")
        
        if employee_data.get('disability') and target_system == 'ruch_ciągły':
            violations.append("Cannot transition disabled employee to continuous operation system")
        
        return violations
    
    return {
        'transition_planner': plan_system_transition,
        'legal_validator': validate_transition_legality,
        'documentation_requirements': 'All transitions must be documented with legal basis'
    }
```

## Integration with Current BDD Scenarios

### Corrected BDD Scenarios

```gherkin
Feature: Accurate Work Time System Implementation

  Scenario: Równoważny System - Security Guard (24h shifts)
    Given I am configuring a security guard position
    When I assign them to "równoważny" system for "security industry"
    Then the system should allow daily shifts up to "24 hours"
    And require "24 hours" of rest after a 24-hour shift
    And validate weekly average does not exceed "40 hours"

  Scenario: Retail Sunday Work Validation
    Given I am scheduling retail work on "Sunday January 14, 2025"
    When I attempt to create the schedule
    Then the system must block the action
    And display error: "Retail work prohibited - not an approved Sunday in 2025"
    And show next available approved Sunday: "January 26, 2025"

  Scenario: Task-Based System Configuration
    Given I am configuring a senior developer position
    When I assign them to "zadaniowy" system
    Then the system should disable start/end time tracking
    But maintain task completion tracking
    And keep full leave management capabilities
    And disable overtime calculations

  Scenario: Work System Transition
    Given an employee currently on "podstawowy" system
    When I attempt to transition them to "równoważny" system
    Then the system must require explicit employee consent
    And schedule transition for end of current settlement period
    And require contract amendment documentation
```

## Critical Implementation Checklist

### Legal Validation Requirements
- [ ] **Polish labor law expert** review of all system implementations
- [ ] **Industry-specific rule validation** with sector experts
- [ ] **PIP consultation** on system-specific inspection requirements
- [ ] **Court precedent research** for system transition rules

### Technical Implementation
- [ ] **Industry-specific validation engines** for each system
- [ ] **Automatic schedule blocking** for illegal configurations  
- [ ] **Real-time compliance monitoring** for all systems
- [ ] **System transition workflows** with legal safeguards

### Testing Requirements
- [ ] **Edge case testing** for each system's maximum limits
- [ ] **Cross-system integration** testing
- [ ] **Compliance violation simulation** with automatic blocking
- [ ] **Performance testing** with complex multi-system organizations

**Complete implementation of all work time systems required for legal compliance.**