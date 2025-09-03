---
title: "Data Retention Implementation"
description: "Technical implementation for Polish labor law data retention requirements"
version: "1.0"
date: "2025-09-02"
type: "Technical Reference"
audience: "Development Team, Data Engineers"
priority: "CRITICAL"
---

# Data Retention Implementation

## Retention Requirements

### 1. Retention Period Calculation

```python
def calculate_retention_period(employee: Employee) -> dict:
    """
    Art. 149 KP: 10 years from END of calendar year of employment termination
    NOT just 10 years from termination date
    """
    
    termination_date = employee.termination_date
    
    if termination_date:
        # End of calendar year of termination
        retention_start = date(termination_date.year + 1, 1, 1)
        retention_expires = date(termination_date.year + 11, 1, 1)
        
        return {
            'termination_date': termination_date,
            'retention_start': retention_start,
            'retention_expires': retention_expires,
            'years_remaining': (retention_expires - date.today()).days / 365.25,
            'legal_basis': 'Art. 149 § 1 KP + MRPiPS Regulation'
        }
    
    # Active employee - no retention expiry
    return {
        'status': 'active_employee',
        'retention_expires': None,
        'note': 'Retention period starts after employment termination'
    }
```

### 2. Required Data Structure for PIP Compliance

```sql
-- Complete data structure based on MRPiPS Regulation § 6 pt 1a
CREATE TABLE ewidencja_czasu_pracy (
    -- Primary identification
    id BIGSERIAL PRIMARY KEY,
    pracownik_id INTEGER NOT NULL,
    data_pracy DATE NOT NULL,
    
    -- Contract information (REQUIRED)
    typ_umowy VARCHAR(50) NOT NULL CHECK (typ_umowy IN ('umowa_o_prace', 'umowa_zlecenie')),
    wymiar_etatu DECIMAL(3,2) NOT NULL DEFAULT 1.0,
    
    -- Work time tracking (EXACT times required)
    czas_rozpoczecia TIMESTAMP,  -- EXACT start time
    czas_zakonczenia TIMESTAMP,  -- EXACT end time
    godziny_przepracowane DECIMAL(4,2),
    
    -- Break tracking
    przerwy_platne DECIMAL(4,2),
    przerwy_nieplatne DECIMAL(4,2),
    
    -- Overtime breakdown (DETAILED tracking required)
    nadgodziny_50_procent DECIMAL(4,2),  -- Regular overtime hours
    nadgodziny_100_procent DECIMAL(4,2), -- Holiday/Sunday overtime hours
    nadgodziny_200_procent DECIMAL(4,2), -- Exceptional overtime hours
    
    -- Night work (REQUIRED if applicable)
    praca_nocna_godziny DECIMAL(4,2),
    praca_nocna_okreslenie VARCHAR(100), -- e.g., "22:00-06:00"
    
    -- Weekend and holiday work
    praca_niedziela BOOLEAN DEFAULT FALSE,
    praca_swieto BOOLEAN DEFAULT FALSE,
    praca_dzien_wolny BOOLEAN DEFAULT FALSE,
    
    -- On-call work (LOCATION REQUIRED - missing from current specs)
    dyzur_godziny DECIMAL(4,2),
    dyzur_lokalizacja VARCHAR(200),  -- LEGALLY REQUIRED
    dyzur_typ VARCHAR(50), -- 'w_miejscu_pracy', 'w_domu', 'dyspozycyjny'
    
    -- Absence tracking (DETAILED breakdown required)
    nieobecnosc_typ VARCHAR(50), -- 'urlop', 'choroba', 'okolicznosciowy', etc.
    nieobecnosc_godziny DECIMAL(4,2),
    nieobecnosc_dni DECIMAL(3,1),
    nieobecnosc_usprawiedliwiona BOOLEAN,
    nieobecnosc_powod VARCHAR(200),
    
    -- Minor work tracking (when applicable)
    praca_nieletni_zadania_zabronione DECIMAL(4,2),
    
    -- Audit trail (IMMUTABLE for PIP compliance)
    utworzono_timestamp TIMESTAMP DEFAULT NOW(),
    utworzono_przez_user_id INTEGER NOT NULL,
    zmodyfikowano_timestamp TIMESTAMP,
    zmodyfikowano_przez_user_id INTEGER,
    powod_modyfikacji TEXT,
    hash_integralnosci VARCHAR(64), -- Data integrity verification
    
    -- PIP inspection readiness
    pip_eksport_gotowy BOOLEAN DEFAULT TRUE,
    ostatni_pip_eksport TIMESTAMP,
    
    -- Indexing for performance and compliance
    INDEX idx_pracownik_data (pracownik_id, data_pracy),
    INDEX idx_data_utworzenia (utworzono_timestamp),
    INDEX idx_pip_eksport (pip_eksport_gotowy, pracownik_id),
    
    -- Legal constraints
    CONSTRAINT valid_work_hours CHECK (godziny_przepracowane <= 24),
    CONSTRAINT valid_overtime CHECK (nadgodziny_50_procent + nadgodziny_100_procent + nadgodziny_200_procent >= 0),
    CONSTRAINT required_location_for_oncall CHECK (
        (dyzur_godziny IS NULL) OR 
        (dyzur_godziny IS NOT NULL AND dyzur_lokalizacja IS NOT NULL)
    )
);
```

