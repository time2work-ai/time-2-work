---
title: "Time2Work MVP API Design"
description: "FastAPI endpoints for MVP with Polish labor law compliance"
version: "1.0"
date: "2025-09-07"
type: "API Design"
audience: "Backend Developers"
document_number: "03"
input: "02_mvp_database.md"
output: "API implementation"
---

# Time2Work MVP API Design

## Overview

Simple FastAPI implementation focused on core time tracking and compliance. No authentication middleware (SuperTokens integration later).

## Base Configuration

```python
# app/main.py
from fastapi import FastAPI, Header
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print("Starting up...")
    yield
    # Shutdown
    print("Shutting down...")

app = FastAPI(
    title="Time2Work MVP API",
    version="1.0.0",
    lifespan=lifespan
)

# For MVP: Simulate auth with header
def get_current_employee_id(x_employee_id: int = Header()) -> int:
    return x_employee_id
```

## Core Models (Pydantic)

```python
# app/schemas.py
from pydantic import BaseModel, Field, ConfigDict
from datetime import date, datetime, time
from decimal import Decimal
from typing import Optional
from enum import Enum

class ContractType(str, Enum):
    employment_contract = "employment_contract"

class WorkSystem(str, Enum):
    basic = "basic"

class LeaveType(str, Enum):
    annual = "annual"
    on_demand = "on_demand"
    sick = "sick"
    unpaid = "unpaid"

class EntryStatus(str, Enum):
    draft = "draft"
    submitted = "submitted"
    approved = "approved"
    rejected = "rejected"

# Employee models
class EmployeeBase(BaseModel):
    employee_number: str
    first_name: str
    last_name: str
    hire_date: date
    birth_date: date
    contract_type: ContractType = ContractType.employment_contract
    work_system: WorkSystem = WorkSystem.basic
    fte_ratio: Decimal = Field(default=Decimal("1.0"), ge=0, le=1)
    education_level: Optional[str] = None
    work_experience_years: int = 0

class EmployeeResponse(EmployeeBase):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    annual_leave_days: int
    is_active: bool
    created_at: datetime
    
class EmployeeWithBalance(EmployeeResponse):
    leave_balance: Optional['LeaveBalanceResponse'] = None

# Time entry models
class ClockInRequest(BaseModel):
    timestamp: datetime = Field(default_factory=datetime.now)
    
class ClockOutRequest(BaseModel):
    timestamp: datetime = Field(default_factory=datetime.now)
    break_minutes: int = Field(default=0, ge=0)

class TimeEntryResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    employee_id: int
    date: date
    clock_in: Optional[datetime]
    clock_out: Optional[datetime]
    break_minutes: int
    total_hours: Optional[Decimal]
    regular_hours: Optional[Decimal]
    overtime_50_hours: Decimal
    overtime_100_hours: Decimal
    status: EntryStatus
    created_at: datetime

# Leave models
class LeaveRequestCreate(BaseModel):
    leave_type: LeaveType
    start_date: date
    end_date: date
    is_on_demand: bool = False
    reason: Optional[str] = None

class LeaveRequestResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    employee_id: int
    leave_type: LeaveType
    start_date: date
    end_date: date
    total_days: Decimal
    is_on_demand: bool
    status: EntryStatus
    created_at: datetime

class LeaveBalanceResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    
    year: int
    total_days: Decimal
    used_days: Decimal
    pending_days: Decimal
    on_demand_used: int
    remaining_days: Decimal
    on_demand_remaining: int
```

## API Endpoints

### Employee Endpoints

