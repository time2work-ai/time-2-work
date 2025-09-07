---
title: "Time2Work MVP Database Schema"
description: "Simple PostgreSQL schema focused on Polish labor law compliance"
version: "1.0"
date: "2025-09-07"
type: "Database Design"
audience: "Backend Developers"
document_number: "02"
input: "01_mvp_architecture.md"
output: "Database implementation"
---

# Time2Work MVP Database Schema

## Overview

Simple, focused database schema that handles Polish labor law compliance without overengineering. No event sourcing, just straightforward tables.

## Schema

```sql
-- Create database
CREATE DATABASE time2work;

-- Extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Enums for constrained fields
CREATE TYPE contract_type AS ENUM (
    'employment_contract'
    -- Phase 2: 'service_agreement', 'specific_work', 'b2b'
);

CREATE TYPE work_system AS ENUM (
    'basic'
    -- Phase 2: 'equivalent', 'task_based', 'flexible', 'interrupted', 'continuous', 'weekend', 'shortened_week'
);

CREATE TYPE education_level AS ENUM (
    'basic_vocational',      -- 3 years bonus
    'general_secondary',     -- 4 years bonus
    'secondary_vocational',  -- 5 years bonus
    'post_secondary',        -- 6 years bonus
    'higher_education'       -- 8 years bonus
);

CREATE TYPE leave_type AS ENUM (
    'annual',
    'on_demand',
    'sick',
    'maternity',
    'paternity',
    'childcare',
    'unpaid'
);

CREATE TYPE entry_status AS ENUM (
    'draft',
    'submitted', 
    'approved',
    'rejected'
);

-- Employees table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    supertokens_user_id UUID UNIQUE, -- For future auth integration
    
    -- Basic info
    employee_number VARCHAR(50) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    
    -- Employment details
    hire_date DATE NOT NULL,
    birth_date DATE NOT NULL,
    contract_type contract_type NOT NULL DEFAULT 'employment_contract',
    work_system work_system NOT NULL DEFAULT 'basic',
    fte_ratio DECIMAL(3,2) NOT NULL DEFAULT 1.0 CHECK (fte_ratio > 0 AND fte_ratio <= 1),
    
    -- Leave calculation fields
    education_level education_level,
    work_experience_years INTEGER NOT NULL DEFAULT 0,
    
    -- Calculated leave entitlement (updated via trigger)
    annual_leave_days INTEGER NOT NULL DEFAULT 20,
    
    -- Special statuses (Phase 2)
    -- is_pregnant BOOLEAN DEFAULT FALSE,
    -- disability_level VARCHAR(20),
    -- has_children_under_4 BOOLEAN DEFAULT FALSE,
    
    -- Metadata
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Time entries table
CREATE TABLE time_entries (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    date DATE NOT NULL,
    
    -- Clock times
    clock_in TIMESTAMP WITH TIME ZONE,
    clock_out TIMESTAMP WITH TIME ZONE,
    
    -- Break tracking (minutes)
    break_minutes INTEGER DEFAULT 0,
    
    -- Calculated hours (via trigger)
    total_hours DECIMAL(4,2),
    regular_hours DECIMAL(4,2),
    overtime_50_hours DECIMAL(4,2) DEFAULT 0,
    overtime_100_hours DECIMAL(4,2) DEFAULT 0,
    night_hours DECIMAL(4,2) DEFAULT 0,
    
    -- Context flags
    is_sunday BOOLEAN DEFAULT FALSE,
    is_holiday BOOLEAN DEFAULT FALSE,
    
    -- Workflow
    status entry_status DEFAULT 'draft',
    submitted_at TIMESTAMP,
    approved_by_id INTEGER REFERENCES employees(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_employee_date UNIQUE(employee_id, date),
    CONSTRAINT valid_clock_times CHECK (clock_out IS NULL OR clock_out > clock_in)
);

-- Leave balances table
CREATE TABLE leave_balances (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    year INTEGER NOT NULL,
    
    -- Entitlements
    total_days DECIMAL(4,1) NOT NULL,
    
    -- Usage
    used_days DECIMAL(4,1) DEFAULT 0,
    pending_days DECIMAL(4,1) DEFAULT 0,
    on_demand_used INTEGER DEFAULT 0,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_employee_year UNIQUE(employee_id, year),
    CONSTRAINT max_on_demand CHECK (on_demand_used <= 4)
);

-- Leave requests table  
CREATE TABLE leave_requests (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    
    -- Request details
    leave_type leave_type NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    total_days DECIMAL(4,1) NOT NULL,
    is_on_demand BOOLEAN DEFAULT FALSE,
    reason TEXT,
    
    -- Workflow
    status entry_status DEFAULT 'draft',
    submitted_at TIMESTAMP,
    approved_by_id INTEGER REFERENCES employees(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT valid_dates CHECK (end_date >= start_date),
    CONSTRAINT on_demand_only_annual CHECK (
        NOT is_on_demand OR leave_type = 'annual'
    )
);

-- Indexes for performance
CREATE INDEX idx_time_entries_employee_date ON time_entries(employee_id, date DESC);
CREATE INDEX idx_time_entries_status ON time_entries(status) WHERE status != 'approved';
CREATE INDEX idx_leave_requests_employee ON leave_requests(employee_id);
CREATE INDEX idx_leave_requests_dates ON leave_requests(start_date, end_date);
CREATE INDEX idx_leave_balances_employee_year ON leave_balances(employee_id, year DESC);

-- Helper function to calculate education bonus years
CREATE OR REPLACE FUNCTION get_education_bonus_years(edu education_level) 
RETURNS INTEGER AS $$
BEGIN
    RETURN CASE edu
        WHEN 'basic_vocational' THEN 3
        WHEN 'general_secondary' THEN 4  
        WHEN 'secondary_vocational' THEN 5
        WHEN 'post_secondary' THEN 6
        WHEN 'higher_education' THEN 8
        ELSE 0
    END;
END;
$$ LANGUAGE plpgsql;

-- Function to calculate leave entitlement
CREATE OR REPLACE FUNCTION calculate_leave_entitlement() 
RETURNS TRIGGER AS $$
DECLARE
    education_bonus INTEGER;
    total_tenure INTEGER;
    base_days INTEGER;
BEGIN
    -- Get education bonus
    education_bonus := get_education_bonus_years(NEW.education_level);
    
    -- Calculate total tenure
    total_tenure := NEW.work_experience_years + education_bonus;
    
    -- Determine base days (20 or 26)
    IF total_tenure >= 10 THEN
        base_days := 26;
    ELSE
        base_days := 20;
    END IF;
    
    -- Apply FTE ratio and round up
    NEW.annual_leave_days := CEIL(base_days * NEW.fte_ratio);
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to update leave entitlement
CREATE TRIGGER update_leave_entitlement
BEFORE INSERT OR UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION calculate_leave_entitlement();

-- Function to calculate overtime
CREATE OR REPLACE FUNCTION calculate_overtime() 
RETURNS TRIGGER AS $$
DECLARE
    work_hours DECIMAL;
BEGIN
    -- Only calculate if we have both clock in and out
    IF NEW.clock_in IS NOT NULL AND NEW.clock_out IS NOT NULL THEN
        -- Calculate total hours worked
        work_hours := EXTRACT(EPOCH FROM (NEW.clock_out - NEW.clock_in)) / 3600.0;
        
        -- Subtract break time
        work_hours := work_hours - (NEW.break_minutes / 60.0);
        
        NEW.total_hours := work_hours;
        
        -- Calculate overtime (simplified for MVP - basic system only)
        IF work_hours > 8 THEN
            -- First 2 hours of overtime at 50%
            NEW.overtime_50_hours := LEAST(work_hours - 8, 2);
            -- Remaining overtime at 100%
            NEW.overtime_100_hours := GREATEST(work_hours - 10, 0);
            NEW.regular_hours := 8;
        ELSE
            NEW.regular_hours := work_hours;
            NEW.overtime_50_hours := 0;
            NEW.overtime_100_hours := 0;
        END IF;
        
        -- Night hours calculation (simplified - assumes 21:00 to 07:00)
        -- TODO: Proper implementation would check actual hours worked during night period
        NEW.night_hours := 0;
        
        -- Set day type flags
        NEW.is_sunday := (EXTRACT(DOW FROM NEW.date) = 0);
        -- TODO: Holiday calendar integration
        NEW.is_holiday := FALSE;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to calculate overtime
CREATE TRIGGER calculate_overtime_trigger
BEFORE INSERT OR UPDATE ON time_entries
FOR EACH ROW
EXECUTE FUNCTION calculate_overtime();

-- Function to update leave balance when request is approved
CREATE OR REPLACE FUNCTION update_leave_balance_on_approval() 
RETURNS TRIGGER AS $$
BEGIN
    -- Only process when status changes to approved
    IF NEW.status = 'approved' AND OLD.status != 'approved' THEN
        -- Update leave balance
        UPDATE leave_balances
        SET used_days = used_days + NEW.total_days,
            on_demand_used = CASE 
                WHEN NEW.is_on_demand THEN on_demand_used + 1 
                ELSE on_demand_used 
            END,
            updated_at = CURRENT_TIMESTAMP
        WHERE employee_id = NEW.employee_id 
        AND year = EXTRACT(YEAR FROM NEW.start_date);
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger for leave balance updates
CREATE TRIGGER update_leave_balance_trigger
AFTER UPDATE ON leave_requests
FOR EACH ROW
EXECUTE FUNCTION update_leave_balance_on_approval();

-- Trigger for updated_at timestamps
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_employees_updated_at BEFORE UPDATE ON employees
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_time_entries_updated_at BEFORE UPDATE ON time_entries
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_leave_balances_updated_at BEFORE UPDATE ON leave_balances
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_leave_requests_updated_at BEFORE UPDATE ON leave_requests
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Sample data for testing
INSERT INTO employees (
    employee_number, first_name, last_name, hire_date, birth_date,
    education_level, work_experience_years
) VALUES 
    ('EMP001', 'Jan', 'Kowalski', '2020-01-15', '1985-03-20', 'higher_education', 5),
    ('EMP002', 'Anna', 'Nowak', '2023-03-01', '1990-07-15', 'secondary_vocational', 2),
    ('MGR001', 'Piotr', 'Wiśniewski', '2018-06-01', '1980-11-30', 'higher_education', 15);

-- Initialize leave balances for current year
INSERT INTO leave_balances (employee_id, year, total_days)
SELECT id, EXTRACT(YEAR FROM CURRENT_DATE), annual_leave_days
FROM employees;
```