### 3. Employee Data Access Rights

```python
def employee_data_access_system():
    """
    Employee right to access their own data
    Art. 149 KP + GDPR compliance
    """
    return {
        'self_service_portal': {
            'current_employee_access': {
                'own_time_records': True,
                'leave_balances': True,
                'work_schedules': True,
                'overtime_calculations': True,
                'compliance_status': True
            },
            'former_employee_access': {
                'historical_records': True,
                'termination_calculations': True,
                'leave_payouts': True,
                'access_period': '10_years_post_termination'
            }
        },
        'data_export_capabilities': {
            'personal_export': 'PDF, Excel, CSV formats',
            'date_range_selection': 'Any period within retention',
            'detailed_breakdown': 'All components of pay calculations',
            'legal_certification': 'Digitally signed for legal proceedings'
        },
        'access_logging': {
            'who_accessed': True,
            'when_accessed': True,
            'what_data_viewed': True,
            'export_activities': True,
            'retention_period': '10_years'
        }
    }
```

### 4. Data Archival and Retention System

```python
def data_archival_system():
    """
    Automated archival system for terminated employees
    """
    return {
        'archival_triggers': {
            'employment_termination': 'Immediate staging for archival',
            'calendar_year_end': 'Move to long-term retention',
            'retention_expiry': 'Secure deletion after 10 years'
        },
        'archival_process': {
            'data_integrity_verification': True,
            'encryption_at_rest': True,
            'backup_redundancy': 'Multiple geographic locations',
            'access_logging': 'Every access logged and monitored'
        },
        'retention_verification': {
            'annual_audit': 'Verify data integrity and accessibility',
            'pip_readiness_test': 'Quarterly export simulation',
            'legal_compliance_check': 'Monthly validation of retention periods'
        }
    }
```

### 5. PIP Export System

```python
def pip_audit_export_system(employee_id: int, date_range: DateRange) -> dict:
    """
    Complete PIP inspection export capability
    Enhanced for 2024-2025 inspection requirements
    """
    
    export_data = {
        'employee_profile': {
            'personal_data': get_employee_personal_data(employee_id),
            'contract_history': get_complete_contract_history(employee_id),
            'position_changes': get_position_change_history(employee_id),
            'special_status_periods': get_special_status_history(employee_id)
        },
        
        'work_time_evidence': {
            'daily_records': get_daily_work_records(employee_id, date_range),
            'exact_times': get_start_end_times_detailed(employee_id, date_range),
            'break_records': get_break_tracking(employee_id, date_range),
            'overtime_breakdown': get_overtime_detailed_breakdown(employee_id, date_range)
        },
        
        'on_call_evidence': {
            'on_call_hours': get_on_call_hours(employee_id, date_range),
            'on_call_locations': get_on_call_locations(employee_id, date_range),  # REQUIRED
            'on_call_compensation': get_on_call_pay_calculations(employee_id, date_range)
        },
        
        'leave_evidence': {
            'annual_leave': get_annual_leave_history(employee_id, date_range),
            'sick_leave': get_sick_leave_records(employee_id, date_range),
            'parental_leave': get_parental_leave_chain(employee_id, date_range),
            'circumstantial_leave': get_circumstantial_leave(employee_id, date_range),
            'leave_calculations': get_leave_entitlement_calculations(employee_id)
        },
        
        'compliance_evidence': {
            'rest_period_validation': validate_rest_periods_historical(employee_id, date_range),
            'work_time_limit_compliance': validate_work_limits(employee_id, date_range),
            'special_protection_compliance': validate_special_protections(employee_id, date_range),
            'overtime_limit_compliance': validate_overtime_limits(employee_id, date_range)
        },
        
        'audit_trail': {
            'data_corrections': get_all_corrections_with_reasons(employee_id, date_range),
            'access_log': get_data_access_history(employee_id, date_range),
            'system_changes': get_system_changes_affecting_employee(employee_id, date_range),
            'legal_basis_documentation': get_legal_basis_for_all_entries(employee_id, date_range)
        },
        
        'export_metadata': {
            'generation_timestamp': datetime.utcnow(),
            'generated_by_user': get_current_user(),
            'data_integrity_hash': calculate_export_hash(),
            'legal_certification': 'Certified compliant with Art. 149 KP',
            'pip_format_version': '2024.1',
            'digital_signature': sign_export_legally()
        }
    }
    
    return export_data
```