```python
# app/api/employees.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List

router = APIRouter(prefix="/employees", tags=["employees"])

@router.get("/me", response_model=EmployeeWithBalance)
async def get_current_employee(
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Get current employee profile with leave balance."""
    employee = db.query(Employee).filter(Employee.id == employee_id).first()
    if not employee:
        raise HTTPException(404, "Employee not found")
    
    # Get current year balance
    current_year = date.today().year
    balance = db.query(LeaveBalance).filter(
        LeaveBalance.employee_id == employee_id,
        LeaveBalance.year == current_year
    ).first()
    
    return EmployeeWithBalance(
        **employee.__dict__,
        leave_balance=balance
    )

@router.get("/team", response_model=List[EmployeeResponse])
async def get_team_members(
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Get team members (managers only - MVP assumes employee is manager)."""
    # MVP: Return all active employees
    employees = db.query(Employee).filter(Employee.is_active == True).all()
    return employees
```

### Time Entry Endpoints

```python
# app/api/time_entries.py
from fastapi import APIRouter, Depends, HTTPException, Query
from datetime import date as date_type, datetime, timedelta
from typing import List, Optional

router = APIRouter(prefix="/time-entries", tags=["time"])

@router.post("/clock-in", response_model=TimeEntryResponse)
async def clock_in(
    request: ClockInRequest,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Clock in for the current day."""
    today = request.timestamp.date()
    
    # Check if already clocked in
    existing = db.query(TimeEntry).filter(
        TimeEntry.employee_id == employee_id,
        TimeEntry.date == today
    ).first()
    
    if existing and existing.clock_in:
        raise HTTPException(400, "Already clocked in today")
    
    # Create or update entry
    if not existing:
        entry = TimeEntry(
            employee_id=employee_id,
            date=today,
            clock_in=request.timestamp
        )
        db.add(entry)
    else:
        existing.clock_in = request.timestamp
        entry = existing
    
    db.commit()
    db.refresh(entry)
    return entry

@router.post("/clock-out", response_model=TimeEntryResponse)
async def clock_out(
    request: ClockOutRequest,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Clock out for the current day."""
    today = request.timestamp.date()
    
    # Get today's entry
    entry = db.query(TimeEntry).filter(
        TimeEntry.employee_id == employee_id,
        TimeEntry.date == today
    ).first()
    
    if not entry or not entry.clock_in:
        raise HTTPException(400, "Not clocked in today")
    
    if entry.clock_out:
        raise HTTPException(400, "Already clocked out today")
    
    # Validate rest period (11 hours)
    yesterday = db.query(TimeEntry).filter(
        TimeEntry.employee_id == employee_id,
        TimeEntry.date == today - timedelta(days=1)
    ).first()
    
    if yesterday and yesterday.clock_out:
        rest_hours = (entry.clock_in - yesterday.clock_out).total_seconds() / 3600
        if rest_hours < 11:
            raise HTTPException(
                400, 
                f"Insufficient rest period: {rest_hours:.1f} hours (minimum 11 required)"
            )
    
    # Update entry
    entry.clock_out = request.timestamp
    entry.break_minutes = request.break_minutes
    
    db.commit()
    db.refresh(entry)
    return entry

@router.get("/", response_model=List[TimeEntryResponse])
async def get_time_entries(
    start_date: Optional[date_type] = None,
    end_date: Optional[date_type] = None,
    status: Optional[EntryStatus] = None,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Get time entries for current employee."""
    query = db.query(TimeEntry).filter(TimeEntry.employee_id == employee_id)
    
    if start_date:
        query = query.filter(TimeEntry.date >= start_date)
    if end_date:
        query = query.filter(TimeEntry.date <= end_date)
    if status:
        query = query.filter(TimeEntry.status == status)
    
    return query.order_by(TimeEntry.date.desc()).all()

@router.put("/{entry_id}/submit", response_model=TimeEntryResponse)
async def submit_time_entry(
    entry_id: int,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Submit time entry for approval."""
    entry = db.query(TimeEntry).filter(
        TimeEntry.id == entry_id,
        TimeEntry.employee_id == employee_id
    ).first()
    
    if not entry:
        raise HTTPException(404, "Time entry not found")
    
    if entry.status != EntryStatus.draft:
        raise HTTPException(400, f"Cannot submit entry in {entry.status} status")
    
    if not entry.clock_in or not entry.clock_out:
        raise HTTPException(400, "Cannot submit incomplete entry")
    
    entry.status = EntryStatus.submitted
    entry.submitted_at = datetime.now()
    
    db.commit()
    db.refresh(entry)
    return entry

@router.put("/{entry_id}/approve", response_model=TimeEntryResponse)
async def approve_time_entry(
    entry_id: int,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Approve time entry (managers only)."""
    entry = db.query(TimeEntry).filter(TimeEntry.id == entry_id).first()
    
    if not entry:
        raise HTTPException(404, "Time entry not found")
    
    if entry.status != EntryStatus.submitted:
        raise HTTPException(400, f"Cannot approve entry in {entry.status} status")
    
    entry.status = EntryStatus.approved
    entry.approved_by_id = employee_id
    entry.approved_at = datetime.now()
    
    db.commit()
    db.refresh(entry)
    return entry
```

