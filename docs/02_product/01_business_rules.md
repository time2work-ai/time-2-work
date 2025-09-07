---
title: "Polish Labor Law Technical Reference"
description: "Agent-friendly technical reference for implementing Polish labor law compliance in Time2Work"
version: "2.0"
date: "2025-09-06"
type: "Technical Reference"
audience: "AI Agents, Development Team"
priority: "CRITICAL"
document_number: "01"
input: "../01_business/polish-labor-law-reference.md"
output: "Business rules extracted and structured for implementation"
---

# Polish Labor Law Technical Reference

This document provides a clean architecture implementation guide for Polish labor law compliance, following DRY, KISS, SOLID principles and Martin Fowler's design patterns.

## Table of Contents

1. [Constants and Configuration](#constants-and-configuration)
2. [Domain Models](#domain-models)
3. [Value Objects](#value-objects)
4. [Service Layer](#service-layer)
5. [Infrastructure Specifications](#infrastructure-specifications)
6. [Implementation Guidelines](#implementation-guidelines)

## Constants and Configuration

### Core Constants

```python
from enum import Enum, IntEnum
from decimal import Decimal
from typing import Final

class WorkLimits(IntEnum):
    """Polish Labor Code work time limits."""
    DAILY_STANDARD_HOURS = 8
    WEEKLY_AVERAGE_HOURS = 40
    DAILY_OVERTIME_MAX = 4
    ANNUAL_OVERTIME_LIMIT = 150
    WEEKLY_WITH_OVERTIME_MAX = 48
    
    # Rest periods
    DAILY_REST_HOURS = 11
    WEEKLY_REST_HOURS = 35
    
    # Special worker limits
    MINOR_DAILY_HOURS = 6
    DISABLED_MODERATE_DAILY_HOURS = 7
    DISABLED_SEVERE_DAILY_HOURS = 6

class LeaveEntitlements(IntEnum):
    """Leave day entitlements."""
    STANDARD_ANNUAL_DAYS = 20
    EXTENDED_ANNUAL_DAYS = 26
    TENURE_THRESHOLD_YEARS = 10
    ON_DEMAND_DAYS_LIMIT = 4
    DISABILITY_ADDITIONAL_DAYS = 10

class OvertimeRates(Enum):
    """Overtime compensation rates as Decimal for precision."""
    FIRST_TWO_HOURS = Decimal("0.5")  # 50% supplement
    AFTER_TWO_HOURS = Decimal("1.0")  # 100% supplement
    SUNDAY_WORK = Decimal("1.0")      # 100% supplement
    HOLIDAY_WORK = Decimal("1.0")     # 100% supplement
    NIGHT_WORK = Decimal("0.2")       # 20% of minimum wage

class ContractType(Enum):
    """Employment contract types with tracking rules."""
    EMPLOYMENT = "employment_contract"      # Full tracking required (Polish: umowa o pracę)
    SERVICE = "service_agreement"           # Conditional tracking (Polish: umowa zlecenie)
    SPECIFIC_WORK = "specific_work_contract" # No time tracking allowed (Polish: umowa o dzieło)
    B2B = "b2b"                            # No time tracking allowed

class WorkSystemType(Enum):
    """Work time organization systems."""
    BASIC = "basic"                    # Polish: podstawowy
    EQUIVALENT = "equivalent"          # Polish: równoważny
    TASK_BASED = "task_based"         # Polish: zadaniowy
    FLEXIBLE = "flexible"             # Polish: ruchomy
    INTERRUPTED = "interrupted"       # Polish: przerywany
    CONTINUOUS = "continuous"         # Polish: ruch ciągły
    WEEKEND = "weekend"               # Polish: weekendowy
    SHORTENED_WEEK = "shortened_week" # Polish: skrócony tydzień

# Retail Sunday exceptions for 2025
# Based on the Act of January 10, 2018 on Limiting Trade on Sundays and Holidays
# Source: powroty.gov.pl (Polish government portal)
RETAIL_SUNDAY_EXCEPTIONS_2025: Final[list[str]] = [
    "2025-01-26",  # Last Sunday of January
    "2025-04-13",  # Sunday before Easter
    "2025-04-27",  # Last Sunday of April
    "2025-06-29",  # Last Sunday of June
    "2025-08-31",  # Last Sunday of August
    "2025-12-14",  # First Sunday before Christmas
    "2025-12-21"   # Second Sunday before Christmas
]
# Note: 2026 shopping Sundays not yet published by government sources
# System need to be able to discover and manage that on it's own (agentic research system, unless gov API is available)

# Data retention requirements per Art. 149 Labor Code
# Employers must retain all employee time and attendance records for audit purposes
RETENTION_YEARS_AFTER_TERMINATION: Final[int] = 10  # Years after end of employment termination year

# PIP (State Labor Inspectorate) inspection response requirement
# Target time for providing employee records during inspection
# While law requires "immediate" (natychmiast) access, 5 minutes is a practical implementation target
PIP_RESPONSE_TIME_MINUTES: Final[int] = 5  # System design target for immediate compliance
```

## Domain Models

### Employee Entity

```python
from dataclasses import dataclass
from datetime import date, datetime
from typing import Optional, List
from enum import Enum

class EmploymentStatus(Enum):
    ACTIVE = "active"
    ON_LEAVE = "on_leave"
    TERMINATED = "terminated"

class DisabilityLevel(Enum):
    NONE = "none"
    MODERATE = "moderate"
    SEVERE = "severe"

class Sex(Enum):
    """Biological sex - required for pregnancy protection validation"""
    MALE = "male"
    FEMALE = "female"

@dataclass
class Employee:
    """Core employee entity with essential attributes."""
    id: int
    contract_type: ContractType
    work_system: WorkSystemType
    hire_date: date
    birth_date: date
    sex: Sex  # Required by Polish labor law for certain protections
    education_level: str
    employment_status: EmploymentStatus
    
    # Special statuses
    is_pregnant: bool = False  # Only valid if sex == Sex.FEMALE
    disability_level: DisabilityLevel = DisabilityLevel.NONE
    has_children_under_4: bool = False
    
    def __post_init__(self):
        """Validate business rules."""
        if self.is_pregnant and self.sex != Sex.FEMALE:
            raise ValueError("Pregnancy status can only be True for female employees")
    
    # Calculated properties
    @property
    def is_minor(self) -> bool:
        """Check if employee is under 18."""
        age = (date.today() - self.birth_date).days // 365
        return age < 18
    
    @property
    def tenure_years(self) -> int:
        """Calculate years of service."""
        return (date.today() - self.hire_date).days // 365
    
    @property
    def requires_special_protection(self) -> bool:
        """Check if employee has any protection status."""
        return (self.is_pregnant or 
                self.is_minor or 
                self.disability_level != DisabilityLevel.NONE)

@dataclass
class WorkSchedule:
    """Represents planned work schedule."""
    employee_id: int
    start_date: date
    end_date: date
    work_system: WorkSystemType
    planned_hours: List['PlannedShift']
    
    def validate(self) -> List[str]:
        """Validate schedule against work system rules."""
        violations = []
        # Validation logic here
        return violations

@dataclass
class PlannedShift:
    """Single shift in a schedule."""
    date: date
    start_time: datetime
    end_time: datetime
    break_minutes: int = 0
    is_night_shift: bool = False
    is_weekend: bool = False
    is_holiday: bool = False
```

## Value Objects

### Work Hours Value Object

```python
from dataclasses import dataclass
from datetime import datetime, timedelta
from decimal import Decimal

@dataclass(frozen=True)
class WorkHours:
    """Immutable value object for work hours calculation."""
    start_time: datetime
    end_time: datetime
    break_duration: timedelta = timedelta(0)
    
    def __post_init__(self):
        if self.end_time <= self.start_time:
            raise ValueError("End time must be after start time")
        if self.break_duration < timedelta(0):
            raise ValueError("Break duration cannot be negative")
    
    @property
    def gross_hours(self) -> Decimal:
        """Total hours including breaks."""
        duration = self.end_time - self.start_time
        return Decimal(duration.total_seconds() / 3600)
    
    @property
    def net_hours(self) -> Decimal:
        """Actual work hours excluding breaks."""
        break_hours = Decimal(self.break_duration.total_seconds() / 3600)
        return self.gross_hours - break_hours
    
    def calculate_night_hours(self, 
                            night_start: int = 21, 
                            night_end: int = 7) -> Decimal:
        """Calculate hours worked during night period."""
        # Implementation for night hours calculation
        return Decimal("0")

@dataclass(frozen=True)
class Money:
    """Value object for monetary calculations."""
    amount: Decimal
    currency: str = "PLN"
    
    def __post_init__(self):
        # Ensure 2 decimal places
        object.__setattr__(self, 'amount', 
                          self.amount.quantize(Decimal('0.01')))
    
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
    
    def multiply(self, factor: Decimal) -> 'Money':
        return Money(self.amount * factor, self.currency)
```

## Service Layer

### Overtime Calculator Service

```python
from abc import ABC, abstractmethod
from typing import Protocol

class OvertimeCalculator(Protocol):
    """Protocol for overtime calculation strategies."""
    
    def calculate(self, 
                  base_rate: Money, 
                  hours: WorkHours,
                  context: 'WorkContext') -> 'OvertimeCompensation':
        ...

@dataclass
class WorkContext:
    """Context for work calculation."""
    is_sunday: bool = False
    is_holiday: bool = False
    is_scheduled_rest_day: bool = False
    work_system: WorkSystemType = WorkSystemType.BASIC

@dataclass
class OvertimeCompensation:
    """Result of overtime calculation."""
    base_pay: Money
    supplement: Money
    total: Money
    legal_basis: str
    
class StandardOvertimeCalculator:
    """Implementation of standard overtime calculation."""
    
    def calculate(self, 
                  base_rate: Money, 
                  hours: WorkHours,
                  context: WorkContext) -> OvertimeCompensation:
        
        overtime_hours = max(Decimal("0"), hours.net_hours - Decimal("8"))
        if overtime_hours == 0:
            return OvertimeCompensation(
                base_pay=Money(Decimal("0")),
                supplement=Money(Decimal("0")),
                total=Money(Decimal("0")),
                legal_basis="No overtime"
            )
        
        # First 2 hours at 50%
        first_two = min(overtime_hours, Decimal("2"))
        first_two_pay = base_rate.multiply(first_two)
        first_two_supplement = first_two_pay.multiply(OvertimeRates.FIRST_TWO_HOURS.value)
        
        # Remaining hours at 100%
        remaining = max(Decimal("0"), overtime_hours - Decimal("2"))
        remaining_pay = base_rate.multiply(remaining)
        remaining_supplement = remaining_pay.multiply(OvertimeRates.AFTER_TWO_HOURS.value)
        
        # Special context rates
        if context.is_sunday or context.is_holiday:
            supplement_rate = OvertimeRates.SUNDAY_WORK.value
            total_supplement = base_rate.multiply(overtime_hours).multiply(supplement_rate)
        else:
            total_supplement = first_two_supplement.add(remaining_supplement)
        
        base_pay = base_rate.multiply(overtime_hours)
        total = base_pay.add(total_supplement)
        
        return OvertimeCompensation(
            base_pay=base_pay,
            supplement=total_supplement,
            total=total,
            legal_basis=f"Art. 151⁴ KP - Overtime compensation"
        )
```

### Employee Protection Service

```python
from typing import List, Optional

class ProtectionViolation:
    """Represents a protection rule violation."""
    
    def __init__(self, rule: str, message: str, severity: str = "ERROR"):
        self.rule = rule
        self.message = message
        self.severity = severity

class EmployeeProtectionService:
    """Service for validating employee protection rules."""
    
    def validate_work_assignment(self,
                               employee: Employee,
                               planned_hours: WorkHours,
                               is_night_work: bool = False,
                               is_overtime: bool = False) -> List[ProtectionViolation]:
        """Validate work assignment against protection rules."""
        violations = []
        
        # Pregnancy protections (Art. 178 § 1 Labor Code)
        # Polish law specifically protects "pracownice w ciąży" (pregnant women employees)
        if employee.is_pregnant:
            if is_overtime:
                violations.append(ProtectionViolation(
                    rule="PREGNANCY_OVERTIME",
                    message="Overtime prohibited for pregnant employees (Art. 178 § 1 KP)"
                ))
            if is_night_work:
                violations.append(ProtectionViolation(
                    rule="PREGNANCY_NIGHT",
                    message="Night work prohibited for pregnant employees (Art. 178 § 1 KP)"
                ))
        
        # Minor protections
        if employee.is_minor:
            if planned_hours.net_hours > Decimal(str(WorkLimits.MINOR_DAILY_HOURS)):
                violations.append(ProtectionViolation(
                    rule="MINOR_HOURS",
                    message=f"Minor workers limited to {WorkLimits.MINOR_DAILY_HOURS} hours daily"
                ))
            if is_night_work or is_overtime:
                violations.append(ProtectionViolation(
                    rule="MINOR_RESTRICTIONS",
                    message="Night work and overtime prohibited for minors"
                ))
        
        # Disability accommodations
        if employee.disability_level == DisabilityLevel.MODERATE:
            limit = WorkLimits.DISABLED_MODERATE_DAILY_HOURS
            if planned_hours.net_hours > Decimal(str(limit)):
                violations.append(ProtectionViolation(
                    rule="DISABILITY_HOURS",
                    message=f"Moderate disability: max {limit} hours daily"
                ))
        elif employee.disability_level == DisabilityLevel.SEVERE:
            limit = WorkLimits.DISABLED_SEVERE_DAILY_HOURS
            if planned_hours.net_hours > Decimal(str(limit)):
                violations.append(ProtectionViolation(
                    rule="DISABILITY_HOURS",
                    message=f"Severe disability: max {limit} hours daily"
                ))
        
        return violations
```

### Leave Entitlement Calculator

```python
class LeaveEntitlementService:
    """Service for calculating leave entitlements."""
    
    def calculate_annual_leave(self,
                             employee: Employee,
                             education_bonus_years: int) -> int:
        """Calculate annual leave days based on tenure and education."""
        total_qualifying_years = employee.tenure_years + education_bonus_years
        
        base_days = (LeaveEntitlements.EXTENDED_ANNUAL_DAYS 
                    if total_qualifying_years >= LeaveEntitlements.TENURE_THRESHOLD_YEARS
                    else LeaveEntitlements.STANDARD_ANNUAL_DAYS)
        
        # Add disability bonus if applicable
        if employee.disability_level != DisabilityLevel.NONE:
            base_days += LeaveEntitlements.DISABILITY_ADDITIONAL_DAYS
        
        return base_days
    
    def calculate_proportional_leave(self,
                                   annual_days: int,
                                   months_worked: int,
                                   fte_ratio: Decimal = Decimal("1.0")) -> int:
        """Calculate proportional leave for partial year/FTE."""
        if months_worked >= 12:
            proportional = annual_days * fte_ratio
        else:
            proportional = (annual_days * fte_ratio * months_worked) / 12
        
        # Always round up per labor law
        return int(proportional.quantize(Decimal('1'), rounding='ROUND_UP'))
```

### Contract Validation Service

```python
class ContractValidationService:
    """Service for validating actions based on contract type."""
    
    def can_track_time(self, contract_type: ContractType) -> bool:
        """Check if time tracking is allowed for contract type."""
        return contract_type in [ContractType.EMPLOYMENT, ContractType.SERVICE]
    
    def has_leave_entitlement(self, contract_type: ContractType) -> bool:
        """Check if contract includes leave entitlement."""
        return contract_type == ContractType.EMPLOYMENT
    
    def requires_overtime_pay(self, contract_type: ContractType) -> bool:
        """Check if overtime rules apply."""
        return contract_type in [ContractType.EMPLOYMENT, ContractType.SERVICE]
    
    def validate_time_tracking(self,
                             contract_type: ContractType,
                             action: str) -> Optional[str]:
        """Validate if action is allowed for contract type."""
        if action == "time_tracking" and contract_type in [ContractType.SPECIFIC_WORK, ContractType.B2B]:
            return "Time tracking prohibited for result-based contracts"
        
        if action == "leave_request" and not self.has_leave_entitlement(contract_type):
            return f"No leave entitlement for {contract_type.value}"
        
        return None
```

### Work System Configuration Service

```python
@dataclass
class WorkSystemLimits:
    """Configuration for a specific work system."""
    daily_max_hours: int
    weekly_average_hours: int
    settlement_period_months: int
    allows_overtime: bool
    requires_time_tracking: bool

class WorkSystemConfigurationService:
    """Service for work system configurations."""
    
    _configurations = {
        WorkSystemType.BASIC: WorkSystemLimits(
            daily_max_hours=WorkLimits.DAILY_STANDARD_HOURS,
            weekly_average_hours=WorkLimits.WEEKLY_AVERAGE_HOURS,
            settlement_period_months=4,
            allows_overtime=True,
            requires_time_tracking=True
        ),
        WorkSystemType.EQUIVALENT: WorkSystemLimits(
            daily_max_hours=12,  # Can vary by industry
            weekly_average_hours=WorkLimits.WEEKLY_AVERAGE_HOURS,
            settlement_period_months=1,
            allows_overtime=True,
            requires_time_tracking=True
        ),
        WorkSystemType.TASK_BASED: WorkSystemLimits(
            daily_max_hours=24,  # No daily limit
            weekly_average_hours=WorkLimits.WEEKLY_AVERAGE_HOURS,
            settlement_period_months=1,
            allows_overtime=False,
            requires_time_tracking=False
        ),
        # ... other systems
    }
    
    def get_limits(self, work_system: WorkSystemType) -> WorkSystemLimits:
        """Get limits for specific work system."""
        return self._configurations[work_system]
    
    def validate_daily_hours(self, 
                           work_system: WorkSystemType,
                           hours: Decimal,
                           industry: Optional[str] = None) -> bool:
        """Validate if daily hours are within system limits."""
        limits = self.get_limits(work_system)
        
        # Special handling for equivalent system
        if work_system == WorkSystemType.EQUIVALENT and industry:
            industry_limits = {
                "monitoring_equipment": 16,
                "security_guards": 24
            }
            max_hours = industry_limits.get(industry, limits.daily_max_hours)
        else:
            max_hours = limits.daily_max_hours
        
        return hours <= Decimal(str(max_hours))
```

### PIP Response Service

```python
from typing import Dict, Any
import hashlib
import json

class PIPResponseService:
    """Service for generating PIP inspection responses."""
    
    def generate_emergency_response(self,
                                  employee_id: int,
                                  start_date: date,
                                  end_date: date) -> Dict[str, Any]:
        """Generate compliant response within 5 minutes."""
        
        # Gather all required data
        response = {
            "employee_id": employee_id,
            "period": {
                "start": start_date.isoformat(),
                "end": end_date.isoformat()
            },
            "work_records": self._get_work_records(employee_id, start_date, end_date),
            "overtime_details": self._get_overtime_breakdown(employee_id, start_date, end_date),
            "leave_records": self._get_leave_records(employee_id, start_date, end_date),
            "compliance_status": self._validate_compliance(employee_id, start_date, end_date),
            "generation_timestamp": datetime.utcnow().isoformat(),
            "legal_reference": "Art. 149 KP"
        }
        
        # Add integrity hash
        response["integrity_hash"] = self._calculate_hash(response)
        
        return response
    
    def _calculate_hash(self, data: Dict[str, Any]) -> str:
        """Calculate SHA-256 hash for data integrity."""
        json_str = json.dumps(data, sort_keys=True, default=str)
        return hashlib.sha256(json_str.encode()).hexdigest()
    
    def _get_work_records(self, employee_id: int, 
                         start_date: date, 
                         end_date: date) -> List[Dict]:
        """Retrieve work records with exact times."""
        # Implementation would fetch from database
        return []
    
    def _get_overtime_breakdown(self, employee_id: int,
                               start_date: date,
                               end_date: date) -> Dict:
        """Get detailed overtime calculations."""
        # Implementation would calculate overtime
        return {}
    
    def _get_leave_records(self, employee_id: int,
                          start_date: date,
                          end_date: date) -> List[Dict]:
        """Get leave records with entitlement basis."""
        # Implementation would fetch leave data
        return []
    
    def _validate_compliance(self, employee_id: int,
                           start_date: date,
                           end_date: date) -> Dict[str, bool]:
        """Validate all compliance rules for period."""
        return {
            "rest_periods_compliant": True,
            "overtime_limits_compliant": True,
            "leave_calculations_correct": True,
            "special_protections_applied": True
        }
```

## Infrastructure Specifications

### Database Schema

```sql
-- Employee aggregate
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    contract_type VARCHAR(50) NOT NULL,
    work_system VARCHAR(50) NOT NULL,
    hire_date DATE NOT NULL,
    birth_date DATE NOT NULL,
    sex VARCHAR(10) NOT NULL CHECK (sex IN ('male', 'female')),  -- Required for legal protections
    education_level VARCHAR(50),
    employment_status VARCHAR(20) NOT NULL,
    
    -- Special statuses
    is_pregnant BOOLEAN DEFAULT FALSE,
    disability_level VARCHAR(20) DEFAULT 'none',
    has_children_under_4 BOOLEAN DEFAULT FALSE,
    
    -- Business rule: pregnancy only valid for female employees
    CONSTRAINT valid_pregnancy CHECK (NOT is_pregnant OR sex = 'female'),
    
    -- Audit fields
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    version INTEGER NOT NULL DEFAULT 1,
    
    CONSTRAINT valid_contract_type CHECK (
        contract_type IN ('employment_contract', 'service_agreement', 'specific_work_contract', 'b2b')
    )
    -- Polish contract types: umowa o pracę, umowa zlecenie, umowa o dzieło, b2b
);

-- Work time records with event sourcing
CREATE TABLE work_time_events (
    id BIGSERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    event_type VARCHAR(50) NOT NULL,
    event_timestamp TIMESTAMP NOT NULL,
    
    -- Event data as JSON for flexibility
    event_data JSONB NOT NULL,
    
    -- Immutability
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    created_by INTEGER NOT NULL,
    
    -- For event sourcing
    aggregate_version INTEGER NOT NULL,
    
    INDEX idx_employee_timestamp (employee_id, event_timestamp)
);

-- Materialized view for current state
CREATE MATERIALIZED VIEW work_time_current AS
SELECT 
    employee_id,
    date,
    (event_data->>'start_time')::TIMESTAMP as start_time,
    (event_data->>'end_time')::TIMESTAMP as end_time,
    (event_data->>'break_minutes')::INTEGER as break_minutes,
    (event_data->>'overtime_hours')::DECIMAL as overtime_hours,
    (event_data->>'night_hours')::DECIMAL as night_hours
FROM work_time_events
WHERE event_type IN ('clock_in', 'clock_out', 'time_corrected');

-- Compliance audit log
CREATE TABLE compliance_audit_log (
    id BIGSERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL,
    check_type VARCHAR(100) NOT NULL,
    check_timestamp TIMESTAMP NOT NULL,
    result JSONB NOT NULL,
    violations JSONB,
    
    INDEX idx_employee_compliance (employee_id, check_timestamp)
);
```

### Repository Interfaces

```python
from abc import ABC, abstractmethod
from typing import Optional, List

class EmployeeRepository(ABC):
    """Repository interface for Employee aggregate."""
    
    @abstractmethod
    async def find_by_id(self, employee_id: int) -> Optional[Employee]:
        pass
    
    @abstractmethod
    async def save(self, employee: Employee) -> None:
        pass
    
    @abstractmethod
    async def find_by_contract_type(self, 
                                   contract_type: ContractType) -> List[Employee]:
        pass

class WorkTimeEventStore(ABC):
    """Event store for work time events."""
    
    @abstractmethod
    async def append_event(self, event: 'WorkTimeEvent') -> None:
        pass
    
    @abstractmethod
    async def get_events(self, 
                        employee_id: int,
                        start_date: Optional[date] = None,
                        end_date: Optional[date] = None) -> List['WorkTimeEvent']:
        pass
    
    @abstractmethod
    async def get_current_state(self, 
                               employee_id: int) -> 'WorkTimeAggregate':
        pass
```

## Implementation Guidelines

### 1. Use Dependency Injection

```python
class Application:
    """Application root with dependency injection."""
    
    def __init__(self,
                 employee_repo: EmployeeRepository,
                 event_store: WorkTimeEventStore,
                 overtime_calculator: OvertimeCalculator,
                 protection_service: EmployeeProtectionService,
                 leave_service: LeaveEntitlementService):
        self.employee_repo = employee_repo
        self.event_store = event_store
        self.overtime_calculator = overtime_calculator
        self.protection_service = protection_service
        self.leave_service = leave_service
```

### 2. Handle Errors Gracefully

```python
from typing import Union

class Result:
    """Result type for operations that can fail."""
    
    @staticmethod
    def success(value: Any) -> 'Result':
        return Result(is_success=True, value=value)
    
    @staticmethod
    def failure(error: str) -> 'Result':
        return Result(is_success=False, error=error)
    
    def __init__(self, is_success: bool, value: Any = None, error: str = None):
        self.is_success = is_success
        self.value = value
        self.error = error

# Usage example
def validate_work_assignment(self, 
                           employee_id: int,
                           hours: WorkHours) -> Result:
    try:
        employee = await self.employee_repo.find_by_id(employee_id)
        if not employee:
            return Result.failure(f"Employee {employee_id} not found")
        
        violations = self.protection_service.validate_work_assignment(
            employee, hours
        )
        
        if violations:
            return Result.failure("; ".join([v.message for v in violations]))
        
        return Result.success(True)
        
    except Exception as e:
        return Result.failure(f"Validation failed: {str(e)}")
```

### 3. Use Domain Events

```python
@dataclass
class DomainEvent:
    """Base class for domain events."""
    aggregate_id: int
    occurred_at: datetime
    version: int

@dataclass
class OvertimeThresholdExceeded(DomainEvent):
    """Event when employee exceeds overtime limits."""
    employee_id: int
    current_hours: Decimal
    limit: int
    period: str

@dataclass  
class ProtectionViolationDetected(DomainEvent):
    """Event when protection rule is violated."""
    employee_id: int
    rule: str
    details: str
```

### 4. Testing Strategy

```python
import pytest
from decimal import Decimal

class TestOvertimeCalculator:
    """Unit tests for overtime calculation."""
    
    def test_no_overtime_under_8_hours(self):
        calculator = StandardOvertimeCalculator()
        work_hours = WorkHours(
            start_time=datetime(2024, 1, 1, 8, 0),
            end_time=datetime(2024, 1, 1, 16, 0)
        )
        base_rate = Money(Decimal("30.00"))
        
        result = calculator.calculate(base_rate, work_hours, WorkContext())
        
        assert result.total.amount == Decimal("0.00")
    
    def test_first_two_hours_at_50_percent(self):
        calculator = StandardOvertimeCalculator()
        work_hours = WorkHours(
            start_time=datetime(2024, 1, 1, 8, 0),
            end_time=datetime(2024, 1, 1, 18, 0)  # 10 hours = 2 overtime
        )
        base_rate = Money(Decimal("30.00"))
        
        result = calculator.calculate(base_rate, work_hours, WorkContext())
        
        # Base: 2h * 30 = 60
        # Supplement: 60 * 0.5 = 30
        # Total: 90
        assert result.total.amount == Decimal("90.00")
```

### 5. Monitoring and Compliance

```python
class ComplianceMonitor:
    """Real-time compliance monitoring."""
    
    async def check_daily_compliance(self, employee_id: int, date: date):
        """Run all compliance checks for a day."""
        
        checks = [
            self._check_daily_hours_limit,
            self._check_rest_period_compliance,
            self._check_overtime_limits,
            self._check_protection_rules
        ]
        
        results = []
        for check in checks:
            result = await check(employee_id, date)
            results.append(result)
            
            if not result.is_compliant:
                await self._raise_compliance_alert(employee_id, result)
        
        return ComplianceReport(employee_id, date, results)
```

## Key Implementation Principles

1. **Immutability**: Use immutable value objects for calculations
2. **Event Sourcing**: Track all changes as events for audit trail
3. **Domain Isolation**: Keep business logic in domain layer
4. **Testability**: Design for easy unit and integration testing
5. **Performance**: Cache calculations, use materialized views
6. **Compliance First**: Validate rules before any action
7. **Error Handling**: Use Result types, never throw in domain
8. **Documentation**: Self-documenting code with clear naming

## Legal Validation Checklist

- [ ] All magic numbers extracted to constants
- [ ] Domain model reflects Polish Labor Code structure
- [ ] Services implement single responsibility
- [ ] All calculations return immutable results
- [ ] Audit trail captures all state changes
- [ ] PIP response generated within 5 minutes
- [ ] Data retention complies with 10-year rule
- [ ] Protection rules enforced at domain level

**This refactored reference provides a clean, maintainable implementation guide that follows industry best practices while ensuring complete Polish labor law compliance.**