### 6. Data Backup and Recovery for Legal Compliance

```python
def legal_data_backup_system():
    """
    Backup system designed for legal compliance, not just data recovery
    """
    return {
        'backup_frequency': {
            'real_time': 'Critical compliance data (corrections, violations)',
            'daily': 'Complete work time records',
            'weekly': 'Employee profile changes',
            'monthly': 'Complete system backup'
        },
        
        'backup_verification': {
            'integrity_checks': 'Daily verification of backup completeness',
            'restoration_testing': 'Monthly test of complete data recovery',
            'pip_export_testing': 'Weekly test of export from backup',
            'legal_compliance_validation': 'Quarterly legal expert review'
        },
        
        'geographic_distribution': {
            'primary_location': 'Poland (data residency compliance)',
            'secondary_location': 'EU (GDPR compliance)',
            'tertiary_location': 'Secure cloud (disaster recovery)',
            'access_restrictions': 'Strict legal compliance controls'
        },
        
        'retention_automation': {
            'automatic_archival': 'On employment termination + calendar year end',
            'retention_monitoring': 'Daily check of retention periods',
            'secure_deletion': 'Automatic after 10-year retention expires',
            'deletion_certification': 'Legal certificate of secure deletion'
        }
    }
```

### 7. Employee Access Portal

```python
def employee_data_access_portal():
    """
    Employee self-service access to their data
    Required by Polish labor law and GDPR
    """
    return {
        'current_employee_features': {
            'view_time_records': 'All historical work time data',
            'download_timesheets': 'PDF/Excel export capability',
            'leave_balance_tracking': 'Real-time balance with accrual details',
            'overtime_calculations': 'Detailed breakdown of all calculations',
            'schedule_viewing': 'Current and historical work schedules',
            'compliance_status': 'Personal compliance dashboard'
        },
        
        'former_employee_features': {
            'historical_data_access': '10-year access period',
            'termination_calculations': 'Final pay and leave payout details',
            'employment_certificates': 'Digital employment history certificates',
            'legal_export': 'Legally certified data export for disputes'
        },
        
        'access_security': {
            'multi_factor_authentication': True,
            'session_timeout': 30,  # minutes
            'access_logging': 'Complete audit trail',
            'ip_restrictions': 'Optional for high-security clients'
        },
        
        'legal_compliance': {
            'gdpr_compliance': 'Full data subject rights',
            'data_portability': 'Machine-readable export formats',
            'correction_requests': 'Employee-initiated data correction workflow',
            'deletion_requests': 'Right to be forgotten (with legal limitations)'
        }
    }
```

## Database Architecture for Legal Compliance

### 1. Active Employee Data

```sql
-- Primary table for active employees
CREATE TABLE ewidencja_czasu_aktywni (
    id BIGSERIAL PRIMARY KEY,
    pracownik_id INTEGER NOT NULL REFERENCES pracownicy(id),
    data_pracy DATE NOT NULL,
    
    -- Time tracking (exact requirements)
    czas_rozpoczecia TIMESTAMP,
    czas_zakonczenia TIMESTAMP,
    godziny_przepracowane DECIMAL(4,2),
    
    -- Detailed overtime breakdown
    nadgodziny_zwykle DECIMAL(4,2),      -- Regular overtime (50% supplement)
    nadgodziny_swieta DECIMAL(4,2),      -- Holiday overtime (100% supplement)
    nadgodziny_niedziela DECIMAL(4,2),   -- Sunday overtime (100% supplement)
    
    -- Night work details
    praca_nocna_godziny DECIMAL(4,2),
    praca_nocna_okres VARCHAR(20),        -- e.g., "22:00-06:00"
    
    -- On-call work (LOCATION REQUIRED)
    dyzur_godziny DECIMAL(4,2),
    dyzur_lokalizacja VARCHAR(200) CHECK (
        (dyzur_godziny IS NULL) OR 
        (dyzur_godziny IS NOT NULL AND dyzur_lokalizacja IS NOT NULL)
    ),
    dyzur_typ VARCHAR(50),
    
    -- Absence tracking
    nieobecnosc_typ VARCHAR(50),
    nieobecnosc_godziny DECIMAL(4,2),
    nieobecnosc_usprawiedliwiona BOOLEAN,
    nieobecnosc_powod VARCHAR(200),
    
    -- Minor work (when applicable)
    praca_nieletni_zabronione DECIMAL(4,2),
    
    -- Audit trail
    utworzono TIMESTAMP DEFAULT NOW(),
    utworzono_przez INTEGER NOT NULL REFERENCES users(id),
    zmodyfikowano TIMESTAMP,
    zmodyfikowano_przez INTEGER REFERENCES users(id),
    powod_zmiany TEXT,
    
    -- Data integrity
    hash_integralnosci VARCHAR(64) NOT NULL,
    wersja_rekordu INTEGER DEFAULT 1,
    
    -- Indexes for performance and compliance
    INDEX idx_pracownik_data (pracownik_id, data_pracy),
    INDEX idx_pip_eksport (data_pracy, utworzono),
    INDEX idx_audit_trail (zmodyfikowano, zmodyfikowano_przez)
);
```