### Leave Management Endpoints

```python
# app/api/leave.py
from fastapi import APIRouter, Depends, HTTPException
from typing import List

router = APIRouter(prefix="/leave", tags=["leave"])

@router.post("/requests", response_model=LeaveRequestResponse)
async def create_leave_request(
    request: LeaveRequestCreate,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Create a new leave request."""
    # Calculate total days
    total_days = (request.end_date - request.start_date).days + 1
    
    # Get current balance
    year = request.start_date.year
    balance = db.query(LeaveBalance).filter(
        LeaveBalance.employee_id == employee_id,
        LeaveBalance.year == year
    ).first()
    
    if not balance:
        raise HTTPException(400, f"No leave balance for year {year}")
    
    # Check available days
    available = balance.total_days - balance.used_days - balance.pending_days
    if total_days > available:
        raise HTTPException(
            400, 
            f"Insufficient leave balance: {available} days available, {total_days} requested"
        )
    
    # Check on-demand limit
    if request.is_on_demand and balance.on_demand_used >= 4:
        raise HTTPException(400, "On-demand leave limit reached (4 days per year)")
    
    # Create request
    leave_request = LeaveRequest(
        employee_id=employee_id,
        leave_type=request.leave_type,
        start_date=request.start_date,
        end_date=request.end_date,
        total_days=total_days,
        is_on_demand=request.is_on_demand,
        reason=request.reason,
        status=EntryStatus.submitted,
        submitted_at=datetime.now()
    )
    
    # Update pending days
    balance.pending_days += total_days
    
    db.add(leave_request)
    db.commit()
    db.refresh(leave_request)
    return leave_request

@router.get("/requests", response_model=List[LeaveRequestResponse])
async def get_leave_requests(
    status: Optional[EntryStatus] = None,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Get leave requests for current employee."""
    query = db.query(LeaveRequest).filter(LeaveRequest.employee_id == employee_id)
    
    if status:
        query = query.filter(LeaveRequest.status == status)
    
    return query.order_by(LeaveRequest.start_date.desc()).all()

@router.put("/requests/{request_id}/approve", response_model=LeaveRequestResponse)
async def approve_leave_request(
    request_id: int,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Approve leave request (managers only)."""
    leave_request = db.query(LeaveRequest).filter(
        LeaveRequest.id == request_id
    ).first()
    
    if not leave_request:
        raise HTTPException(404, "Leave request not found")
    
    if leave_request.status != EntryStatus.submitted:
        raise HTTPException(400, f"Cannot approve request in {leave_request.status} status")
    
    # Update request
    leave_request.status = EntryStatus.approved
    leave_request.approved_by_id = employee_id
    leave_request.approved_at = datetime.now()
    
    # Update balance (trigger handles this)
    db.commit()
    db.refresh(leave_request)
    return leave_request

@router.get("/balance/{year}", response_model=LeaveBalanceResponse)
async def get_leave_balance(
    year: int,
    employee_id: int = Depends(get_current_employee_id),
    db: Session = Depends(get_db)
):
    """Get leave balance for specific year."""
    balance = db.query(LeaveBalance).filter(
        LeaveBalance.employee_id == employee_id,
        LeaveBalance.year == year
    ).first()
    
    if not balance:
        raise HTTPException(404, f"No leave balance for year {year}")
    
    # Calculate remaining days
    balance.remaining_days = balance.total_days - balance.used_days - balance.pending_days
    balance.on_demand_remaining = 4 - balance.on_demand_used
    
    return balance
```