## Key Design Decisions

1. **Simple Calculated Fields**: Use triggers to calculate overtime and leave entitlement automatically
2. **No User Table**: SuperTokens will own authentication, we just map to their user IDs
3. **Status Enums**: Use PostgreSQL enums for type safety
4. **Simplified Overtime**: Only handle basic work system for MVP (8h/day standard)
5. **Audit via Updated Timestamps**: Simple updated_at instead of complex audit tables

## Compliance Features Built-In

1. **Leave Calculation**: Automatic calculation based on tenure and education
2. **Overtime Tracking**: Separate 50% and 100% overtime hours
3. **On-Demand Leave**: Maximum 4 days per year enforced
4. **Unique Constraints**: One time entry per employee per day
5. **FTE Support**: Part-time employees get proportional leave

## Future Considerations (Phase 2+)

- Add special status fields (pregnancy, disability)
- Support multiple work systems
- Holiday calendar integration
- Audit trail table
- Data retention policies (10-year requirement)

## Usage Examples

```sql
-- Clock in
INSERT INTO time_entries (employee_id, date, clock_in)
VALUES (1, CURRENT_DATE, CURRENT_TIMESTAMP);

-- Clock out (triggers overtime calculation)
UPDATE time_entries 
SET clock_out = CURRENT_TIMESTAMP, break_minutes = 30
WHERE employee_id = 1 AND date = CURRENT_DATE;

-- Request leave
INSERT INTO leave_requests (
    employee_id, leave_type, start_date, end_date, total_days
) VALUES (
    1, 'annual', '2025-10-01', '2025-10-05', 5
);

-- Approve leave (triggers balance update)
UPDATE leave_requests 
SET status = 'approved', approved_by_id = 3, approved_at = CURRENT_TIMESTAMP
WHERE id = 1;
```

This schema provides the minimum viable compliance for Polish labor law while keeping things simple for rapid MVP development.