### 2. Archived Employee Data

```sql
-- Separate table for terminated employees (10-year retention)
CREATE TABLE ewidencja_czasu_archiwum (
    -- Inherit all columns from active table
    LIKE ewidencja_czasu_aktywni INCLUDING ALL,
    
    -- Additional archival metadata
    data_archiwizacji TIMESTAMP DEFAULT NOW(),
    data_wygasniecia_retencji TIMESTAMP NOT NULL,
    powod_archiwizacji VARCHAR(100) DEFAULT 'zakonczenie_zatrudnienia',
    
    -- PIP inspection readiness
    pip_gotowy BOOLEAN DEFAULT TRUE,
    ostatni_pip_eksport TIMESTAMP,
    liczba_pip_eksportow INTEGER DEFAULT 0,
    
    -- Legal compliance verification
    weryfikacja_prawna_timestamp TIMESTAMP,
    weryfikacja_prawna_przez INTEGER REFERENCES users(id),
    
    -- Secure deletion tracking
    zaplanowane_usuniecie TIMESTAMP,
    usunieto_timestamp TIMESTAMP,
    certyfikat_usuniecia VARCHAR(200),
    
    INDEX idx_retencja (data_wygasniecia_retencji),
    INDEX idx_pip_archiwum (pip_gotowy, data_archiwizacji)
);
```

### 3. Data Correction Audit Trail

```sql
-- Immutable audit trail for all data corrections
CREATE TABLE ewidencja_audit_trail (
    id BIGSERIAL PRIMARY KEY,
    pracownik_id INTEGER NOT NULL,
    rekord_id BIGINT NOT NULL, -- Reference to original record
    
    -- Change details
    typ_zmiany VARCHAR(50) NOT NULL, -- 'utworzenie', 'modyfikacja', 'usuniecie'
    pole_zmienione VARCHAR(100),
    wartosc_stara TEXT,
    wartosc_nowa TEXT,
    
    -- Authorization and reasoning
    uzytkownik_id INTEGER NOT NULL REFERENCES users(id),
    powod_zmiany TEXT NOT NULL,
    podstawa_prawna VARCHAR(200),
    
    -- Audit metadata
    timestamp_zmiany TIMESTAMP DEFAULT NOW(),
    ip_address INET,
    session_id VARCHAR(100),
    
    -- Legal compliance
    pip_znaczenie VARCHAR(200), -- Impact on PIP compliance
    hash_niezmiennosci VARCHAR(64), -- Immutability hash
    
    -- IMMUTABLE table - no updates or deletes allowed
    INDEX idx_pracownik_audit (pracownik_id, timestamp_zmiany),
    INDEX idx_rekord_audit (rekord_id, timestamp_zmiany)
);

-- Prevent any modifications to audit trail
CREATE OR REPLACE FUNCTION prevent_audit_modifications()
RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'Audit trail records are immutable for legal compliance';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER prevent_audit_updates
    BEFORE UPDATE OR DELETE ON ewidencja_audit_trail
    FOR EACH ROW EXECUTE FUNCTION prevent_audit_modifications();
```

## Automated Retention Management

### 1. Retention Lifecycle Automation