## Compliance Functions

```python
# app/core/compliance.py
from datetime import date, datetime, timedelta
from decimal import Decimal
from typing import Tuple, Optional

def calculate_overtime(worked_hours: Decimal) -> Tuple[Decimal, Decimal, Decimal]:
    """
    Calculate overtime based on Polish labor law.
    Returns: (regular_hours, overtime_50, overtime_100)
    """
    if worked_hours <= 8:
        return worked_hours, Decimal("0"), Decimal("0")
    
    regular = Decimal("8")
    overtime = worked_hours - regular
    
    # First 2 hours at 50%
    overtime_50 = min(overtime, Decimal("2"))
    # Remaining at 100%
    overtime_100 = max(overtime - Decimal("2"), Decimal("0"))
    
    return regular, overtime_50, overtime_100

def calculate_leave_entitlement(
    work_experience_years: int,
    education_level: Optional[str],
    fte_ratio: Decimal = Decimal("1.0")
) -> int:
    """Calculate annual leave days based on Polish labor law."""
    # Education bonus years
    education_bonus = {
        "basic_vocational": 3,
        "general_secondary": 4,
        "secondary_vocational": 5,
        "post_secondary": 6,
        "higher_education": 8
    }.get(education_level, 0)
    
    total_tenure = work_experience_years + education_bonus
    
    # Base entitlement
    base_days = 26 if total_tenure >= 10 else 20
    
    # Apply FTE ratio and round up
    return int(Decimal(base_days * fte_ratio).to_integral_value(rounding="ROUND_UP"))

def validate_rest_period(
    previous_clock_out: datetime,
    next_clock_in: datetime
) -> Tuple[bool, float]:
    """
    Validate 11-hour rest period between shifts.
    Returns: (is_valid, actual_hours)
    """
    rest_duration = next_clock_in - previous_clock_out
    rest_hours = rest_duration.total_seconds() / 3600
    return rest_hours >= 11, rest_hours

def calculate_weekly_hours(time_entries: List[TimeEntry]) -> Decimal:
    """Calculate total hours worked in a week."""
    return sum(
        entry.total_hours or Decimal("0") 
        for entry in time_entries
    )
```

## Error Handling

```python
# app/main.py (additions)
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "message": exc.detail,
                "status_code": exc.status_code
            }
        }
    )

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content={
            "error": {
                "message": str(exc),
                "type": "validation_error"
            }
        }
    )
```

## Running the API

```python
# app/main.py
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

## Testing Examples

```bash
# Clock in
curl -X POST http://localhost:8000/time-entries/clock-in \
  -H "X-Employee-ID: 1" \
  -H "Content-Type: application/json"

# Clock out  
curl -X POST http://localhost:8000/time-entries/clock-out \
  -H "X-Employee-ID: 1" \
  -H "Content-Type: application/json" \
  -d '{"break_minutes": 30}'

# Get my profile
curl http://localhost:8000/employees/me \
  -H "X-Employee-ID: 1"

# Request leave
curl -X POST http://localhost:8000/leave/requests \
  -H "X-Employee-ID: 1" \
  -H "Content-Type: application/json" \
  -d '{
    "leave_type": "annual",
    "start_date": "2025-10-01",
    "end_date": "2025-10-05"
  }'
```

This API provides the core functionality for time tracking with Polish labor law compliance, ready for a 1-2 week MVP implementation.