```python
def automated_retention_lifecycle():
    """
    Automated system for managing data retention lifecycle
    """
    
    class RetentionManager:
        def daily_retention_tasks(self):
            """Run daily to maintain compliance"""
            
            # Check for employees terminated in previous calendar year
            terminated_last_year = get_employees_terminated_previous_calendar_year()
            for employee in terminated_last_year:
                self.archive_employee_data(employee)
            
            # Check for data approaching retention expiry
            expiring_soon = get_data_expiring_within_months(6)
            for record in expiring_soon:
                self.notify_retention_expiry(record)
            
            # Perform secure deletion of expired data
            expired_data = get_data_past_retention_period()
            for record in expired_data:
                self.secure_delete_with_certification(record)
        
        def archive_employee_data(self, employee: Employee):
            """Move terminated employee data to archival system"""
            
            # Calculate exact retention period
            retention_period = calculate_retention_period(employee)
            
            # Verify data completeness before archival
            completeness_check = verify_data_completeness(employee.id)
            if not completeness_check['complete']:
                raise ComplianceError(f"Cannot archive incomplete data: {completeness_check['missing']}")
            
            # Move to archival table with metadata
            archive_record = {
                'source_data': get_complete_employee_data(employee.id),
                'archival_metadata': {
                    'archived_date': datetime.utcnow(),
                    'retention_expires': retention_period['retention_expires'],
                    'legal_basis': 'Art. 149 § 1 KP',
                    'pip_ready': True
                }
            }
            
            # Atomic operation: insert into archive and remove from active
            with database.transaction():
                insert_into_archive(archive_record)
                verify_archive_integrity(employee.id)
                remove_from_active_with_audit_trail(employee.id)
            
            # Generate archival certificate
            generate_archival_certificate(employee.id, retention_period)
        
        def secure_delete_with_certification(self, record):
            """Secure deletion after retention period expires"""
            
            # Verify retention period has actually expired
            if record.retention_expires > datetime.utcnow():
                raise ComplianceError("Cannot delete data before retention period expires")
            
            # Generate deletion certificate BEFORE deletion
            deletion_certificate = generate_deletion_certificate(record)
            
            # Secure deletion with multiple overwrite passes
            secure_delete_record(record.id)
            
            # Store deletion certificate for audit purposes
            store_deletion_certificate(deletion_certificate)
            
            # Notify stakeholders of completed deletion
            notify_legal_deletion_completed(record, deletion_certificate)
```

### 2. PIP Inspection Emergency Response

```python
def pip_inspection_emergency_response():
    """
    Rapid response system for unannounced PIP inspections
    New 2025 powers: 24/7 unannounced inspections
    """
    
    return {
        'immediate_response': {
            'max_response_time': 5,  # minutes to produce any requested data
            'complete_export_time': 15,  # minutes for full employee history
            'inspector_access_portal': 'Secure temporary access with full audit logging'
        },
        
        'inspection_ready_data': {
            'real_time_compliance_status': 'Current violations and warnings',
            'employee_data_completeness': 'Verification all required fields present',
            'calculation_verification': 'Mathematical proof of all overtime/leave calculations',
            'legal_basis_documentation': 'Citation for every system rule and calculation'
        },
        
        'inspector_support_tools': {
            'data_export_formats': 'PDF, Excel, CSV as requested by inspector',
            'historical_comparison': 'Year-over-year compliance trends',
            'violation_remediation': 'Evidence of corrective actions taken',
            'employee_interview_preparation': 'Data summaries for employee verification'
        },
        
        'legal_protection': {
            'data_integrity_certification': 'Cryptographic proof of data authenticity',
            'audit_trail_completeness': 'Complete chain of custody for all data',
            'legal_expert_contact': 'Immediate access to Polish labor law counsel',
            'compliance_documentation': 'Proof of good faith compliance efforts'
        }
    }
```

## Implementation Checklist

### Critical Pre-Deployment Requirements
- [ ] **10-year retention system** tested and verified
- [ ] **PIP export capability** validated by legal expert
- [ ] **Employee access portal** implemented and tested
- [ ] **Data integrity** verification system deployed
- [ ] **Automatic archival** process tested with terminated employees

### Legal Validation Requirements
- [ ] **Polish labor law expert** review of retention system
- [ ] **PIP consultation** on export format requirements
- [ ] **Data protection authority** guidance on GDPR compliance
- [ ] **Legal counsel** approval of deletion procedures

### Ongoing Compliance Monitoring
- [ ] **Daily retention audits** automated
- [ ] **Monthly PIP readiness testing** scheduled
- [ ] **Quarterly legal compliance review** with expert
- [ ] **Annual data integrity certification** process

**Proper data retention implementation required for legal compliance.**