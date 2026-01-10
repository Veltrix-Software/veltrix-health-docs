# Medical Management System - System Design Document

## Table of Contents
1. [System Overview](#1-system-overview)
2. [Stakeholders & User Types](#2-stakeholders--user-types)
3. [Functional Requirements](#3-functional-requirements)
4. [Non-Functional Requirements](#4-non-functional-requirements)
5. [User Scenarios](#5-user-scenarios)
6. [High-Level Design (HLD)](#6-high-level-design-hld)
7. [Entity Relationship Diagrams (ERD)](#7-entity-relationship-diagrams-erd)
8. [Sequence Diagrams](#8-sequence-diagrams)
9. [Flow Charts](#9-flow-charts)
10. [Technical Implementation Details](#10-technical-implementation-details)

---

## 1. System Overview

### 1.1 Purpose
The Medical Management System is a comprehensive healthcare platform designed to serve three distinct operational models with two deployment options:
- **Personal Clinic**: Single doctor practice
- **Multi-Doctor Clinic**: Clinic with multiple doctors
- **Hospital**: Large facility with multiple departments and doctors

### 1.2 Deployment Models
The system supports two deployment strategies:
1. **SaaS Multi-Tenant Model**: Centralized hosting with subscription-based access managed by Super Admin
2. **Private Deployment Model**: Self-hosted installation where clients receive source code for deployment on their private servers

### 1.3 Version Roadmap

#### Version 1.0 (Current Scope)
- Core patient management functionality
- Appointment scheduling (without SMS/Email notifications)
- Electronic Medical Records (EMR)
- Prescription management
- Basic billing and invoicing (cash/manual payments only)
- Multi-tenant architecture with manual entry
- Role-based access control
- Cost-optimized deployment (no external service integrations)

#### Version 2.0 (Future Scope)
- Mobile application (iOS & Android)
- Automated SMS notifications
- Email notifications and reminders
- Automated payment gateway integration
- Advanced reporting and analytics
- Lab integration
- Pharmacy integration
- Insurance claim management
- Telemedicine capabilities

### 1.4 Key Features (Version 1.0)
- Patient registration and management
- Manual appointment scheduling
- Electronic Medical Records (EMR)
- Prescription management
- Manual billing and invoicing
- Basic reporting
- Multi-tenant architecture
- Role-based access control
- Manual data entry for all operations

### 1.5 Architectural Approach
- **Architecture Pattern**: Clean Architecture (Onion Architecture)
- **Deployment Style**: Monolithic application
- **Data Mapping**: Manual mapping (no AutoMapper)
- **Validation**: FluentValidation
- **Database**: Single database with tenant isolation via OrganizationId
- **Caching**: Redis for performance optimization

---

## 2. Stakeholders & User Types

### 2.1 Primary Stakeholders

#### Healthcare Providers
- Doctors providing medical consultations
- Nurses assisting with patient care
- Medical staff managing day-to-day operations

#### Patients
- Individuals seeking medical care
- Can be registered by receptionist or self-register (if enabled)

#### Administrative Staff
- Receptionists managing appointments and patient registration
- Billing personnel handling invoices and payments
- Organization administrators managing clinic/hospital settings

#### System Stakeholders
- **Super Admin**: Platform owner managing multi-tenant SaaS deployments
- **IT Staff**: Technical teams managing private deployments
- **Clients**: Healthcare organizations subscribing to or purchasing the system

### 2.2 User Roles Hierarchy

```mermaid
graph TD
    A[Users] --> B[Super Admin]
    A --> C[Organization Admin]
    A --> D[Doctor]
    A --> E[Nurse]
    A --> F[Receptionist]
    A --> G[Billing Staff]
    A --> H[Patient]
    
    B --> I[Multi-Tenant Management]
    B --> J[Subscription Management]
    B --> K[System Configuration]
    
    C --> L[Organization Settings]
    C --> M[User Management]
    C --> N[Department Management]
    C --> O[Service Configuration]
    
    D --> P[Patient Consultation]
    D --> Q[Medical Records]
    D --> R[Prescriptions]
    
    E --> S[Patient Vitals]
    E --> T[Assist Doctor]
    
    F --> U[Patient Registration]
    F --> V[Appointment Scheduling]
    
    G --> W[Invoice Generation]
    G --> X[Payment Processing]
    
    H --> Y[View Records]
    H --> Z[Appointment Requests]
```

### 2.3 Role Permissions Matrix

| Role | Patient Management | Appointments | Medical Records | Prescriptions | Billing | User Management | System Config | Organization Config |
|------|-------------------|--------------|-----------------|---------------|---------|-----------------|---------------|---------------------|
| **Super Admin** | ✓ (All Orgs) | ✓ (All Orgs) | ✓ (All Orgs) | ✓ (All Orgs) | ✓ (All Orgs) | ✓ (All Orgs) | ✓ | ✓ (All Orgs) |
| **Organization Admin** | ✓ (Own Org) | ✓ (Own Org) | ✓ (View Only) | ✓ (View Only) | ✓ (Own Org) | ✓ (Own Org) | ✗ | ✓ (Own Org) |
| **Doctor** | ✓ (Read) | ✓ (View/Update) | ✓ (Create/Edit) | ✓ (Create/Edit) | ✗ | ✗ | ✗ | ✗ |
| **Nurse** | ✓ (Read) | ✓ (View) | ✓ (Vitals Only) | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Receptionist** | ✓ (Create/Edit) | ✓ (Create/Edit) | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Billing Staff** | ✓ (Read) | ✓ (Read) | ✓ (Read) | ✓ (Read) | ✓ (All) | ✗ | ✗ | ✗ |
| **Patient** | ✓ (Own Only) | ✓ (View Own) | ✓ (View Own) | ✓ (View Own) | ✓ (View Own) | ✗ | ✗ | ✗ |

### 2.4 Deployment-Specific Roles

#### SaaS Multi-Tenant Deployment
- **Super Admin**: Manages all organizations, subscriptions, and system-wide settings
- **Organization Admin**: Manages their specific organization
- **Regular Users**: Work within their assigned organization

#### Private Deployment
- **No Super Admin**: Organization Admin has full system control
- **Organization Admin**: Acts as system administrator
- **Regular Users**: Same as multi-tenant model

---

## 3. Functional Requirements

### 3.1 Deployment & Multi-Tenancy Management

#### FR-DEPLOY-001: Dual Deployment Support
**Description**: System shall support both SaaS multi-tenant and private deployment models  
**Priority**: Critical  
**Details**:
- Single codebase supporting both deployment types
- Configuration-based switching between deployment modes
- Tenant isolation maintained in both models

#### FR-DEPLOY-002: SaaS Subscription Management
**Description**: Super Admin shall manually manage organization subscriptions  
**Priority**: High (SaaS only)  
**Details**:
- Manual creation of organization accounts
- Manual subscription activation/deactivation
- Manual user limit configuration per organization
- No automated payment processing in Version 1.0

#### FR-DEPLOY-003: Private Deployment Configuration
**Description**: System shall support standalone private deployment  
**Priority**: Critical  
**Details**:
- Single organization mode
- No Super Admin role in private deployment
- Organization Admin has system-level access
- No subscription management features visible

### 3.2 Organization Management

#### FR-ORG-001: Organization Type Support
**Description**: System shall support three organization types  
**Priority**: Critical  
**Details**:
- Personal Clinic: Single doctor, no departments
- Multi-Doctor Clinic: Multiple doctors, optional departments
- Hospital: Multiple departments, multiple doctors per department

#### FR-ORG-002: Organization Configuration
**Description**: Organization Admin shall configure organization settings  
**Priority**: High  
**Details**:
- Working hours and days
- Service offerings and pricing
- Branding (logo, colors, contact information)
- Appointment duration defaults

#### FR-ORG-003: Department Management
**Description**: Hospital-type organizations shall manage departments  
**Priority**: Medium  
**Details**:
- Create/edit/delete departments
- Assign doctors to departments
- Configure department-specific services
- Department hierarchy support

#### FR-ORG-004: Multi-Organization Doctor Assignment
**Description**: Doctors shall work across multiple organizations  
**Priority**: Medium  
**Details**:
- Doctor can be assigned to multiple organizations
- Separate schedules per organization
- Organization-specific permissions

### 3.3 User Management

#### FR-USER-001: Role-Based Access Control
**Description**: System shall implement comprehensive role-based permissions  
**Priority**: Critical  
**Details**:
- Seven distinct roles (Super Admin, Org Admin, Doctor, Nurse, Receptionist, Billing Staff, Patient)
- Permission matrix enforcement
- Role-based UI rendering

#### FR-USER-002: User Account Management
**Description**: Organization Admin shall manage user accounts  
**Priority**: High  
**Details**:
- Create/edit/deactivate user accounts
- Assign roles and permissions
- Reset passwords
- Set user-organization associations

#### FR-USER-003: User Authentication
**Description**: System shall provide secure authentication  
**Priority**: Critical  
**Details**:
- Email and password-based login
- JWT token-based session management
- Password strength requirements
- Account lockout after failed attempts
- Optional two-factor authentication (Version 2.0)

#### FR-USER-004: User Profile Management
**Description**: Users shall manage their profile information  
**Priority**: Medium  
**Details**:
- Update personal information
- Change password
- Set preferences
- Upload profile picture

### 3.4 Patient Management

#### FR-PAT-001: Patient Registration
**Description**: Receptionists shall register new patients  
**Priority**: Critical  
**Details**:
- Capture complete demographics (name, DOB, gender, contact)
- Assign unique patient ID
- Record emergency contact information
- Capture blood group and allergies
- Optional patient photo

#### FR-PAT-002: Patient Search and Retrieval
**Description**: System shall provide efficient patient search  
**Priority**: High  
**Details**:
- Search by name, phone, email, patient ID
- Filter by registration date, age, gender
- Quick search with autocomplete
- Recent patients list

#### FR-PAT-003: Patient History Management
**Description**: System shall maintain comprehensive patient history  
**Priority**: Critical  
**Details**:
- Complete visit history
- Chronological medical records
- Prescription history
- Lab test history (manual entry)
- Billing history

#### FR-PAT-004: Patient Demographics Update
**Description**: Authorized users shall update patient information  
**Priority**: Medium  
**Details**:
- Edit contact information
- Update address and emergency contacts
- Modify medical history
- Audit trail of changes

#### FR-PAT-005: Patient Document Management
**Description**: System shall store patient-related documents  
**Priority**: Medium  
**Details**:
- Upload medical reports
- Store lab results (PDF/Image)
- Attach prescriptions
- Archive consent forms

### 3.5 Appointment Management

#### FR-APT-001: Manual Appointment Scheduling
**Description**: Receptionists shall manually schedule appointments  
**Priority**: Critical  
**Details**:
- Select patient (existing or register new)
- Choose doctor and date/time
- Set appointment duration
- Add appointment notes
- Confirm booking (no automated notifications in V1.0)

#### FR-APT-002: Doctor Schedule Management
**Description**: System shall manage doctor availability  
**Priority**: High  
**Details**:
- Define working hours per doctor
- Set availability by day of week
- Block time slots for breaks/meetings
- Mark unavailable dates (leaves/holidays)

#### FR-APT-003: Appointment Calendar View
**Description**: Users shall view appointments in calendar format  
**Priority**: High  
**Details**:
- Daily, weekly, monthly views
- Doctor-specific calendars
- Color-coded appointment status
- Quick appointment details on hover

#### FR-APT-004: Appointment Status Management
**Description**: System shall track appointment lifecycle  
**Priority**: Medium  
**Details**:
- Status: Scheduled, Confirmed, In Progress, Completed, Cancelled, No-Show
- Manual status updates
- Cancellation with reason
- Rescheduling capability

#### FR-APT-005: Walk-in Patient Handling
**Description**: System shall support walk-in appointments  
**Priority**: Medium  
**Details**:
- Quick appointment registration
- Add to waiting queue
- Prioritization options
- Immediate consultation flow

### 3.6 Medical Records Management

#### FR-MED-001: Consultation Record Creation
**Description**: Doctors shall create medical records per visit  
**Priority**: Critical  
**Details**:
- Chief complaint entry
- Symptoms documentation
- Physical examination findings
- Diagnosis with ICD-10 code support
- Treatment plan
- Doctor notes

#### FR-MED-002: Vital Signs Recording
**Description**: Nurses shall record patient vital signs  
**Priority**: High  
**Details**:
- Blood pressure, pulse, temperature
- Height, weight, BMI calculation
- Respiratory rate, oxygen saturation
- Timestamp and recorded by information

#### FR-MED-003: Medical History Access
**Description**: Doctors shall access complete patient medical history  
**Priority**: Critical  
**Details**:
- Chronological visit records
- Previous diagnoses
- Ongoing treatments
- Allergy information
- Family medical history

#### FR-MED-004: Medical Record Search
**Description**: System shall enable searching medical records  
**Priority**: Medium  
**Details**:
- Search by diagnosis
- Filter by date range
- Search by doctor
- Filter by ICD-10 codes

#### FR-MED-005: Medical Record Editing
**Description**: Doctors shall edit their own medical records  
**Priority**: Medium  
**Details**:
- Edit within 24 hours of creation
- Add addendum to older records
- Full audit trail of changes
- Cannot delete records

### 3.7 Prescription Management

#### FR-PRESC-001: Electronic Prescription Creation
**Description**: Doctors shall create electronic prescriptions  
**Priority**: Critical  
**Details**:
- Add multiple medications
- Specify dosage and frequency
- Set duration of treatment
- Add special instructions
- Link to medical record

#### FR-PRESC-002: Medication Database
**Description**: System shall maintain medication master list  
**Priority**: High  
**Details**:
- Searchable medication database
- Generic and brand names
- Standard dosages
- Drug interaction warnings (basic)
- Frequently prescribed medications

#### FR-PRESC-003: Prescription Templates
**Description**: Doctors shall create and use prescription templates  
**Priority**: Medium  
**Details**:
- Save common prescriptions
- Quick template selection
- Modify template before finalizing
- Personal template library

#### FR-PRESC-004: Prescription Printing
**Description**: System shall generate printable prescriptions  
**Priority**: High  
**Details**:
- Professional prescription format
- Doctor and clinic details
- Patient information
- QR code for verification (optional)
- Prescription number

#### FR-PRESC-005: Prescription History
**Description**: System shall maintain prescription history  
**Priority**: Medium  
**Details**:
- View all patient prescriptions
- Reprint previous prescriptions
- Track refills manually
- Search by medication name

### 3.8 Laboratory Test Management (Manual Entry)

#### FR-LAB-001: Lab Test Ordering
**Description**: Doctors shall order laboratory tests  
**Priority**: Medium  
**Details**:
- Select from test catalog
- Add clinical notes
- Mark as urgent if needed
- Generate lab request form

#### FR-LAB-002: Lab Test Catalog
**Description**: System shall maintain lab test master list  
**Priority**: Medium  
**Details**:
- Searchable test catalog
- Test descriptions
- Sample types
- Normal ranges

#### FR-LAB-003: Manual Lab Results Entry
**Description**: Staff shall manually enter lab results  
**Priority**: Medium  
**Details**:
- Enter test results manually
- Attach PDF/image reports
- Mark abnormal values
- Add lab technician notes

#### FR-LAB-004: Lab Results Viewing
**Description**: Doctors and patients shall view lab results  
**Priority**: Medium  
**Details**:
- Display results with normal ranges
- Highlight abnormal values
- View historical trends
- Compare with previous results

### 3.9 Billing & Invoicing (Manual Payment Only)

#### FR-BILL-001: Manual Invoice Generation
**Description**: Billing staff shall manually create invoices  
**Priority**: Critical  
**Details**:
- Link to medical record/appointment
- Add consultation charges
- Add procedure charges
- Add lab test charges (manual)
- Add medication charges (if applicable)
- Calculate total amount

#### FR-BILL-002: Service Pricing Management
**Description**: Organization Admin shall configure service pricing  
**Priority**: High  
**Details**:
- Set consultation fees by doctor
- Configure procedure pricing
- Set lab test prices
- Define medication pricing (optional)

#### FR-BILL-003: Manual Payment Recording
**Description**: Billing staff shall record payments manually  
**Priority**: Critical  
**Details**:
- Record cash payments
- Record card payments (manual entry)
- Record check payments
- Enter payment reference number
- Generate payment receipt

#### FR-BILL-004: Partial Payment Support
**Description**: System shall handle partial payments  
**Priority**: Medium  
**Details**:
- Record partial amount
- Track outstanding balance
- Multiple payment entries per invoice
- Payment history per invoice

#### FR-BILL-005: Invoice Management
**Description**: System shall manage invoice lifecycle  
**Priority**: High  
**Details**:
- Status: Draft, Issued, Partially Paid, Paid, Cancelled
- Invoice search and filtering
- Reprint invoices
- Credit note generation
- Invoice voiding with reason

#### FR-BILL-006: Receipt Generation
**Description**: System shall generate payment receipts  
**Priority**: High  
**Details**:
- Professional receipt format
- Organization details
- Payment details
- Outstanding balance (if any)
- Receipt number

### 3.10 Reporting & Analytics

#### FR-REP-001: Patient Reports
**Description**: System shall generate patient-related reports  
**Priority**: Medium  
**Details**:
- Patient registration report
- Patient visit summary
- Demographics analysis
- New vs returning patients

#### FR-REP-002: Appointment Reports
**Description**: System shall generate appointment reports  
**Priority**: Medium  
**Details**:
- Daily appointment schedule
- Doctor utilization report
- Appointment status summary
- No-show analysis

#### FR-REP-003: Financial Reports
**Description**: System shall generate financial reports  
**Priority**: High  
**Details**:
- Daily collection report
- Revenue by service type
- Revenue by doctor
- Outstanding payments report
- Monthly financial summary

#### FR-REP-004: Doctor Performance Reports
**Description**: System shall track doctor metrics  
**Priority**: Low  
**Details**:
- Consultation count
- Revenue generated
- Average consultation time
- Patient satisfaction (if implemented)

#### FR-REP-005: Report Export
**Description**: Users shall export reports  
**Priority**: Medium  
**Details**:
- Export to PDF
- Export to Excel
- Print functionality
- Email report (Version 2.0)

---

## 4. Non-Functional Requirements

### 4.1 Performance Requirements

#### NFR-PERF-001: Response Time
**Description**: System shall respond quickly to user actions  
**Target**:
- Page load time: < 2 seconds
- API response time: < 500ms for 95% of requests
- Search operations: < 1 second
- Report generation: < 5 seconds for standard reports

#### NFR-PERF-002: Concurrent Users
**Description**: System shall support multiple simultaneous users  
**Target**:
- Support 100+ concurrent users per organization
- Support 1000+ concurrent users across all tenants (SaaS)
- No performance degradation up to target load

#### NFR-PERF-003: Database Performance
**Description**: Database operations shall be optimized  
**Target**:
- Query execution: < 200ms for standard queries
- Indexed searches: < 100ms
- Transaction commit: < 100ms

#### NFR-PERF-004: Caching Strategy
**Description**: System shall implement efficient caching  
**Target**:
- Redis cache for frequently accessed data
- Cache hit ratio: > 80% for static data
- Cache invalidation strategy in place

### 4.2 Security Requirements

#### NFR-SEC-001: Data Encryption
**Description**: All sensitive data shall be encrypted  
**Implementation**:
- Data at rest: Database encryption
- Data in transit: TLS 1.3
- Password storage: Bcrypt/PBKDF2 hashing
- Sensitive fields: Application-level encryption

#### NFR-SEC-002: Authentication & Authorization
**Description**: Secure access control mechanisms  
**Implementation**:
- JWT-based authentication
- Token expiration: 1 hour
- Refresh token support
- Role-based authorization
- Organization-based data isolation

#### NFR-SEC-003: HIPAA Compliance (Where Applicable)
**Description**: System shall follow healthcare data protection standards  
**Implementation**:
- Audit logging of all data access
- Patient consent management
- Data access restrictions
- Secure data disposal procedures

#### NFR-SEC-004: Multi-Tenant Data Isolation
**Description**: Complete data segregation between organizations  
**Implementation**:
- Organization ID in all database queries
- Query filters at database context level
- No cross-tenant data leakage
- Separate data backups per tenant (optional)

#### NFR-SEC-005: Security Audit Trail
**Description**: Complete audit logging for security events  
**Implementation**:
- Log all authentication attempts
- Log all data modifications
- Log all access to sensitive data
- User action tracking with timestamps
- IP address and device logging

### 4.3 Scalability Requirements

#### NFR-SCALE-001: Horizontal Scalability
**Description**: System architecture shall support horizontal scaling  
**Implementation**:
- Stateless application design
- Load balancing support
- Distributed caching with Redis
- Database connection pooling

#### NFR-SCALE-002: Database Scalability
**Description**: Database shall scale with growing data  
**Implementation**:
- Indexed columns for fast queries
- Partitioning strategy for large tables
- Archive old data strategy
- Read replicas for reporting (optional)

#### NFR-SCALE-003: Organization Limit
**Description**: System shall support multiple organizations  
**Target**:
- Version 1.0: Support 100+ organizations (SaaS)
- Version 1.0: Unlimited patients per organization
- Version 1.0: 1000+ appointments per day per organization

#### NFR-SCALE-004: Data Growth Management
**Description**: System shall handle data growth efficiently  
**Implementation**:
- Pagination for all list views
- Lazy loading for large datasets
- Data archival policies
- Storage optimization strategies

### 4.4 Availability & Reliability

#### NFR-AVAIL-001: System Uptime
**Description**: System shall maintain high availability  
**Target**:
- Uptime: 99.5% (SaaS deployment)
- Planned downtime: < 4 hours per month
- Off-peak maintenance windows

#### NFR-AVAIL-002: Backup & Recovery
**Description**: Regular data backups and recovery procedures  
**Implementation**:
- Automated daily backups
- Point-in-time recovery capability
- Backup retention: 30 days
- Backup verification procedures
- Recovery Time Objective (RTO): < 4 hours
- Recovery Point Objective (RPO): < 24 hours

#### NFR-AVAIL-003: Error Handling
**Description**: Graceful error handling and recovery  
**Implementation**:
- User-friendly error messages
- Detailed error logging
- Automatic error reporting
- Transaction rollback on failures

#### NFR-AVAIL-004: Data Integrity
**Description**: Maintain data consistency and integrity  
**Implementation**:
- Database constraints and foreign keys
- Transaction management
- Validation at all layers
- Referential integrity enforcement

### 4.5 Usability Requirements

#### NFR-USE-001: User Interface Design
**Description**: Intuitive and user-friendly interface  
**Implementation**:
- Responsive design (desktop, tablet)
- Consistent UI patterns
- Material Design principles (Angular Material)
- Maximum 3 clicks to common functions

#### NFR-USE-002: Accessibility
**Description**: System shall be accessible to users with disabilities  
**Implementation**:
- WCAG 2.1 Level AA compliance (where possible)
- Keyboard navigation support
- Screen reader compatibility
- High contrast mode support
- Minimum font size: 14px

#### NFR-USE-003: Multi-Language Support
**Description**: Support multiple languages (Version 2.0)  
**Implementation**:
- i18n framework integration
- RTL language support
- Language selection per user
- Default: English

#### NFR-USE-004: Help & Documentation
**Description**: Comprehensive user guidance  
**Implementation**:
- In-app help tooltips
- User manual (PDF)
- Video tutorials (Version 2.0)
- Context-sensitive help

### 4.6 Maintainability Requirements

#### NFR-MAINT-001: Code Quality
**Description**: High-quality, maintainable codebase  
**Implementation**:
- Clean Architecture principles
- SOLID principles
- Comprehensive comments
- Consistent naming conventions
- Code review process

#### NFR-MAINT-002: Testing Requirements
**Description**: Comprehensive test coverage  
**Implementation**:
- Unit tests: > 70% coverage
- Integration tests for critical paths
- Manual testing checklist
- Regression testing procedures

#### NFR-MAINT-003: Logging & Monitoring
**Description**: Comprehensive system monitoring  
**Implementation**:
- Structured logging (Serilog)
- Application performance monitoring
- Error tracking and alerting
- Usage analytics
- Log retention: 90 days

#### NFR-MAINT-004: Configuration Management
**Description**: Flexible configuration without code changes  
**Implementation**:
- Environment-based configuration
- Feature flags for gradual rollouts
- Runtime configuration updates
- Separate dev/staging/production configs

### 4.7 Portability Requirements

#### NFR-PORT-001: Cross-Platform Support
**Description**: System shall run on multiple platforms  
**Implementation**:
- Backend: Windows Server, Linux
- Database: SQL Server, PostgreSQL
- Web browsers: Chrome, Firefox, Safari, Edge (latest 2 versions)
- No browser plugins required

#### NFR-PORT-002: Deployment Flexibility
**Description**: Easy deployment to different environments  
**Implementation**:
- Docker containerization support (optional)
- Cloud deployment ready (Azure, AWS)
- On-premise deployment support
- Minimal infrastructure requirements

### 4.8 Compliance Requirements

#### NFR-COMP-001: Healthcare Regulations
**Description**: Comply with healthcare industry standards  
**Implementation**:
- HIPAA compliance (US) - where applicable
- GDPR compliance (EU) - where applicable
- Local healthcare regulations
- Patient consent management
- Data privacy policies

#### NFR-COMP-002: Data Protection
**Description**: Protect patient and organizational data  
**Implementation**:
- Data minimization principles
- Right to be forgotten (soft delete)
- Data export capability
- Privacy policy enforcement
- Terms of service acceptance

### 4.9 Cost Optimization (Version 1.0)

#### NFR-COST-001: Minimal External Dependencies
**Description**: Reduce operational costs by avoiding paid services  
**Implementation**:
- No SMS gateway integration (V1.0)
- No email service integration (V1.0)
- No payment gateway integration (V1.0)
- No third-party APIs
- Self-hosted deployment options

#### NFR-COST-002: Resource Efficiency
**Description**: Optimize infrastructure costs  
**Implementation**:
- Efficient database queries
- Proper indexing strategy
- Redis caching to reduce DB load
- Minimal server requirements
- Single-server deployment capable

---

## 5. User Scenarios

### 5.1 Super Admin Scenarios (SaaS Deployment Only)

#### Scenario 5.1.1: Creating New Organization Account

**Actor**: Super Admin  
**Precondition**: Super Admin is logged into the system  
**Goal**: Manually create a new organization subscription

**Main Flow**:
1. Super Admin navigates to Organizations management section
2. Clicks "Add New Organization" button
3. Fills in organization details:
   - Organization name
   - Organization type (Personal Clinic / Multi-Doctor Clinic / Hospital)
   - Contact information (address, phone, email)
   - Primary contact person details
4. Sets organization configuration:
   - User limit (number of staff accounts)
   - Storage limit
   - Features enabled
   - Subscription start date
   - Subscription end date (if applicable)
5. Creates initial Organization Admin account:
   - Admin full name
   - Admin email
   - Temporary password
6. Reviews and confirms organization creation
7. System creates organization and admin account
8. Super Admin manually notifies admin via phone/external email with login credentials

**Postcondition**: New organization is active and Organization Admin can log in

**Alternative Flow**:
- If organization name already exists, show error message
- If email already in use, show error message and suggest using different email

---

#### Scenario 5.1.2: Managing Organization Subscription

**Actor**: Super Admin  
**Precondition**: Organization exists in the system  
**Goal**: Modify organization subscription or settings

**Main Flow**:
1. Super Admin searches for organization by name or ID
2. Selects organization to view details
3. Views current subscription status and usage:
   - Active users count vs. limit
   - Storage used vs. limit
   - Subscription expiry date
   - Last activity date
4. Performs desired action:
   - Extend subscription period
   - Increase/decrease user limit
   - Suspend organization (temporarily disable access)
   - Reactivate suspended organization
   - Terminate organization (permanent)
5. Adds notes explaining the change
6. Confirms action
7. System updates organization status
8. Super Admin manually notifies Organization Admin if needed

**Postcondition**: Organization subscription is updated according to changes

---

### 5.2 Organization Admin Scenarios

#### Scenario 5.2.1: Setting Up Organization Profile

**Actor**: Organization Admin  
**Precondition**: Organization Admin has logged in for the first time  
**Goal**: Complete organization setup and configuration

**Main Flow**:
1. Organization Admin logs in with credentials provided by Super Admin (SaaS) or initial setup (Private)
2. System prompts for organization profile completion
3. Organization Admin enters/updates:
   - Complete organization details
   - Logo upload
   - Working hours (Monday-Sunday)
   - Consultation duration defaults (15, 30, 45, 60 minutes)
   - Currency and timezone
4. For Hospital type, creates departments:
   - Department name
   - Description
   - Department head (optional at this stage)
5. Configures services offered:
   - Consultation types
   - Procedure types
   - Pricing for each service
6. Sets up appointment rules:
   - Advance booking limit (e.g., 30 days)
   - Same-day appointment policy
   - Cancellation policy
7. Reviews and saves configuration
8. System confirms setup completion

**Postcondition**: Organization is fully configured and ready to add staff and patients

---

#### Scenario 5.2.2: Creating User Accounts

**Actor**: Organization Admin  
**Precondition**: Organization profile is set up  
**Goal**: Add staff members to the system

**Main Flow**:
1. Organization Admin navigates to User Management section
2. Clicks "Add New User" button
3. Selects user role (Doctor / Nurse / Receptionist / Billing Staff)
4. Enters user details:
   - First name and last name
   - Email address (will be used for login)
   - Phone number
   - Temporary password
5. If role is Doctor:
   - Select specialization
   - Set consultation fee
   - Assign to department (for Hospital type)
   - Set working schedule
6. If role is Nurse:
   - Assign to department (optional)
   - Assign to support specific doctors (optional)
7. Sets user permissions (if custom permissions needed)
8. Reviews and creates user account
9. System validates email uniqueness within organization
10. System creates user account
11. Organization Admin manually provides credentials to new user

**Postcondition**: New user account is created and user can log in

**Alternative Flow**:
- If email already exists in organization, show error
- If user limit reached (SaaS), show error and suggest contacting Super Admin

---

#### Scenario 5.2.3: Configuring Doctor Schedule

**Actor**: Organization Admin  
**Precondition**: Doctor account exists  
**Goal**: Set up doctor's working hours and availability

**Main Flow**:
1. OrganizationAdmin navigates to Doctor Management
2. Selects doctor from list
3. Clicks "Configure Schedule"
4. Sets weekly schedule:
   - Monday: 9:00 AM - 5:00 PM (with lunch break 1:00 PM - 2:00 PM)
   - Tuesday: 9:00 AM - 5:00 PM
   - ... repeats for each day
   - Marks off days (e.g., Sunday)
5. Sets consultation duration for this doctor (overrides default if needed)
6. Sets maximum appointments per day (optional)
7. Marks upcoming leave/vacation dates:
   - Date range
   - Reason (optional)
8. Saves schedule configuration
9. System validates no overlapping time slots
10. System updates doctor availability calendar

**Postcondition**: Doctor's schedule is configured and available for appointment booking

---

### 5.3 Receptionist Scenarios

#### Scenario 5.3.1: Registering New Patient

**Actor**: Receptionist  
**Precondition**: Receptionist is logged in  
**Goal**: Register walk-in patient or patient calling for appointment

**Main Flow**:
1. Patient arrives at clinic or calls for appointment
2. Receptionist clicks "New Patient Registration"
3. Fills in patient details:
   - Full name
   - Date of birth (system calculates age)
   - Gender
   - Phone number (primary)
   - Email address (optional)
   - Complete address
   - Blood group (if known)
   - Emergency contact name and phone
4. Records medical history:
   - Known allergies
   - Chronic conditions
   - Current medications
   - Previous surgeries
   - Family medical history
5. Captures identification:
   - ID type (National ID, Passport, etc.)
   - ID number
6. Takes patient photo (optional)
7. Reviews entered information with patient
8. Saves patient record
9. System generates unique Patient ID
10. System displays Patient ID prominently
11. Receptionist provides Patient ID to patient (verbal or written)

**Postcondition**: Patient is registered and can now book appointments

**Alternative Flow**:
- If patient has incomplete information, save as draft and mark fields to complete
- If phone number already exists, show warning about potential duplicate

---

#### Scenario 5.3.2: Scheduling Appointment for Patient

**Actor**: Receptionist  
**Precondition**: Patient is registered in the system  
**Goal**: Book appointment with doctor

**Main Flow**:
1. Receptionist searches for patient by name, phone, or Patient ID
2. Selects correct patient from search results
3. Clicks "Book Appointment"
4. System displays patient's upcoming and past appointments
5. Receptionist enters appointment details:
   - Selects department (if Hospital type)
   - Selects doctor from available list
   - Selects preferred date
6. System displays doctor's available time slots for selected date
7. Receptionist selects available time slot
8. Adds appointment reason/notes (e.g., "Follow-up", "Fever and cough")
9. Confirms appointment type (New Consultation / Follow-up)
10. Reviews appointment details with patient
11. Clicks "Confirm Appointment"
12. System validates:
    - Time slot still available
    - No conflicting appointments for this patient
13. System creates appointment
14. System displays appointment confirmation with:
    - Appointment ID
    - Date and time
    - Doctor name
    - Token number (for that day)
15. Receptionist manually notes down appointment details for patient
16. If patient is present, receptionist can print appointment slip

**Postcondition**: Appointment is scheduled and visible in system

**Alternative Flow 1**: No available slots on preferred date
- Receptionist suggests alternative dates
- Shows next available date for selected doctor

**Alternative Flow 2**: Walk-in patient needs immediate consultation
- Receptionist marks as "Walk-in" appointment
- Assigns to current time slot if doctor is available
- Adds to waiting queue

---

#### Scenario 5.3.3: Managing Walk-in Queue

**Actor**: Receptionist  
**Precondition**: Some patients have walked in without appointments  
**Goal**: Manage walk-in patients and assign them to doctors

**Main Flow**:
1. Receptionist registers walk-in patient (if new) or searches existing patient
2. Clicks "Add to Walk-in Queue"
3. Selects doctor patient wants to see
4. System shows:
   - Current scheduled appointments
   - Current walk-in queue
   - Estimated wait time
5. Receptionist informs patient of estimated wait time
6. If patient agrees, adds to queue
7. System assigns token number
8. Receptionist provides token number to patient
9. Throughout the day, receptionist monitors queue on dashboard
10. When doctor becomes available, receptionist:
    - Views next patient in queue
    - Calls patient by token number
    - Updates status to "In Consultation"

**Postcondition**: Walk-in patients are properly queued and managed

---

### 5.4 Doctor Scenarios

#### Scenario 5.4.1: Viewing Daily Schedule and Patient Queue

**Actor**: Doctor  
**Precondition**: Doctor is logged in  
**Goal**: See today's appointments and manage consultation flow

**Main Flow**:
1. Doctor logs into system
2. System displays dashboard with:
   - Today's date and total appointments
   - List of scheduled appointments with times
   - Walk-in patients in queue
   - Completed consultations count
   - Pending consultations count
3. For each appointment, displays:
   - Token number
   - Patient name and age
   - Appointment time
   - Appointment type (New/Follow-up)
   - Status (Scheduled/Waiting/In Progress/Completed)
4. Doctor views next patient to see:
   - Patient photo (if available)
   - Chief complaint/reason for visit
   - Last visit date
5. Doctor clicks "Start Consultation" on patient
6. System updates status to "In Progress"
7. Opens patient medical record

**Postcondition**: Doctor is ready to begin consultation

---

#### Scenario 5.4.2: Conducting Patient Consultation

**Actor**: Doctor  
**Precondition**: Patient is in consultation room  
**Goal**: Complete patient consultation and create medical record

**Main Flow**:
1. Doctor clicks on patient from queue/appointment list
2. System displays patient summary:
   - Demographics
   - Recent visit history
   - Active prescriptions
   - Known allergies (highlighted in red)
   - Chronic conditions
3. Doctor reviews vital signs recorded by nurse:
   - Blood pressure
   - Pulse rate
   - Temperature
   - Weight
   - Oxygen saturation
4. Doctor clicks "Create Medical Record" for this visit
5. Enters chief complaint: "Persistent cough and fever for 3 days"
6. Documents present illness:
   - Onset and duration
   - Associated symptoms
   - Severity
   - Aggravating/relieving factors
7. Records physical examination findings:
   - General appearance
   - System-wise examination
   - Relevant findings
8. Enters diagnosis:
   - Searches for condition: "Acute Bronchitis"
   - System suggests ICD-10 code: J20.9
   - Adds diagnosis
9. Creates treatment plan:
   - Prescribes medications (detailed in next scenario)
   - Orders lab tests if needed
   - Provides lifestyle advice
   - Sets follow-up requirements
10. Adds doctor's notes (private notes not visible to patient)
11. Marks any referrals needed
12. Reviews complete medical record
13. Saves medical record
14. Marks consultation as "Completed"

**Postcondition**: Medical record is created and consultation is complete

---

#### Scenario 5.4.3: Creating Prescription

**Actor**: Doctor  
**Precondition**: Medical record is being created/edited  
**Goal**: Prescribe medications for patient

**Main Flow**:
1. Within medical record screen, doctor clicks "Add Prescription"
2. System opens prescription creation form
3. For first medication:
   - Doctor types medication name: "Amoxicillin"
   - System shows autocomplete suggestions from medication database
   - Doctor selects "Amoxicillin 500mg Capsules"
4. Enters dosage details:
   - Dosage: 500mg
   - Frequency: Three times daily (TDS)
   - Duration: 7 days
   - Special instructions: "Take after meals"
5. Clicks "Add Medication" to add to prescription
6. Repeats for second medication:
   - Medication: "Salbutamol Syrup 100ml"
   - Dosage: 5ml
   - Frequency: Three times daily
   - Duration: 5 days
   - Instructions: "Shake well before use"
7. Adds general prescription notes: "Complete full course of antibiotics"
8. Reviews prescription:
   - System checks for drug interactions (basic)
   - System checks against patient allergies
   - Shows warnings if any
9. Confirms prescription
10. System generates prescription with:
    - Prescription number
    - Date
    - Doctor details and signature
    - Patient details
    - All medications with instructions
11. Doctor can print prescription immediately or patient can get it at reception

**Postcondition**: Prescription is created and linked to medical record

**Alternative Flow**: Drug interaction detected
- System shows warning dialog
- Doctor reviews interaction
- Doctor can override with reason or change medication

---

#### Scenario 5.4.4: Ordering Lab Tests

**Actor**: Doctor  
**Precondition**: Medical record is being created  
**Goal**: Order laboratory tests for patient

**Main Flow**:
1. Within medical record, doctor clicks "Order Lab Tests"
2. System displays lab test catalog organized by categories:
   - Blood Tests
   - Urine Tests
   - Imaging
   - Other Tests
3. Doctor searches or browses for required test: "Complete Blood Count"
4. Selects "Complete Blood Count (CBC)" from results
5. System shows test details:
   - Test name and code
   - Sample type: Blood
   - Preparation required: Fasting (if applicable)
   - Expected turnaround time
6. Doctor adds test to order
7. Adds more tests if needed (e.g., "Blood Sugar - Fasting")
8. For each test, adds clinical indication: "Rule out anemia"
9. Marks urgency (Routine / Urgent)
10. Reviews test order
11. Confirms and saves
12. System generates lab request form with:
    - Test order number
    - Patient details
    - Tests ordered
    - Clinical indication
    - Doctor details
13. Form is printed and given to patient

**Postcondition**: Lab tests are ordered and form is generated

**Note**: In Version 1.0, lab test results will be manually entered by staff later

---

#### Scenario 5.4.5: Reviewing Patient Medical History

**Actor**: Doctor  
**Precondition**: Patient has previous visits  
**Goal**: Review comprehensive patient history before consultation

**Main Flow**:
1. Doctor opens patient record
2. Clicks "Medical History" tab
3. System displays chronological visit history showing:
   - Visit dates
   - Consulting doctor
   - Chief complaints
   - Diagnoses
   - Prescriptions given
4. Doctor can filter history by:
   - Date range
   - Diagnosis type
   - Specific conditions
5. Doctor clicks on specific previous visit
6. System displays complete details of that visit:
   - Vital signs at that time
   - Symptoms documented
   - Examination findings
   - Diagnosis with ICD-10 code
   - Treatment provided
   - Prescriptions given
   - Lab tests ordered and results
7. Doctor can view prescription history:
   - All medications ever prescribed
   - Date ranges
   - Prescribing doctors
8. Doctor reviews ongoing treatments
9. Doctor notes any chronic conditions
10. Returns to current consultation with context

**Postcondition**: Doctor has complete understanding of patient history

---

### 5.5 Nurse Scenarios

#### Scenario 5.5.1: Recording Patient Vitals

**Actor**: Nurse  
**Precondition**: Patient has checked in for appointment  
**Goal**: Record vital signs before doctor consultation

**Main Flow**:
1. Nurse logs into system
2. Views list of patients waiting for vitals recording
3. Calls patient by name/token number
4. Searches for patient record
5. Clicks "Record Vitals" 
6. Measures and enters:
   - Blood Pressure: 120/80 mmHg
   - Pulse Rate: 72 bpm
   - Temperature: 98.6°F (system can convert to Celsius)
   - Respiratory Rate: 16 breaths/min
   - Weight: 70 kg
   - Height: 170 cm (system calculates BMI: 24.2)
   - Oxygen Saturation (SpO2): 98%
7. Adds any observation notes: "Patient appears comfortable"
8. Saves vital signs
9. System timestamps entry with nurse's name
10. System updates patient status to "Ready for Doctor"
11. Vital signs appear in doctor's dashboard

**Postcondition**: Patient vitals are recorded and available for doctor

---

### 5.6 Billing Staff Scenarios

#### Scenario 5.6.1: Generating Invoice for Completed Consultation

**Actor**: Billing Staff  
**Precondition**: Doctor has completed consultation and medical record  
**Goal**: Generate invoice for services rendered

**Main Flow**:
1. Billing staff views list of completed consultations awaiting billing
2. Selects patient consultation from list
3. Clicks "Generate Invoice"
4. System pre-populates invoice with:
   - Patient details
   - Doctor details
   - Consultation date
   - Medical record reference
5. System automatically adds consultation fee based on:
   - Doctor's consultation rate
   - Consultation type (New vs. Follow-up)
6. Billing staff reviews and adds additional charges:
   - If procedures were done, adds procedure charges
   - If lab tests ordered, adds lab test fees
   - If medications provided (if clinic has pharmacy), adds medication charges
7. System calculates:
   - Subtotal
   - Any applicable taxes
   - Total amount
8. Reviews invoice details with patient:
   - Item-wise breakup
   - Quantities
   - Unit prices
   - Total amount
9. If patient requests discount:
   - Billing staff checks if authorized to give discount
   - Applies approved discount with reason
   - System recalculates total
10. Confirms invoice generation
11. System assigns invoice number
12. System saves invoice with status "Issued"
13. Presents invoice to patient

**Postcondition**: Invoice is generated and ready for payment

---

#### Scenario 5.6.2: Recording Payment

**Actor**: Billing Staff  
**Precondition**: Invoice has been generated  
**Goal**: Record patient payment

**Main Flow**:
1. Billing staff has invoice displayed on screen
2. Patient indicates payment method
3. **For Cash Payment**:
   - Billing staff selects "Cash" as payment method
   - Enters amount received: 1000 (example)
   - System shows amount due: 850
   - System calculates change to return: 150
   - Billing staff accepts payment
4. **For Card Payment**:
   - Billing staff selects "Card" as payment method
   - Processes payment on external card machine
   - Receives transaction approval
   - Manually enters card transaction details:
     - Last 4 digits of card
     - Transaction ID/Reference number
     - Amount: 850
5. **For Check Payment**:
   - Billing staff selects "Check" as payment method
   - Enters check details:
     - Check number
     - Bank name
     - Check date
     - Amount: 850
6. Clicks "Record Payment"
7. System validates payment amount
8. System updates invoice status to "Paid"
9. System generates receipt with:
   - Receipt number
   - Date and time
   - Patient details
   - Invoice reference
   - Payment method
   - Amount paid
   - Organization details
10. Prints receipt
11. Provides receipt to patient
12. If change due (cash), returns change

**Postcondition**: Payment is recorded and receipt is issued

**Alternative Flow**: Partial Payment
- Patient cannot pay full amount
- Billing staff records partial payment
- System updates invoice status to "Partially Paid"
- System shows outstanding balance
- Future payments can be recorded against same invoice

---

#### Scenario 5.6.3: Handling Refund Request

**Actor**: Billing Staff  
**Precondition**: Payment has been made but service was not rendered or error in billing  
**Goal**: Process refund for patient

**Main Flow**:
1. Patient requests refund with reason (e.g., "Appointment cancelled after payment")
2. Billing staff verifies:
   - Original invoice
   - Payment record
   - Reason for refund
3. Checks refund approval authority:
   - Small amounts: Billing staff can approve
   - Large amounts: Requires Organization Admin approval
4. If approval needed, forwards request to Organization Admin
5. Once approved, creates refund record:
   - Links to original invoice
   - Selects refund reason from dropdown
   - Enters refund amount
   - Enters refund method (Cash / Bank Transfer)
6. If cash refund:
   - Issues cash immediately
   - Gets patient signature on refund receipt
7. If bank transfer:
   - Records bank account details
   - Marks as "Pending Transfer"
   - Processes transfer externally
   - Updates status to "Refunded" once complete
8. System generates credit note
9. System updates original invoice status
10. Provides refund receipt to patient

**Postcondition**: Refund is processed and documented

---

### 5.7 Patient Scenarios (Patient Portal - If Implemented)

#### Scenario 5.7.1: Viewing Own Medical Records

**Actor**: Patient  
**Precondition**: Patient has login credentials  
**Goal**: Access own medical history

**Main Flow**:
1. Patient logs into patient portal using credentials
2. System displays patient dashboard with:
   - Upcoming appointments
   - Recent visits
   - Active prescriptions
   - Pending payments
3. Patient clicks "Medical Records"
4. System displays list of all visits with:
   - Date of visit
   - Doctor name
   - Diagnosis
   - View details option
5. Patient selects specific visit
6. System displays:
   - Vital signs recorded
   - Chief complaint
   - Diagnosis
   - Prescriptions
   - Lab tests ordered (without detailed results if confidential)
   - Follow-up instructions
7. Patient can:
   - Download/print medical record
   - Download/print prescription
   - View prescription history
8. For active prescriptions, can see:
   - Medication names
   - Dosages
   - Duration remaining

**Postcondition**: Patient has accessed their medical information

**Note**: Certain sensitive information may be hidden based on organization policy

---

### 5.8 Cross-Role Scenarios

#### Scenario 5.8.1: Complete Patient Visit Workflow (All Roles)

**Actors**: Receptionist, Nurse, Doctor, Billing Staff  
**Goal**: Handle complete patient journey from registration to payment

**Main Flow**:

**Step 1: Patient Check-in (Receptionist)**
1. Patient arrives at 10:00 AM for 10:30 AM appointment
2. Receptionist searches patient by name: "John Doe"
3. Finds patient record and checks appointment
4. Confirms appointment details with patient
5. Marks patient as "Checked In"
6. Prints token number: T-05
7. Directs patient to waiting area

**Step 2: Vital Signs (Nurse)**
8. Nurse sees "John Doe - T-05" in queue at 10:15 AM
9. Calls patient: "Token number T-05, John Doe"
10. Takes patient to vitals room
11. Records vital signs:
    - BP: 130/85 mmHg
    - Pulse: 78 bpm
    - Temperature: 99.2°F
    - Weight: 75 kg
12. Notes: "Patient reports fever since yesterday"
13. Saves vitals
14. Directs patient back to waiting area
15. Patient status updated to "Ready for Doctor"

**Step 3: Doctor Consultation (Doctor)**
16. Doctor sees "John Doe - T-05 - Ready" on dashboard at 10:30 AM
17. Calls patient into consultation room
18. Clicks on patient to view:
    - Today's vitals (elevated temp)
    - Medical history (no chronic conditions)
    - No known allergies
19. Starts consultation - status changes to "In Progress"
20. Interviews patient about symptoms
21. Conducts physical examination
22. Creates medical record:
    - Chief Complaint: "Fever and body ache for 2 days"
    - Examination: "Throat inflammation noted"
    - Diagnosis: "Acute Pharyngitis (J02.9)"
23. Creates prescription:
    - Paracetamol 500mg - TDS for 3 days
    - Azithromycin 500mg - OD for 3 days
    - Chlorhexidine mouth rinse
24. Advises: "Rest, plenty of fluids, avoid cold foods"
25. Sets follow-up: "If not better in 3 days"
26. Saves and completes medical record
27. Marks consultation as "Completed" at 10:45 AM

**Step 4: Billing (Billing Staff)**
28. Billing staff sees completed consultation in queue
29. Opens consultation details
30. Generates invoice:
    - Consultation Fee: $50
    - Subtotal: $50
    - Total: $50
31. Calls patient: "John Doe, please proceed to billing counter"
32. Shows invoice to patient
33. Patient pays by card
34. Processes payment on card machine
35. Enters transaction ID: TXN123456
36. Records payment in system
37. Generates receipt
38. Prints receipt and prescription
39. Provides documents to patient

**Step 5: Patient Exit**
40. Patient collects prescription from reception
41. Receptionist provides printed prescription
42. Patient leaves clinic at 10:55 AM

**Postcondition**: Complete patient visit documented, billed, and patient satisfied

**Total Time**: 55 minutes from arrival to exit

---

#### Scenario 5.8.2: Emergency Patient Handling

**Actors**: Receptionist, Doctor, Billing Staff  
**Goal**: Handle urgent patient needing immediate attention

**Main Flow**:
1. Patient walks in with acute chest pain
2. Receptionist immediately recognizes urgency
3. Instead of normal registration:
   - Clicks "Emergency Registration"
   - Enters minimal details: Name, Age, Phone
   - Assigns temporary Patient ID
4. Immediately notifies available doctor
5. Doctor prioritizes emergency patient
6. Conducts examination immediately
7. Creates emergency medical record
8. If patient needs referral to hospital:
   - Creates referral letter
   - Provides emergency treatment summary
   - Calls ambulance if needed
9. If patient can be treated:
   - Provides immediate care
   - Creates prescription
   - Advises follow-up
10. Billing is done after stabilization
11. Complete registration details collected when patient is stable

**Postcondition**: Emergency patient receives immediate care

---

### 5.9 Reporting Scenarios

#### Scenario 5.9.1: Generating Daily Collection Report

**Actor**: Organization Admin or Billing Staff  
**Precondition**: End of business day  
**Goal**: Generate daily financial summary

**Main Flow**:
1. User navigates to Reports section
2. Selects "Daily Collection Report"
3. Selects date: Today's date
4. Clicks "Generate Report"
5. System generates report showing:
   - Total consultations: 25
   - Total revenue: $1,250
   - Payment breakdown:
     - Cash: $600 (12 transactions)
     - Card: $550 (11 transactions)
     - Check: $100 (2 transactions)
   - Outstanding invoices: 3 (Amount: $150)
   - Refunds issued: 1 (Amount: $50)
6. Report also shows:
   - Revenue by doctor
   - Revenue by service type
   - Peak consultation hours
7. User can:
   - Export to PDF
   - Export to Excel
   - Print report
8. Saves report for records

**Postcondition**: Daily financial summary is generated and documented

---

### 5.10 System Administration Scenarios

#### Scenario 5.10.1: Organization Admin Reviewing System Usage

**Actor**: Organization Admin  
**Goal**: Monitor system usage and performance

**Main Flow**:
1. Organization Admin logs in
2. Views dashboard showing:
   - Today's statistics:
     - Total appointments: 30 (Completed: 25, Pending: 3, Cancelled: 2)
     - Total patients seen: 25
     - Total revenue: $1,250
     - New patient registrations: 5
   - This week's summary
   - This month's summary
3. Views doctor-wise statistics:
   - Dr. Smith: 15 patients, $750 revenue
   - Dr. Jones: 10 patients, $500 revenue
4. Checks appointment utilization:
   - Scheduled slots: 35
   - Filled appointments: 30
   - Utilization rate: 85.7%
5. Reviews patient satisfaction (if feedback system implemented)
6. Identifies areas needing attention:
   - High no-show rate on Mondays
   - Peak hours: 10 AM - 12 PM
7. Adjusts scheduling accordingly

**Postcondition**: Admin has insight into operations and can make informed decisions

---

## 6. High-Level Design (HLD)

### 6.1 System Architecture Overview

The Medical Management System follows a **Clean Architecture** pattern with a **monolithic deployment** model. The architecture is designed to support both SaaS multi-tenant and private deployment scenarios while maintaining clear separation of concerns.

```mermaid
graph TB
    subgraph "Presentation Layer"
        A[Angular Web Application]
        B[Admin Dashboard]
        C[Patient Portal]
    end
    
    subgraph "API Gateway Layer"
        D[ASP.NET Core API Gateway]
        E[Authentication Middleware]
        F[Tenant Context Middleware]
        G[Logging Middleware]
    end
    
    subgraph "Application Core - Clean Architecture"
        H[API Controllers Layer]
        I[Application Services Layer]
        J[Domain Layer - Business Logic]
        K[Infrastructure Layer]
    end
    
    subgraph "Data Access Layer"
        L[(SQL Server / PostgreSQL)]
        M[(Redis Cache)]
        N[File Storage]
    end
    
    subgraph "Cross-Cutting Concerns"
        O[Logging - Serilog]
        P[Validation - FluentValidation]
        Q[Exception Handling]
    end
    
    A --> D
    B --> D
    C --> D
    
    D --> E
    E --> F
    F --> G
    G --> H
    
    H --> I
    I --> J
    J --> K
    
    K --> L
    K --> M
    K --> N
    
    I --> P
    H --> Q
    K --> O
```

### 6.2 Clean Architecture Layers

```mermaid
graph LR
    subgraph "Domain Layer - Core"
        A[Entities]
        B[Value Objects]
        C[Enums]
        D[Domain Exceptions]
        E[Domain Interfaces]
    end
    
    subgraph "Application Layer"
        F[Services]
        G[DTOs]
        H[Service Interfaces]
        I[Validators - FluentValidation]
        J[Manual Mappers]
    end
    
    subgraph "Infrastructure Layer"
        K[DbContext]
        L[Repositories]
        M[External Services]
        N[File Storage]
    end
    
    subgraph "Presentation Layer"
        O[API Controllers]
        P[Filters]
        Q[Middleware]
    end
    
    O --> F
    F --> A
    F --> I
    F --> J
    K --> A
    L --> A
    F --> H
    L --> E
```

**Layer Responsibilities**:

1. **Domain Layer** (Innermost - No Dependencies):
   - Pure business entities
   - Business rules
   - Domain exceptions
   - Interfaces (repository contracts)

2. **Application Layer** (Depends only on Domain):
   - Application services implementing use cases
   - DTOs for data transfer
   - Manual mapping between entities and DTOs
   - FluentValidation validators
   - Service interfaces

3. **Infrastructure Layer** (Depends on Domain & Application):
   - Entity Framework Core DbContext
   - Repository implementations
   - External service integrations (future: email, SMS)
   - File storage implementations
   - Database migrations

4. **Presentation Layer** (API):
   - ASP.NET Core Web API controllers
   - Request/response models
   - Authentication/authorization attributes
   - API filters and middleware

### 6.3 Multi-Tenant Architecture

```mermaid
graph TB
    subgraph "Request Flow"
        A[HTTP Request with JWT Token]
        B[Authentication Middleware]
        C[Extract Organization ID from Claims]
        D[Set Tenant Context]
        E[Application Service]
        F[Repository Layer]
        G[DbContext with Global Query Filter]
        H[Database Query with WHERE OrganizationId = X]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    
    subgraph "Tenant Isolation"
        I[User Login]
        J[JWT with OrganizationId Claim]
        K[Every Request Includes Token]
        L[Automatic Data Filtering]
    end
    
    I --> J
    J --> K
    K --> L
```

**Multi-Tenancy Implementation Strategy**:

1. **Shared Database, Shared Schema**:
   - Single database for all tenants
   - OrganizationId column in all tenant-specific tables
   - Global query filters in EF Core ensure data isolation

2. **Tenant Context**:
   - Extracted from JWT token claims
   - Set in scoped service for request lifetime
   - Available throughout application layers

3. **Data Isolation**:
   - All queries automatically filtered by OrganizationId
   - Repository methods enforce tenant context
   - No cross-tenant data access possible

4. **Deployment Model Switching**:
   - Configuration flag determines SaaS vs. Private mode
   - Private mode: Single organization, no Super Admin
   - SaaS mode: Multiple organizations, Super Admin enabled

### 6.4 Authentication & Authorization Flow

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant API
    participant Auth Service
    participant Database
    participant Token Service
    
    User->>Frontend: Enter Credentials
    Frontend->>API: POST /api/auth/login
    API->>Auth Service: Validate Credentials
    Auth Service->>Database: Query User by Email
    Database-->>Auth Service: User Record + Organization
    Auth Service->>Auth Service: Verify Password Hash
    
    alt Valid Credentials
        Auth Service->>Token Service: Generate JWT Token
        Token Service->>Token Service: Add Claims (UserId, Role, OrgId)
        Token Service-->>Auth Service: JWT Token + Refresh Token
        Auth Service-->>API: Authentication Success
        API-->>Frontend: Tokens + User Info
        Frontend->>Frontend: Store Tokens
        Frontend-->>User: Redirect to Dashboard
    else Invalid Credentials
        Auth Service-->>API: Authentication Failed
        API-->>Frontend: Error Response
        Frontend-->>User: Show Error Message
    end
```

**JWT Token Claims**:
```json
{
  "sub": "user_id",
  "email": "user@example.com",
  "role": "Doctor",
  "organizationId": "123",
  "organizationType": "Hospital",
  "departmentId": "456",
  "exp": 1234567890,
  "iat": 1234567890
}
```

### 6.5 Data Access Architecture

```mermaid
graph TB
    subgraph "Application Layer"
        A[Service Layer]
    end
    
    subgraph "Repository Pattern"
        B[IGenericRepository Interface]
        C[IPatientRepository Interface]
        D[IAppointmentRepository Interface]
        E[Generic Repository Implementation]
        F[Patient Repository Implementation]
        G[Appointment Repository Implementation]
    end
    
    subgraph "Entity Framework Core"
        H[ApplicationDbContext]
        I[Entity Configurations]
        J[Global Query Filters]
        K[Interceptors]
    end
    
    subgraph "Database"
        L[(SQL Server / PostgreSQL)]
    end
    
    A --> B
    A --> C
    A --> D
    
    B -.implements.- E
    C -.implements.- F
    D -.implements.- G
    
    E --> H
    F --> H
    G --> H
    
    H --> I
    H --> J
    H --> K
    H --> L
```

**Repository Pattern Benefits**:
- Abstraction over data access
- Testability through interface mocking
- Centralized query logic
- Easier to change data source

**Global Query Filter Example**:
```
OnModelCreating:
  For each entity with OrganizationId property:
    Apply filter: WHERE OrganizationId == CurrentTenantContext.OrganizationId
```

###6.6 Caching Strategy

```mermaid
graph TB
    subgraph "Cache Layers"
        A[Application Service Request]
        B{Check Redis Cache}
        C[Cache Hit]
        D[Cache Miss]
        E[Query Database]
        F[Store in Cache]
        G[Return Data]
    end
    
    A --> B
    B -->|Found| C
    B -->|Not Found| D
    C --> G
    D --> E
    E --> F
    F --> G
```

**Caching Implementation**:

1. **Cached Data Types**:
   - User sessions and permissions
   - Organization configuration
   - Department lists
   - Doctor schedules
   - Service pricing
   - Medication master list
   - Lab test catalog

2. **Cache Invalidation**:
   - Time-based expiration (TTL)
   - Event-based invalidation on updates
   - Manual cache clear option for admins

3. **Cache Key Pattern**:
   ```
   {OrganizationId}:{EntityType}:{EntityId}
   Example: 123:Doctor:456:Schedule
   ```

### 6.7 Deployment Architecture (Monolithic)

```mermaid
graph TB
    subgraph "Internet"
        A[Users - Web Browsers]
    end
    
    subgraph "Load Balancer / Reverse Proxy"
        B[NGINX / IIS]
    end
    
    subgraph "Web Server Tier"
        C[Application Server 1]
        D[Application Server 2 - Optional]
    end
    
    subgraph "Application Components - Monolithic"
        E[ASP.NET Core Web API]
        F[Angular SPA - Static Files]
    end
    
    subgraph "Cache Layer"
        G[Redis Server]
    end
    
    subgraph "Database Tier"
        H[(Primary Database)]
        I[(Read Replica - Optional)]
    end
    
    subgraph "File Storage"
        J[Local File System / Network Storage]
    end
    
    A --> B
    B --> C
    B --> D
    
    C --> E
    C --> F
    D --> E
    D --> F
    
    E --> G
    E --> H
    E --> I
    E --> J
```

**Deployment Options**:

1. **Single Server Deployment** (Small Clinics):
   - Single server hosting application + database
   - Suitable for < 50 concurrent users
   - Cost-effective for private deployments

2. **Two-Tier Deployment** (Medium Clinics):
   - Separate application and database servers
   - Suitable for 50-200 concurrent users
   - Better security and performance

3. **Scaled Deployment** (Large Hospitals / SaaS):
   - Multiple application servers behind load balancer
   - Separate database server with optional read replicas
   - Redis cache server
   - Separate file storage
   - Suitable for 200+ concurrent users

### 6.8 Security Architecture

```mermaid
graph TB
    subgraph "Security Layers"
        A[HTTPS / TLS 1.3]
        B[Authentication - JWT]
        C[Authorization - Role-Based]
        D[Multi-Tenant Isolation]
        E[Data Encryption at Rest]
        F[Audit Logging]
        G[Input Validation]
    end
    
    subgraph "Request Flow"
        H[Client Request]
        I[SSL/TLS Termination]
        J[Authentication Check]
        K[Authorization Check]
        L[Tenant Context Validation]
        M[Input Validation]
        N[Business Logic]
        O[Audit Log]
        P[Response]
    end
    
    H --> I
    I --> J
    J --> K
    K --> L
    L --> M
    M --> N
    N --> O
    O --> P
    
    A --- I
    B --- J
    C --- K
    D --- L
    G --- M
    F --- O
```

**Security Measures**:

1. **Transport Security**:
   - HTTPS enforced for all connections
   - TLS 1.3 for encryption
   - HSTS headers

2. **Authentication**:
   - JWT Bearer tokens
   - Password hashing (Bcrypt)
   - Token expiration and refresh
   - Account lockout after failed attempts

3. **Authorization**:
   - Role-based permissions
   - Resource-based authorization
   - Organization-level isolation

4. **Data Protection**:
   - Sensitive data encryption
   - Database encryption at rest
   - Secure file storage permissions

5. **Audit & Compliance**:
   - All data access logged
   - User action tracking
   - Compliance with healthcare standards

---

## 7. Entity Relationship Diagrams (ERD)

### 7.1 Core Entities ERD

```mermaid
erDiagram
    ORGANIZATION ||--o{ USER : employs
    ORGANIZATION ||--o{ PATIENT : serves
    ORGANIZATION ||--o{ DEPARTMENT : contains
    ORGANIZATION ||--o{ APPOINTMENT : hosts
    ORGANIZATION ||--o{ SERVICE : offers
    
    ORGANIZATION {
        int Id PK
        string Name
        string Type
        string Address
        string Phone
        string Email
        string LogoUrl
        json WorkingHours
        string Currency
        string Timezone
        bool IsActive
        datetime CreatedAt
        datetime UpdatedAt
    }
    
    DEPARTMENT ||--o{ USER : contains
    DEPARTMENT {
        int Id PK
        int OrganizationId FK
        string Name
        string Description
        int HeadDoctorId FK
        bool IsActive
        datetime CreatedAt
    }
    
    USER ||--o{ APPOINTMENT : attends_as_doctor
    USER ||--o{ MEDICAL_RECORD : creates
    USER ||--o{ USER_SCHEDULE : has
    USER ||--o{ AUDIT_LOG : generates
    
    USER {
        int Id PK
        int OrganizationId FK
        int DepartmentId FK
        string FirstName
        string LastName
        string Email
        string PasswordHash
        string Role
        string Phone
        string Specialization
        decimal ConsultationFee
        string ProfilePictureUrl
        bool IsActive
        int FailedLoginAttempts
        datetime LastLoginAt
        datetime CreatedAt
        datetime UpdatedAt
    }
    
    USER_SCHEDULE {
        int Id PK
        int UserId FK
        int OrganizationId FK
        string DayOfWeek
        time StartTime
        time EndTime
        time BreakStartTime
        time BreakEndTime
        int ConsultationDuration
        bool IsAvailable
        datetime CreatedAt
    }
    
    PATIENT ||--o{ APPOINTMENT : books
    PATIENT ||--o{ MEDICAL_RECORD : has
    PATIENT ||--o{ PATIENT_DOCUMENT : owns
    PATIENT ||--o{ INVOICE : receives
    
    PATIENT {
        int Id PK
        int OrganizationId FK
        string PatientNumber
        string FirstName
        string LastName
        date DateOfBirth
        int Age
        string Gender
        string Phone
        string Email
        string Address
        string BloodGroup
        text Allergies
        text ChronicConditions
        text MedicalHistory
        string EmergencyContactName
        string EmergencyContactPhone
        string PhotoUrl
        string IdentificationType
        string IdentificationNumber
        bool IsActive
        datetime CreatedAt
        datetime UpdatedAt
    }
    
    APPOINTMENT ||--|| MEDICAL_RECORD : generates
    APPOINTMENT {
        int Id PK
        string AppointmentNumber
        int PatientId FK
        int DoctorId FK
        int OrganizationId FK
        int DepartmentId FK
        datetime AppointmentDate
        time StartTime
        time EndTime
        int Duration
        string Status
        string AppointmentType
        string ChiefComplaint
        string TokenNumber
        bool IsWalkIn
        text Notes
        datetime CheckInTime
        datetime CompletedTime
        datetime CancelledAt
        string CancellationReason
        datetime CreatedAt
        datetime UpdatedAt
    }
```

### 7.2 Medical Records ERD

```mermaid
erDiagram
    MEDICAL_RECORD ||--o{ VITAL_SIGNS : includes
    MEDICAL_RECORD ||--o{ PRESCRIPTION : contains
    MEDICAL_RECORD ||--o{ LAB_TEST_ORDER : orders
    MEDICAL_RECORD ||--o{ DIAGNOSIS : documents
    MEDICAL_RECORD ||--|| INVOICE : generates
    
    MEDICAL_RECORD {
        int Id PK
        string RecordNumber
        int PatientId FK
        int DoctorId FK
        int AppointmentId FK
        int OrganizationId FK
        datetime VisitDate
        text ChiefComplaint
        text PresentIllness
        text PhysicalExamination
        text TreatmentPlan
        text DoctorNotes
        text FollowUpInstructions
        date NextFollowUpDate
        string Status
        datetime CreatedAt
        datetime UpdatedAt
        int CreatedBy FK
        int UpdatedBy FK
    }
    
    VITAL_SIGNS {
        int Id PK
        int MedicalRecordId FK
        int PatientId FK
        decimal BloodPressureSystolic
        decimal BloodPressureDiastolic
        decimal PulseRate
        decimal Temperature
        string TemperatureUnit
        decimal RespiratoryRate
        decimal Weight
        string WeightUnit
        decimal Height
        string HeightUnit
        decimal BMI
        decimal OxygenSaturation
        text Notes
        datetime RecordedAt
        int RecordedBy FK
    }
    
    DIAGNOSIS {
        int Id PK
        int MedicalRecordId FK
        string DiagnosisName
        string ICD10Code
        string Type
        string Severity
        text Notes
        datetime DiagnosedAt
    }
    
    PRESCRIPTION ||--o{ PRESCRIPTION_ITEM : contains
    PRESCRIPTION {
        int Id PK
        string PrescriptionNumber
        int MedicalRecordId FK
        int PatientId FK
        int DoctorId FK
        int OrganizationId FK
        datetime PrescribedDate
        text GeneralInstructions
        text Notes
        int ValidityDays
        datetime ExpiresAt
        datetime CreatedAt
    }
    
    PRESCRIPTION_ITEM ||--o{ MEDICATION : references
    PRESCRIPTION_ITEM {
        int Id PK
        int PrescriptionId FK
        int MedicationId FK
        string MedicationName
        string Dosage
        string DosageUnit
        string Frequency
        string Route
        int Duration
        string DurationUnit
        text SpecialInstructions
        int Quantity
        datetime CreatedAt
    }
    
    MEDICATION {
        int Id PK
        int OrganizationId FK
        string Name
        string GenericName
        string BrandName
        string Type
        string Category
        text Description
        text SideEffects
        text Contraindications
        bool RequiresPrescription
        bool IsActive
        datetime CreatedAt
    }
    
    LAB_TEST_ORDER ||--o{ LAB_TEST_RESULT : produces
    LAB_TEST_ORDER {
        int Id PK
        string OrderNumber
        int MedicalRecordId FK
        int PatientId FK
        int DoctorId FK
        int OrganizationId FK
        int LabTestId FK
        string TestName
        text ClinicalIndication
        string Priority
        string SampleType
        string Status
        datetime OrderedAt
        datetime SampleCollectedAt
        datetime ResultDate
        datetime CreatedAt
    }
    
    LAB_TEST_RESULT {
        int Id PK
        int LabTestOrderId FK
        string TestParameter
        string Result
        string Unit
        string NormalRange
        bool IsAbnormal
        text Comments
        string ResultFileUrl
        datetime ResultDate
        int EnteredBy FK
    }
    
    LAB_TEST {
        int Id PK
        int OrganizationId FK
        string TestCode
        string TestName
        string Category
        text Description
        string SampleType
        text Preparation
        decimal Price
        int TurnaroundTimeDays
        bool IsActive
        datetime CreatedAt
    }
```

### 7.3 Billing & Financial ERD

```mermaid
erDiagram
    INVOICE ||--o{ INVOICE_ITEM : contains
    INVOICE ||--o{ PAYMENT : receives
    INVOICE ||--o{ CREDIT_NOTE : may_have
    
    INVOICE {
        int Id PK
        string InvoiceNumber
        int PatientId FK
        int OrganizationId FK
        int MedicalRecordId FK
        int AppointmentId FK
        datetime InvoiceDate
        date DueDate
        decimal SubTotal
        decimal TaxAmount
        decimal TaxPercentage
        decimal DiscountAmount
        decimal DiscountPercentage
        string DiscountReason
        decimal TotalAmount
        decimal PaidAmount
        decimal BalanceAmount
        string Status
        text Notes
        datetime CreatedAt
        datetime UpdatedAt
        int CreatedBy FK
    }
    
    INVOICE_ITEM {
        int Id PK
        int InvoiceId FK
        int ServiceId FK
        string ItemType
        string ServiceName
        string Description
        int Quantity
        decimal UnitPrice
        decimal Discount
        decimal TotalPrice
        datetime CreatedAt
    }
    
    SERVICE {
        int Id PK
        int OrganizationId FK
        int DepartmentId FK
        string ServiceCode
        string ServiceName
        string ServiceType
        string Category
        text Description
        decimal Price
        string PriceUnit
        bool IsTaxable
        bool IsActive
        datetime CreatedAt
        datetime UpdatedAt
    }
    
    PAYMENT {
        int Id PK
        string PaymentNumber
        int InvoiceId FK
        int PatientId FK
        int OrganizationId FK
        datetime PaymentDate
        decimal Amount
        string PaymentMethod
        string TransactionId
        string CardLastFourDigits
        string CheckNumber
        string BankName
        string PaymentStatus
        text Notes
        datetime CreatedAt
        int ReceivedBy FK
    }
    
    CREDIT_NOTE {
        int Id PK
        string CreditNoteNumber
        int InvoiceId FK
        int PatientId FK
        int OrganizationId FK
        datetime IssueDate
        decimal Amount
        string Reason
        string RefundMethod
        string Status
        text Notes
        datetime CreatedAt
        int ApprovedBy FK
    }
```

### 7.4 System Administration ERD

```mermaid
erDiagram
    ORGANIZATION ||--o{ SUBSCRIPTION : has
    ORGANIZATION ||--o{ ORGANIZATION_SETTINGS : configures
    USER ||--o{ AUDIT_LOG : generates
    PATIENT_DOCUMENT ||--o{ PATIENT : belongs_to
    
    SUBSCRIPTION {
        int Id PK
        int OrganizationId FK
        string PlanType
        int UserLimit
        int StorageLimitGB
        decimal MonthlyFee
        date StartDate
        date EndDate
        bool IsActive
        bool AutoRenew
        text Notes
        datetime CreatedAt
        datetime UpdatedAt
        int CreatedBy FK
    }
    
    ORGANIZATION_SETTINGS {
        int Id PK
        int OrganizationId FK
        string SettingKey
        string SettingValue
        string DataType
        text Description
        datetime UpdatedAt
        int UpdatedBy FK
    }
    
    AUDIT_LOG {
        int Id PK
        int OrganizationId FK
        int UserId FK
        string Action
        string EntityType
        int EntityId
        json OldValues
        json NewValues
        string IPAddress
        string UserAgent
        datetime CreatedAt
    }
    
    PATIENT_DOCUMENT {
        int Id PK
        int PatientId FK
        int OrganizationId FK
        int UploadedBy FK
        string DocumentType
        string FileName
        string FileUrl
        long FileSizeBytes
        string MimeType
        text Description
        datetime UploadedAt
    }
    
    SYSTEM_CONFIG {
        int Id PK
        string ConfigKey
        string ConfigValue
        string Category
        text Description
        bool IsEditable
        datetime UpdatedAt
    }
    
    NOTIFICATION_TEMPLATE {
        int Id PK
        int OrganizationId FK
        string TemplateName
        string TemplateType
        string Subject
        text Body
        bool IsActive
        datetime CreatedAt
        datetime UpdatedAt
    }
```

### 7.5 Database Indexes Strategy

**Critical Indexes for Performance**:

1. **Multi-Column Indexes**:
   ```
   Appointments: (OrganizationId, AppointmentDate, DoctorId)
   Appointments: (OrganizationId, PatientId, AppointmentDate)
   MedicalRecords: (OrganizationId, PatientId, VisitDate)
   Invoices: (OrganizationId, Status, InvoiceDate)
   Patients: (OrganizationId, Phone)
   Patients: (OrganizationId, Email)
   ```

2. **Single Column Indexes**:
   ```
   Users: (Email)
   Patients: (PatientNumber)
   Appointments: (AppointmentNumber)
   Prescriptions: (PrescriptionNumber)
   Invoices: (InvoiceNumber)
   ```

3. **Foreign Key Indexes**:
   ```
   All foreign key columns automatically indexed
   ```

---

## 8. Sequence Diagrams

### 8.1 User Authentication & Token Generation

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant API Controller
    participant Auth Service
    participant User Repository
    participant Database
    participant Token Service
    participant Tenant Context
    
    User->>Frontend: Enter Email & Password
    Frontend->>API Controller: POST /api/auth/login {email, password}
    API Controller->>Auth Service: AuthenticateAsync(credentials)
    Auth Service->>User Repository: GetUserByEmailAsync(email)
    User Repository->>Database: SELECT * FROM Users WHERE Email = @email
    Database-->>User Repository: User Record + Organization
    User Repository-->>Auth Service: User Entity
    
    alt User Not Found
        Auth Service-->>API Controller: Null
        API Controller-->>Frontend: 401 Unauthorized
        Frontend-->>User: Invalid Credentials
    else User Found
        Auth Service->>Auth Service: VerifyPasswordHash(password, user.PasswordHash)
        
        alt Invalid Password
            Auth Service->>User Repository: IncrementFailedLoginAttempts(userId)
            Auth Service-->>API Controller: Authentication Failed
            API Controller-->>Frontend: 401 Unauthorized
            Frontend-->>User: Invalid Credentials
        else Valid Password
            Auth Service->>User Repository: ResetFailedLoginAttempts(userId)
            Auth Service->>User Repository: UpdateLastLogin(userId)
            Auth Service->>Token Service: GenerateJwtToken(user)
            Token Service->>Token Service: Create JWT with Claims
            Note over Token Service: Claims: UserId, Email, Role,<br/>OrganizationId, DepartmentId
            Token Service->>Token Service: Generate Refresh Token
            Token Service-->>Auth Service: JWT + Refresh Token
            Auth Service-->>API Controller: AuthResult {token, refreshToken, user}
            API Controller-->>Frontend: 200 OK + Tokens + User Data
            Frontend->>Frontend: Store Tokens in LocalStorage
            Frontend->>Frontend: Set Authorization Header
            Frontend-->>User: Redirect to Dashboard
        end
    end
```

---

### 8.2 Multi-Tenant Data Access Flow

```mermaid
sequenceDiagram
    actor Doctor
    participant Frontend
    participant API Controller
    participant Auth Middleware
    participant Tenant Middleware
    participant Tenant Context
    participant Service Layer
    participant Repository
    participant EF DbContext
    participant Database
    
    Doctor->>Frontend: Request Patient List
    Frontend->>API Controller: GET /api/patients<br/>Header: Authorization: Bearer {JWT}
    API Controller->>Auth Middleware: Validate Token
    Auth Middleware->>Auth Middleware: Decode JWT
    Auth Middleware->>Auth Middleware: Extract Claims
    Auth Middleware-->>API Controller: User Authenticated
    
    API Controller->>Tenant Middleware: Process Request
    Tenant Middleware->>Tenant Middleware: Extract OrganizationId from Claims
    Tenant Middleware->>Tenant Context: SetTenant(organizationId)
    Tenant Context->>Tenant Context: Store OrganizationId in Scoped Service
    Tenant Middleware-->>API Controller: Tenant Context Set
    
    API Controller->>Service Layer: GetAllPatientsAsync()
    Service Layer->>Repository: GetAllAsync()
    Repository->>EF DbContext: patients.ToListAsync()
    
    Note over EF DbContext: Global Query Filter Applied:<br/>WHERE OrganizationId = {CurrentTenantId}
    
    EF DbContext->>Database: SELECT * FROM Patients<br/>WHERE OrganizationId = 123<br/>AND IsActive = 1
    Database-->>EF DbContext: Filtered Patient Records
    EF DbContext-->>Repository: List<Patient>
    Repository-->>Service Layer: List<Patient>
    Service Layer->>Service Layer: Manual Mapping to DTOs
    Service Layer-->>API Controller: List<PatientDto>
    API Controller-->>Frontend: 200 OK + Patient List
    Frontend-->>Doctor: Display Patients
```

---

### 8.3 Appointment Booking Flow

```mermaid
sequenceDiagram
    actor Receptionist
    participant Frontend
    participant API
    participant Appointment Service
    participant Doctor Schedule Service
    participant Patient Service
    participant Notification Service
    participant Database
    participant Cache
    
    Receptionist->>Frontend: Search Patient "John Doe"
    Frontend->>API: GET /api/patients/search?query=John Doe
    API->>Patient Service: SearchPatientsAsync("John Doe")
    Patient Service->>Database: Query Patients
    Database-->>Patient Service: Patient Results
    Patient Service-->>Frontend: Display Patient List
    
    Receptionist->>Frontend: Select Patient
    Frontend->>API: GET /api/patients/{patientId}
    API->>Patient Service: GetPatientByIdAsync(patientId)
    Patient Service->>Cache: Check Cache
    Cache-->>Patient Service: Cache Miss
    Patient Service->>Database: Query Patient
    Database-->>Patient Service: Patient Details
    Patient Service->>Cache: Store in Cache
    Patient Service-->>Frontend: Patient Data
    
    Receptionist->>Frontend: Click "Book Appointment"
    Frontend->>API: GET /api/doctors?organizationId={orgId}
    API->>Doctor Schedule Service: GetAvailableDoctorsAsync(orgId)
    Doctor Schedule Service->>Cache: Check Cached Doctor List
    Cache-->>Doctor Schedule Service: Doctor List
    Doctor Schedule Service-->>Frontend: Available Doctors
    
    Receptionist->>Frontend: Select Doctor & Date
    Frontend->>API: GET /api/appointments/available-slots<br/>?doctorId={id}&date={date}
    API->>Appointment Service: GetAvailableSlotsAsync(doctorId, date)
    Appointment Service->>Doctor Schedule Service: GetDoctorSchedule(doctorId, date)
    Doctor Schedule Service->>Database: Query Schedule & Existing Appointments
    Database-->>Doctor Schedule Service: Schedule + Appointments
    Doctor Schedule Service->>Doctor Schedule Service: Calculate Available Slots
    Doctor Schedule Service-->>Appointment Service: Available Time Slots
    Appointment Service-->>Frontend: Time Slots Array
    
    Receptionist->>Frontend: Select Time Slot & Confirm
    Frontend->>API: POST /api/appointments {patientId, doctorId, date, time}
    API->>Appointment Service: CreateAppointmentAsync(appointmentDto)
    
    Appointment Service->>Appointment Service: Validate Request (FluentValidation)
    Appointment Service->>Database: BEGIN TRANSACTION
    
    Appointment Service->>Database: Check Slot Still Available
    Database-->>Appointment Service: Slot Available = true
    
    alt Slot Not Available
        Appointment Service-->>API: Conflict Error
        API-->>Frontend: 409 Conflict
        Frontend-->>Receptionist: Time slot no longer available
    else Slot Available
        Appointment Service->>Database: INSERT INTO Appointments
        Database-->>Appointment Service: Appointment Created
        
        Appointment Service->>Database: COMMIT TRANSACTION
        
        Appointment Service->>Notification Service: QueueAppointmentConfirmation(appointment)
        Note over Notification Service: In V1.0: No actual sending<br/>Just logs for future implementation
        
        Appointment Service->>Cache: Invalidate Doctor Schedule Cache
        Appointment Service-->>API: Appointment Created Successfully
        API-->>Frontend: 201 Created + Appointment Details
        Frontend->>Frontend: Generate Appointment Slip
        Frontend-->>Receptionist: Display Success + Print Option
    end
```

---

### 8.4 Doctor Consultation Flow

```mermaid
sequenceDiagram
    actor Doctor
    participant Frontend
    participant API
    participant Medical Record Service
    participant Prescription Service
    participant Lab Test Service
    participant Billing Service
    participant Database
    
    Doctor->>Frontend: View Today's Appointments
    Frontend->>API: GET /api/appointments/today
    API->>Medical Record Service: GetTodaysAppointmentsAsync(doctorId)
    Medical Record Service->>Database: Query Appointments
    Database-->>Frontend: Appointment List with Patient Info
    
    Doctor->>Frontend: Select Patient & Click "Start Consultation"
    Frontend->>API: GET /api/patients/{id}/history
    API->>Medical Record Service: GetPatientHistoryAsync(patientId)
    Medical Record Service->>Database: Query Medical Records
    Database-->>Frontend: Patient History + Vitals + Prescriptions
    
    Frontend->>API: PATCH /api/appointments/{id}/status {status: "InProgress"}
    API->>Database: UPDATE Appointment Status
    
    Doctor->>Frontend: Enter Consultation Details
    Doctor->>Frontend: Click "Save Medical Record"
    Frontend->>API: POST /api/medical-records {recordData}
    API->>Medical Record Service: CreateMedicalRecordAsync(recordDto)
    
    Medical Record Service->>Medical Record Service: Validate (FluentValidation)
    Medical Record Service->>Database: BEGIN TRANSACTION
    Medical Record Service->>Database: INSERT Medical Record
    Database-->>Medical Record Service: Record Created with ID
    
    alt Has Prescriptions
        Medical Record Service->>Prescription Service: CreatePrescriptionAsync(prescriptionDto)
        Prescription Service->>Prescription Service: Validate Medications
        Prescription Service->>Database: Check Drug Interactions
        Database-->>Prescription Service: Interaction Results
        
        alt Critical Interaction Found
            Prescription Service-->>Frontend: Warning Message
            Doctor->>Frontend: Override or Modify
        end
        
        Prescription Service->>Database: INSERT Prescription
        Prescription Service->>Database: INSERT Prescription Items
        Database-->>Prescription Service: Prescription Created
    end
    
    alt Has Lab Tests
        Medical Record Service->>Lab Test Service: CreateLabOrderAsync(labOrderDto)
        Lab Test Service->>Database: INSERT Lab Test Order
        Database-->>Lab Test Service: Order Created
    end
    
    Medical Record Service->>Database: UPDATE Appointment Status = "Completed"
    
    Medical Record Service->>Billing Service: TriggerInvoiceGeneration(medicalRecordId)
    Note over Billing Service: Creates Draft Invoice<br/>Ready for Billing Staff
    
    Medical Record Service->>Database: COMMIT TRANSACTION
    Medical Record Service-->>API: Medical Record Created
    API-->>Frontend: 201 Created + Record Details
    Frontend-->>Doctor: Success Message + Print Options
```

---

### 8.5 Billing & Payment Flow

```mermaid
sequenceDiagram
    actor Billing Staff
    actor Patient
    participant Frontend
    participant API
    participant Billing Service
    participant Invoice Service
    participant Payment Service
    participant Database
    
    Billing Staff->>Frontend: View Completed Consultations
    Frontend->>API: GET /api/medical-records/pending-billing
    API->>Billing Service: GetPendingBillingRecordsAsync()
    Billing Service->>Database: Query Records
    Database-->>Frontend: Pending Consultations List
    
    Billing Staff->>Frontend: Select Consultation & Click "Generate Invoice"
    Frontend->>API: POST /api/invoices/generate {medicalRecordId}
    API->>Invoice Service: GenerateInvoiceAsync(medicalRecordId)
    
    Invoice Service->>Database: Get Medical Record Details
    Database-->>Invoice Service: Medical Record + Appointment
    
    Invoice Service->>Database: Get Applicable Services
    Database-->>Invoice Service: Consultation Fee + Service Prices
    
    Invoice Service->>Invoice Service: Calculate Invoice Items
    Note over Invoice Service: - Consultation Fee<br/>- Lab Tests (if any)<br/>- Procedures (if any)
    
    Invoice Service->>Invoice Service: Calculate Total
    Note over Invoice Service: Subtotal + Tax - Discount = Total
    
    Invoice Service->>Database: INSERT Invoice
    Invoice Service->>Database: INSERT Invoice Items
    Database-->>Invoice Service: Invoice Created
    Invoice Service-->>API: Invoice DTO
    API-->>Frontend: Invoice Details
    
    Billing Staff->>Frontend: Review Invoice with Patient
    
    alt Apply Discount
        Billing Staff->>Frontend: Enter Discount Amount/Percentage
        Frontend->>API: PATCH /api/invoices/{id}/discount
        API->>Invoice Service: ApplyDiscountAsync(invoiceId, discount)
        Invoice Service->>Database: UPDATE Invoice
        Database-->>Frontend: Updated Invoice
    end
    
    Patient->>Billing Staff: Ready to Pay
    Billing Staff->>Frontend: Record Payment
    
    alt Cash Payment
        Frontend->>API: POST /api/payments {invoiceId, amount, method: "Cash"}
    else Card Payment
        Billing Staff->>Billing Staff: Process on External Card Machine
        Billing Staff->>Frontend: Enter Transaction Details
        Frontend->>API: POST /api/payments {invoiceId, amount, method: "Card", transactionId}
    else Check Payment
        Billing Staff->>Frontend: Enter Check Details
        Frontend->>API: POST /api/payments {invoiceId, amount, method: "Check", checkNumber}
    end
    
    API->>Payment Service: RecordPaymentAsync(paymentDto)
    Payment Service->>Payment Service: Validate Payment
    Payment Service->>Database: BEGIN TRANSACTION
    Payment Service->>Database: INSERT Payment Record
    
    Payment Service->>Database: UPDATE Invoice.PaidAmount
    Payment Service->>Database: UPDATE Invoice.BalanceAmount
    
    alt Full Payment
        Payment Service->>Database: UPDATE Invoice.Status = "Paid"
    else Partial Payment
        Payment Service->>Database: UPDATE Invoice.Status = "PartiallyPaid"
    end
    
    Payment Service->>Database: COMMIT TRANSACTION
    Payment Service->>Payment Service: Generate Receipt
    Payment Service-->>API: Payment Receipt
    API-->>Frontend: Receipt Details
    
    Frontend->>Frontend: Open Print Dialog
    Billing Staff->>Frontend: Print Receipt
    Frontend-->>Patient: Provide Printed Receipt
```

---

### 8.6 Lab Test Result Entry Flow (Manual)

```mermaid
sequenceDiagram
    actor Lab Staff
    participant Frontend
    participant API
    participant Lab Test Service
    participant Notification Service
    participant File Storage
    participant Database
    
    Lab Staff->>Frontend: View Pending Lab Orders
    Frontend->>API: GET /api/lab-tests/pending
    API->>Lab Test Service: GetPendingOrdersAsync()
    Lab Test Service->>Database: Query Pending Orders
    Database-->>Frontend: Pending Lab Orders List
    
    Lab Staff->>Frontend: Select Order & Click "Enter Results"
    Frontend->>API: GET /api/lab-tests/orders/{orderId}
    API->>Lab Test Service: GetOrderDetailsAsync(orderId)
    Lab Test Service->>Database: Query Order
    Database-->>Frontend: Order Details + Patient Info
    
    Lab Staff->>Frontend: Enter Test Results Manually
    Frontend->>Frontend: Fill Result Form
    Note over Frontend: - Test Parameter<br/>- Result Value<br/>- Unit<br/>- Normal Range<br/>- Mark if Abnormal
    
    alt Has Result PDF/Image
        Lab Staff->>Frontend: Upload Result File
        Frontend->>API: POST /api/files/upload
        API->>File Storage: Save File
        File Storage-->>API: File URL
    end
    
    Lab Staff->>Frontend: Submit Results
    Frontend->>API: POST /api/lab-tests/{orderId}/results {resultData}
    API->>Lab Test Service: SaveLabResultsAsync(orderId, resultDto)
    
    Lab Test Service->>Lab Test Service: Validate Results
    Lab Test Service->>Database: BEGIN TRANSACTION
    Lab Test Service->>Database: INSERT Lab Test Results
    Lab Test Service->>Database: UPDATE Order Status = "Completed"
    Lab Test Service->>Database: UPDATE ResultDate
    Lab Test Service->>Database: COMMIT TRANSACTION
    
    Lab Test Service->>Notification Service: NotifyDoctorResultReady(orderId)
    Note over Notification Service: V1.0: Creates notification record<br/>V2.0: Sends email/SMS
    
    Lab Test Service-->>API: Results Saved Successfully
    API-->>Frontend: 200 OK
    Frontend-->>Lab Staff: Success Message
```

---

## 9. Flow Charts

### 9.1 User Login & Authentication Flow

```mermaid
flowchart TD
    A[Start: User Visits Login Page] --> B[Enter Email & Password]
    B --> C[Click Login Button]
    C --> D[Frontend Validates Input]
    D --> E{Input Valid?}
    
    E -->|No| F[Show Validation Errors]
    F --> B
    
    E -->|Yes| G[Send POST /api/auth/login]
    G --> H[Backend Validates Credentials]
    H --> I{User Exists?}
    
    I -->|No| J[Return 401 Unauthorized]
    J --> K[Show "Invalid Credentials" Message]
    K --> B
    
    I -->|Yes| L{Password Correct?}
    
    L -->|No| M[Increment Failed Login Attempts]
    M --> N{Failed Attempts >= 5?}
    N -->|Yes| O[Lock Account]
    O --> P[Return "Account Locked" Error]
    P --> Q[End]
    
    N -->|No| J
    
    L -->|Yes| R{Account Active?}
    R -->|No| S[Return "Account Inactive" Error]
    S --> K
    
    R -->|Yes| T[Reset Failed Login Attempts]
    T --> U[Update Last Login Time]
    U --> V[Generate JWT Token with Claims]
    V --> W[Generate Refresh Token]
    W --> X[Return Tokens + User Data]
    X --> Y[Frontend Stores Tokens]
    Y --> Z[Set Auth Header for Future Requests]
    Z --> AA{User Role?}
    
    AA -->|Super Admin| AB[Redirect to Super Admin Dashboard]
    AA -->|Org Admin| AC[Redirect to Admin Dashboard]
    AA -->|Doctor| AD[Redirect to Doctor Dashboard]
    AA -->|Nurse| AE[Redirect to Nurse Dashboard]
    AA -->|Receptionist| AF[Redirect to Receptionist Dashboard]
    AA -->|Billing Staff| AG[Redirect to Billing Dashboard]
    AA -->|Patient| AH[Redirect to Patient Portal]
    
    AB --> Q
    AC --> Q
    AD --> Q
    AE --> Q
    AF --> Q
    AG --> Q
    AH --> Q
```

---

### 9.2 Patient Registration Flow

```mermaid
flowchart TD
    A[Start: Patient Arrives/Calls] --> B{Existing Patient?}
    
    B -->|Yes| C[Search Patient by Phone/Name/ID]
    C --> D[Found Patient Record]
    D --> E[Verify Patient Identity]
    E --> F{Details Need Update?}
    
    F -->|Yes| G[Update Patient Information]
    G --> H[Save Updated Record]
    H --> I[Proceed to Appointment Booking]
    
    F -->|No| I
    
    B -->|No| J[Click "New Patient Registration"]
    J --> K[Open Registration Form]
    K --> L[Enter Basic Information]
    Note over L: Name, DOB, Gender,<br/>Phone, Email, Address
    
    L --> M[Enter Medical Information]
    Note over M: Blood Group, Allergies,<br/>Chronic Conditions,<br/>Current Medications
    
    M --> N[Enter Emergency Contact]
    N --> O{Photo Available?}
    O -->|Yes| P[Capture/Upload Photo]
    O -->|No| Q[Skip Photo]
    P --> R[Validate All Fields]
    Q --> R
    
    R --> S{Validation Passed?}
    S -->|No| T[Show Validation Errors]
    T --> L
    
    S -->|Yes| U{Phone/Email Already Exists?}
    U -->|Yes| V[Show Duplicate Warning]
    V --> W{Continue Anyway?}
    W -->|No| L
    W -->|Yes| X[Create Patient Record]
    
    U -->|No| X
    X --> Y[Generate Unique Patient ID]
    Y --> Z[Save to Database]
    Z --> AA[Display Patient ID to Staff]
    AA --> AB[Provide Patient ID to Patient]
    AB --> I
    I --> AC[End]
```

---

### 9.3 Appointment Scheduling Decision Flow

```mermaid
flowchart TD
    A[Start: Schedule Appointment] --> B{Organization Type?}
    
    B -->|Personal Clinic| C[Single Doctor - Auto Select]
    B -->|Multi-Doctor Clinic| D[Show Doctor List]
    B -->|Hospital| E[Select Department First]
    
    C --> F[Load Doctor Schedule]
    
    D --> G[Receptionist Selects Doctor]
    G --> F
    
    E --> H[Show Doctors in Department]
    H --> G
    
    F --> I[Select Preferred Date]
    I --> J[Fetch Available Time Slots]
    J --> K{Slots Available?}
    
    K -->|No| L[Show Message: No Slots Available]
    L --> M[Suggest Next Available Date]
    M --> N{Accept Alternative?}
    N -->|Yes| I
    N -->|No| O[End - Appointment Not Booked]
    
    K -->|Yes| P[Display Available Time Slots]
    P --> Q[Receptionist Selects Time Slot]
    Q --> R[Enter Appointment Details]
    Note over R: Reason for Visit,<br/>Appointment Type,<br/>Special Notes
    
    R --> S{Walk-in Patient?}
    S -->|Yes| T[Mark as Walk-in]
    T --> U[Assign Current Time]
    U --> V[Add to Waiting Queue]
    
    S -->|No| W[Confirm Selected Time]
    W --> X[Review Appointment Summary]
    
    V --> X
    X --> Y{Details Correct?}
    Y -->|No| R
    
    Y -->|Yes| Z[Click "Confirm Appointment"]
    Z --> AA[Validate Slot Still Available]
    AA --> AB{Still Available?}
    
    AB -->|No| AC[Show Error: Slot Taken]
    AC --> J
    
    AB -->|Yes| AD[Create Appointment Record]
    AD --> AE[Assign Token Number]
    AE --> AF[Generate Appointment Confirmation]
    AF --> AG{Print Appointment Slip?}
    
    AG -->|Yes| AH[Print Slip with Details]
    AG -->|No| AI[Note Details Manually]
    
    AH --> AJ[Provide to Patient]
    AI --> AJ
    AJ --> AK[Update Doctor's Schedule]
    AK --> AL[End - Appointment Confirmed]
```

---

### 9.4 Medical Record Creation Workflow

```mermaid
flowchart TD
    A[Start: Doctor Selects Patient] --> B[View Patient Summary]
    B --> C[Check Recent Vitals]
    C --> D{Vitals Recorded?}
    
    D -->|No| E[Request Nurse to Record Vitals]
    E --> F[Wait for Vitals]
    F --> C
    
    D -->|Yes| G[Review Medical History]
    G --> H[Check Active Prescriptions]
    H --> I[Check Known Allergies]
    I --> J[Start Consultation]
    J --> K[Update Appointment Status to "In Progress"]
    K --> L[Click "Create Medical Record"]
    L --> M[Enter Chief Complaint]
    M --> N[Document Present Illness]
    Note over N: Onset, Duration,<br/>Associated Symptoms,<br/>Severity
    
    N --> O[Record Physical Examination]
    O --> P[Enter Diagnosis]
    P --> Q[Search ICD-10 Code]
    Q --> R[Select Appropriate Code]
    R --> S{Need Lab Tests?}
    
    S -->|Yes| T[Click "Order Lab Tests"]
    T --> U[Search Test Catalog]
    U --> V[Select Required Tests]
    V --> W[Add Clinical Indication]
    W --> X[Confirm Lab Order]
    
    S -->|No| Y{Need Prescription?}
    X --> Y
    
    Y -->|Yes| Z[Click "Add Prescription"]
    Z --> AA[Search Medication Database]
    AA --> AB[Select Medication]
    AB --> AC[Enter Dosage Details]
    Note over AC: Dosage, Frequency,<br/>Duration, Instructions
    
    AC --> AD{Check Drug Interactions}
    AD --> AE{Interaction Found?}
    
    AE -->|Yes - Critical| AF[Show Warning Dialog]
    AF --> AG{Override?}
    AG -->|No| AA
    AG -->|Yes| AH[Document Override Reason]
    AH --> AI[Add Medication to Prescription]
    
    AE -->|No| AI
    AE -->|Yes - Minor| AJ[Show Info Message]
    AJ --> AI
    
    AI --> AK{Add More Medications?}
    AK -->|Yes| AA
    AK -->|No| AL[Review Complete Prescription]
    
    Y -->|No| AM[Add Treatment Plan]
    AL --> AM
    
    AM --> AN[Add Doctor's Private Notes]
    AN --> AO{Follow-up Needed?}
    
    AO -->|Yes| AP[Set Follow-up Date]
    AP --> AQ[Add Follow-up Instructions]
    AQ --> AR[Review Complete Medical Record]
    
    AO -->|No| AR
    
    AR --> AS{Everything Correct?}
    AS -->|No| AT{What to Modify?}
    AT -->|Complaints| M
    AT -->|Examination| O
    AT -->|Diagnosis| P
    AT -->|Prescription| Z
    AT -->|Treatment Plan| AM
    
    AS -->|Yes| AU[Click "Save Medical Record"]
    AU --> AV[Validate All Required Fields]
    AV --> AW{Validation Passed?}
    
    AW -->|No| AX[Show Validation Errors]
    AX --> AR
    
    AW -->|Yes| AY[Save Medical Record to Database]
    AY --> AZ[Update Appointment Status to "Completed"]
    AZ --> BA[Trigger Invoice Generation]
    BA --> BB[Generate Prescription PDF]
    BB --> BC{Print Documents?}
    
    BC -->|Yes| BD[Print Medical Summary]
    BD --> BE[Print Prescription]
    BE --> BF[Print Lab Request Form if any]
    
    BC -->|No| BG[Make Available for Later Printing]
    
    BF --> BH[Provide Documents to Patient]
    BG --> BH
    BH --> BI[End - Consultation Complete]
```

---

### 9.5 Invoice Generation & Payment Processing

```mermaid
flowchart TD
    A[Start: Billing Staff View Pending Bills] --> B[Select Completed Consultation]
    B --> C[Click "Generate Invoice"]
    C --> D[System Fetches Medical Record]
    D --> E[Load Consultation Fee]
    E --> F{Lab Tests Ordered?}
    
    F -->|Yes| G[Add Lab Test Charges]
    F -->|No| H{Procedures Done?}
    G --> H
    
    H -->|Yes| I[Add Procedure Charges]
    H -->|No| J[Calculate Subtotal]
    I --> J
    
    J --> K{Apply Tax?}
    K -->|Yes| L[Calculate Tax Amount]
    L --> M[Add to Subtotal]
    K -->|No| M
    
    M --> N[Calculate Total Amount]
    N --> O[Display Invoice to Billing Staff]
    O --> P{Patient Requests Discount?}
    
    P -->|Yes| Q{Staff Authorized for Discount?}
    Q -->|No| R[Request Admin Approval]
    R --> S{Approved?}
    S -->|No| T[Proceed Without Discount]
    S -->|Yes| U[Apply Discount]
    
    Q -->|Yes| U
    U --> V[Enter Discount Reason]
    V --> W[Recalculate Total]
    W --> X[Update Invoice]
    
    P -->|No| T
    T --> X
    X --> Y[Review Invoice with Patient]
    Y --> Z{Patient Agrees?}
    
    Z -->|No| AA[Discuss Invoice Items]
    AA --> AB{Modify Invoice?}
    AB -->|Yes| O
    AB -->|No| AC[End - No Payment]
    
    Z -->|Yes| AD{Payment Method?}
    
    AD -->|Cash| AE[Enter Amount Received]
    AE --> AF[Calculate Change]
    AF --> AG[Accept Cash Payment]
    AG --> AH[Record Payment in System]
    AH --> AI{Change Due?}
    AI -->|Yes| AJ[Return Change to Patient]
    AI -->|No| AK[No Change]
    AJ --> AL[Generate Receipt]
    AK --> AL
    
    AD -->|Card| AM[Process on Card Machine]
    AM --> AN{Transaction Approved?}
    AN -->|No| AO[Show Error Message]
    AO --> AP{Retry?}
    AP -->|Yes| AM
    AP -->|No| AD
    
    AN -->|Yes| AQ[Enter Transaction ID]
    AQ --> AR[Enter Card Last 4 Digits]
    AR --> AH
    
    AD -->|Check| AS[Receive Check from Patient]
    AS --> AT[Enter Check Details]
    Note over AT: Check Number,<br/>Bank Name,<br/>Check Date
    AT --> AU[Verify Check Details]
    AU --> AH
    
    AL --> AV[Update Invoice Status]
    AV --> AW{Full Payment?}
    AW -->|Yes| AX[Mark as "Paid"]
    AW -->|No| AY[Mark as "Partially Paid"]
    AX --> AZ[Record Outstanding = 0]
    AY --> BA[Calculate Outstanding Balance]
    
    AZ --> BB[Print Receipt]
    BA --> BB
    BB --> BC[Print Invoice Copy if Requested]
    BC --> BD[Provide Documents to Patient]
    BD --> BE{Outstanding Balance?}
    
    BE -->|Yes| BF[Inform Patient of Balance]
    BF --> BG[Schedule Follow-up for Payment]
    BE -->|No| BH[Transaction Complete]
    
    BG --> BI[End]
    BH --> BI
```

---

### 9.6 System Deployment Mode Selection

```mermaid
flowchart TD
    A[Start: System Installation] --> B{Deployment Type?}
    
    B -->|SaaS Multi-Tenant| C[Configure Multi-Tenant Mode]
    C --> D[Set Configuration: DeploymentMode = "MultiTenant"]
    D --> E[Enable Super Admin Features]
    E --> F[Create Super Admin Account]
    F --> G[Configure Subscription Management]
    G --> H[Set Organization Limits]
    H --> I[Configure Multi-Tenant Database]
    I --> J[Apply Global Query Filters]
    J --> K[Initialize Organization Context Middleware]
    K --> L[Configure Cross-Tenant Security]
    L --> M[System Ready for Multi-Tenant Use]
    M --> N[Super Admin Can Create Organizations]
    N --> O[End]
    
    B -->|Private Deployment| P[Configure Single-Tenant Mode]
    P --> Q[Set Configuration: DeploymentMode = "Private"]
    Q --> R[Disable Super Admin Features]
    R --> S[Create Single Organization Record]
    S --> T[Set OrganizationId as Fixed Value]
    T --> U[Create Organization Admin Account]
    U --> V[Grant Admin Full Permissions]
    V --> W[Hide Subscription Management UI]
    W --> X[Configure Single Organization Context]
    X --> Y[Simplify Database Queries - Fixed OrgId]
    Y --> Z[Remove Organization Switcher UI]
    Z --> AA[System Ready for Private Use]
    AA --> AB[Org Admin Manages Everything]
    AB --> O
```

---

## 10. Technical Implementation Details

### 10.1 Technology Stack

#### 10.1.1 Backend Technology Stack

**Framework & Platform**:
- **.NET 8.0** (LTS) - Latest long-term support version
- **ASP.NET Core 8.0 Web API** - RESTful API framework
- **C# 12** - Programming language

**Architecture & Patterns**:
- **Clean Architecture** (Onion Architecture)
- **Repository Pattern** - Data access abstraction
- **Unit of Work Pattern** - Transaction management
- **CQRS Pattern** (Optional for complex operations)
- **Dependency Injection** - Built-in .NET DI Container

**Data Access**:
- **Entity Framework Core 8.0** - ORM
- **SQL Server 2022** - Primary database (also supports PostgreSQL)
- **Manual Mapping** - No AutoMapper, custom DTOs
- **FluentValidation** - Input validation
- **Dapper** (Optional) - For complex queries and reporting

**Authentication & Security**:
- **JWT Bearer Tokens** - Authentication
- **ASP.NET Core Identity** (Optional) - User management
- **BCrypt.Net** - Password hashing
- **Microsoft.AspNetCore.Authentication.JwtBearer** - JWT middleware

**Caching**:
- **StackExchange.Redis** - Distributed caching
- **IMemoryCache** - In-memory caching for small data

**Logging & Monitoring**:
- **Serilog** - Structured logging
- **Serilog.Sinks.File** - File logging
- **Serilog.Sinks.MSSqlServer** - Database logging
- **Serilog.Sinks.Console** - Console logging

**API Documentation**:
- **Swashbuckle.AspNetCore** (Swagger/OpenAPI) - API documentation
- **Swagger UI** - Interactive API testing

**Additional Libraries**:
- **Newtonsoft.Json** - JSON serialization
- **ClosedXML** - Excel generation for reports
- **QuestPDF** - PDF generation
- **System.Drawing.Common** - Image processing (patient photos)

#### 10.1.2 Frontend Technology Stack

**Framework**:
- **Angular 17+** - Frontend framework
- **TypeScript 5+** - Primary programming language
- **RxJS 7+** - Reactive programming

**UI Framework & Components**:
- **Angular Material 17** - Primary UI component library
- **PrimeNG** (Alternative) - Rich UI components
- **Angular CDK** - Component development kit

**State Management**:
- **NgRx** - Redux-based state management (for complex state)
- **Services with BehaviorSubject** - Simple state management
- **LocalStorage** - Client-side data persistence

**Forms**:
- **Reactive Forms** - Form handling
- **Custom Validators** - Form validation
- **Template-driven Forms** (minimal use)

**HTTP & Communication**:
- **HttpClient** - HTTP communications
- **HTTP Interceptors** - Request/response intercepting
- **RxJS Operators** - Stream manipulation

**Routing & Navigation**:
- **Angular Router** - Application routing
- **Route Guards** - Authentication guards
- **Lazy Loading** - Module optimization

**Additional Libraries**:
- **Chart.js / ng2-charts** - Charts and graphs
- **FullCalendar** - Calendar and scheduling
- **ngx-pagination** - Pagination
- **ngx-print** - Printing functionality
- **ngx-toastr** - Toast notifications
- **angular-file-uploader** - File uploads
- **ngx-barcode** - Barcode generation
- **date-fns** - Date manipulation

**Development Tools**:
- **Angular CLI** - Project scaffolding and building
- **ESLint** - Code linting
- **Prettier** - Code formatting
- **Karma/Jasmine** - Unit testing
- **Protractor/Cypress** - E2E testing

#### 10.1.3 Database

**Primary Database**:
- **SQL Server 2022** (Recommended)
- **PostgreSQL 15+** (Alternative)

**Database Features Used**:
- Transactions and ACID compliance
- Foreign key constraints
- Indexes for performance
- Stored procedures (minimal, for complex reports)
- Views for common queries
- Full-text search (for patient/document search)

#### 10.1.4 Infrastructure

**Caching Layer**:
- **Redis 7+** - Distributed cache
- **Redis Commander** - Redis management UI (development)

**Web Server**:
- **IIS 10+** (Windows deployment)
- **Nginx** (Linux deployment)
- **Kestrel** (built-in .NET web server)

**File Storage**:
- **Local File System** - For private deployments
- **Network Attached Storage (NAS)** - For larger deployments
- **Azure Blob Storage** (Optional for cloud deployment)

**Operating Systems**:
- **Windows Server 2019/2022** - Windows deployment
- **Ubuntu Server 22.04 LTS** - Linux deployment

###10.2 Project Structure

#### 10.2.1 Backend .NET Solution Structure

```
MedicalManagementSystem.sln
│
├── src/
│   ├── MedicalSystem.API/                           # Presentation Layer - Web API
│   │   ├── Controllers/
│   │   │   ├── AuthController.cs
│   │   │   ├── PatientsController.cs
│   │   │   ├── AppointmentsController.cs
│   │   │   ├── MedicalRecordsController.cs
│   │   │   ├── PrescriptionsController.cs
│   │   │   ├── LabTestsController.cs
│   │   │   ├── InvoicesController.cs
│   │   │   ├── PaymentsController.cs
│   │   │   ├── UsersController.cs
│   │   │   ├── OrganizationsController.cs
│   │   │   └── ReportsController.cs
│   │   ├── Middleware/
│   │   │   ├── AuthenticationMiddleware.cs
│   │   │   ├── TenantContextMiddleware.cs
│   │   │   ├── ExceptionHandlingMiddleware.cs
│   │   │   └── LoggingMiddleware.cs
│   │   ├── Filters/
│   │   │   ├── ValidateModelStateAttribute.cs
│   │   │   └── AuthorizeRoleAttribute.cs
│   │   ├── Extensions/
│   │   │   ├── ServiceCollectionExtensions.cs
│   │   │   └── ApplicationBuilderExtensions.cs
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   └── appsettings.Production.json
│   │
│   ├── MedicalSystem.Application/                   # Application Layer
│   │   ├── Services/
│   │   │   ├── AuthService.cs
│   │   │   ├── PatientService.cs
│   │   │   ├── AppointmentService.cs
│   │   │   ├── MedicalRecordService.cs
│   │   │   ├── PrescriptionService.cs
│   │   │   ├── LabTestService.cs
│   │   │   ├── BillingService.cs
│   │   │   ├── InvoiceService.cs
│   │   │   ├── PaymentService.cs
│   │   │   ├── UserService.cs
│   │   │   ├── OrganizationService.cs
│   │   │   ├── DoctorScheduleService.cs
│   │   │   ├── ReportService.cs
│   │   │   └── FileStorageService.cs
│   │   ├── Interfaces/
│   │   │   ├── IAuthService.cs
│   │   │   ├── IPatientService.cs
│   │   │   ├── IAppointmentService.cs
│   │   │   └── ... (other service interfaces)
│   │   ├── DTOs/
│   │   │   ├── Auth/
│   │   │   │   ├── LoginRequestDto.cs
│   │   │   │   ├── LoginResponseDto.cs
│   │   │   │   └── TokenDto.cs
│   │   │   ├── Patient/
│   │   │   │   ├── PatientDto.cs
│   │   │   │   ├── CreatePatientDto.cs
│   │   │   │   └── UpdatePatientDto.cs
│   │   │   ├── Appointment/
│   │   │   ├── MedicalRecord/
│   │   │   ├── Prescription/
│   │   │   ├── Billing/
│   │   │   └── Common/
│   │   │       └── PagedResultDto.cs
│   │   ├── Mappers/                                  # Manual Mapping
│   │   │   ├── PatientMapper.cs
│   │   │   ├── AppointmentMapper.cs
│   │   │   ├── MedicalRecordMapper.cs
│   │   │   └── ... (other mappers)
│   │   ├── Validators/                               # FluentValidation
│   │   │   ├── CreatePatientDtoValidator.cs
│   │   │   ├── CreateAppointmentDtoValidator.cs
│   │   │   └── ... (other validators)
│   │   └── Exceptions/
│   │       ├── BusinessException.cs
│   │       ├── NotFoundException.cs
│   │       └── ValidationException.cs
│   │
│   ├── MedicalSystem.Domain/                         # Domain Layer
│   │   ├── Entities/
│   │   │   ├── Organization.cs
│   │   │   ├── User.cs
│   │   │   ├── Patient.cs
│   │   │   ├── Appointment.cs
│   │   │   ├── MedicalRecord.cs
│   │   │   ├── VitalSigns.cs
│   │   │   ├── Diagnosis.cs
│   │   │   ├── Prescription.cs
│   │   │   ├── PrescriptionItem.cs
│   │   │   ├── Medication.cs
│   │   │   ├── LabTestOrder.cs
│   │   │   ├── LabTestResult.cs
│   │   │   ├── LabTest.cs
│   │   │   ├── Invoice.cs
│   │   │   ├── InvoiceItem.cs
│   │   │   ├── Payment.cs
│   │   │   ├── Service.cs
│   │   │   ├── Department.cs
│   │   │   ├── UserSchedule.cs
│   │   │   ├── AuditLog.cs
│   │   │   └── BaseEntity.cs
│   │   ├── Enums/
│   │   │   ├── UserRole.cs
│   │   │   ├── OrganizationType.cs
│   │   │   ├── AppointmentStatus.cs
│   │   │   ├── InvoiceStatus.cs
│   │   │   ├── PaymentMethod.cs
│   │   │   ├── Gender.cs
│   │   │   └── DayOfWeek.cs
│   │   ├── ValueObjects/
│   │   │   ├── Address.cs
│   │   │   └── ContactInfo.cs
│   │   ├── Interfaces/
│   │   │   ├── IRepository.cs
│   │   │   ├── IPatientRepository.cs
│   │   │   ├── IAppointmentRepository.cs
│   │   │   └── ... (other repository interfaces)
│   │   └── Exceptions/
│   │       └── DomainException.cs
│   │
│   ├── MedicalSystem.Infrastructure/                 # Infrastructure Layer
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── Configurations/
│   │   │   │   ├── OrganizationConfiguration.cs
│   │   │   │   ├── UserConfiguration.cs
│   │   │   │   ├── PatientConfiguration.cs
│   │   │   │   └── ... (other entity configurations)
│   │   │   ├── Interceptors/
│   │   │   │   └── AuditInterceptor.cs
│   │   │   └── Migrations/
│   │   ├── Repositories/
│   │   │   ├── GenericRepository.cs
│   │   │   ├── PatientRepository.cs
│   │   │   ├── AppointmentRepository.cs
│   │   │   ├── MedicalRecordRepository.cs
│   │   │   └── ... (other repositories)
│   │   ├── Services/
│   │   │   ├── TokenService.cs
│   │   │   ├── CacheService.cs
│   │   │   ├── EmailService.cs                      # V2.0
│   │   │   └── SmsService.cs                        # V2.0
│   │   └── UnitOfWork/
│   │       └── UnitOfWork.cs
│   │
│   └── MedicalSystem.Shared/                         # Shared/Common
│       ├── Constants/
│       │   ├── AppConstants.cs
│       │   ├── RoleConstants.cs
│       │   └── CacheKeys.cs
│       ├── Helpers/
│       │   ├── PasswordHasher.cs
│       │   ├── DateTimeHelper.cs
│       │   └── FileHelper.cs
│       └── Extensions/
│           ├── StringExtensions.cs
│           └── EnumExtensions.cs
│
└── tests/                                             # Test Projects
    ├── MedicalSystem.UnitTests/
    ├── MedicalSystem.IntegrationTests/
    └── MedicalSystem.API.Tests/
```

#### 10.2.2 Frontend Angular Structure

```
medical-system-frontend/
│
├── src/
│   ├── app/
│   │   ├── core/                                     # Singleton Services
│   │   │   ├── services/
│   │   │   │   ├── auth.service.ts
│   │   │   │   ├── api.service.ts
│   │   │   │   ├── token.service.ts
│   │   │   │   ├── tenant.service.ts
│   │   │   │   └── notification.service.ts
│   │   │   ├── guards/
│   │   │   │   ├── auth.guard.ts
│   │   │   │   └── role.guard.ts
│   │   │   ├── interceptors/
│   │   │   │   ├── auth.interceptor.ts
│   │   │   │   ├── error.interceptor.ts
│   │   │   │   └── loading.interceptor.ts
│   │   │   ├── models/
│   │   │   │   ├── user.model.ts
│   │   │   │   ├── auth-response.model.ts
│   │   │   │   └── api-response.model.ts
│   │   │   └── core.module.ts
│   │   │
│   │   ├── shared/            # Shared Components & Modules
│   │   │   ├── components/
│   │   │   │   ├── header/
│   │   │   │   ├── sidebar/
│   │   │   │   ├── footer/
│   │   │   │   ├── loading-spinner/
│   │   │   │   ├── confirmation-dialog/
│   │   │   │   ├── data-table/
│   │   │   │   └── page-header/
│   │   │   ├── directives/
│   │   │   │   ├── has-role.directive.ts
│   │   │   │   └── auto-focus.directive.ts
│   │   │   ├── pipes/
│   │   │   │   ├── age.pipe.ts
│   │   │   │   ├── date-format.pipe.ts
│   │   │   │   └── currency-format.pipe.ts
│   │   │   ├── validators/
│   │   │   │   └── custom-validators.ts
│   │   │   └── shared.module.ts
│   │   │
│   │   ├── features/                                 # Feature Modules
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   ├── forgot-password/
│   │   │   │   └── auth-routing.module.ts
│   │   │   │
│   │   │   ├── dashboard/
│   │   │   │   ├── admin-dashboard/
│   │   │   │   ├── doctor-dashboard/
│   │   │   │   ├── receptionist-dashboard/
│   │   │   │   └── dashboard-routing.module.ts
│   │   │   │
│   │   │   ├── patients/
│   │   │   │   ├── patient-list/
│   │   │   │   ├── patient-detail/
│   │   │   │   ├── patient-create/
│   │   │   │   ├── patient-edit/
│   │   │   │   ├── patient-history/
│   │   │   │   ├── services/
│   │   │   │   │   └── patient.service.ts
│   │   │   │   ├── models/
│   │   │   │   │   └── patient.model.ts
│   │   │   │   └── patients-routing.module.ts
│   │   │   │
│   │   │   ├── appointments/
│   │   │   │   ├── appointment-list/
│   │   │   │   ├── appointment-calendar/
│   │   │   │   ├── appointment-create/
│   │   │   │   ├── appointment-detail/
│   │   │   │   ├── doctor-schedule/
│   │   │   │   ├── services/
│   │   │   │   │   └── appointment.service.ts
│   │   │   │   └── appointments-routing.module.ts
│   │   │   │
│   │   │   ├── medical-records/
│   │   │   │   ├── medical-record-list/
│   │   │   │   ├── medical-record-create/
│   │   │   │   ├── medical-record-detail/
│   │   │   │   ├── vital-signs/
│   │   │   │   ├── diagnosis/
│   │   │   │   ├── services/
│   │   │   │   │   └── medical-record.service.ts
│   │   │   │   └── medical-records-routing.module.ts
│   │   │   │
│   │   │   ├── prescriptions/
│   │   │   │   ├── prescription-create/
│   │   │   │   ├── prescription-detail/
│   │   │   │   ├── prescription-history/
│   │   │   │   ├── services/
│   │   │   │   │   └── prescription.service.ts
│   │   │   │   └── prescriptions-routing.module.ts
│   │   │   │
│   │   │   ├── lab-tests/
│   │   │   │   ├── lab-test-list/
│   │   │   │   ├── lab-test-order/
│   │   │   │   ├── lab-test-results/
│   │   │   │   ├── services/
│   │   │   │   │   └── lab-test.service.ts
│   │   │   │   └── lab-tests-routing.module.ts
│   │   │   │
│   │   │   ├── billing/
│   │   │   │   ├── invoice-list/
│   │   │   │   ├── invoice-create/
│   │   │   │   ├── invoice-detail/
│   │   │   │   ├── payment-record/
│   │   │   │   ├── services/
│   │   │   │   │   ├── invoice.service.ts
│   │   │   │   │   └── payment.service.ts
│   │   │   │   └── billing-routing.module.ts
│   │   │   │
│   │   │   ├── reports/
│   │   │   │   ├── financial-reports/
│   │   │   │   ├── patient-reports/
│   │   │   │   ├── appointment-reports/
│   │   │   │   ├── services/
│   │   │   │   │   └── report.service.ts
│   │   │   │   └── reports-routing.module.ts
│   │   │   │
│   │   │   ├── administration/
│   │   │   │   ├── user-management/
│   │   │   │   ├── organization-settings/
│   │   │   │   ├── department-management/
│   │   │   │   ├── service-pricing/
│   │   │   │   ├── services/
│   │   │   │   │   ├── user.service.ts
│   │   │   │   │   └── organization.service.ts
│   │   │   │   └── administration-routing.module.ts
│   │   │   │
│   │   │   └── super-admin/                          # SaaS Only
│   │   │       ├── organization-list/
│   │   │       ├── organization-create/
│   │   │       ├── subscription-management/
│   │   │       ├── services/
│   │   │       │   └── super-admin.service.ts
│   │   │       └── super-admin-routing.module.ts
│   │   │
│   │   ├── layout/                                   # Layout Components
│   │   │   ├── main-layout/
│   │   │   ├── auth-layout/
│   │   │   └── print-layout/
│   │   │
│   │   ├── app-routing.module.ts
│   │   ├── app.component.ts
│   │   └── app.module.ts
│   │
│   ├── assets/
│   │   ├── images/
│   │   ├── icons/
│   │   ├── styles/
│   │   └── i18n/                                     # V2.0
│   │
│   ├── environments/
│   │   ├── environment.ts
│   │   ├── environment.development.ts
│   │   └── environment.production.ts
│   │
│   ├── styles.scss
│   ├── index.html
│   └── main.ts
│
├── angular.json
├── package.json
├── tsconfig.json
└── README.md
```

### 10.3 Configuration Management

#### 10.3.1 Backend Configuration (appsettings.json)

```json
{
  "DeploymentSettings": {
    "Mode": "MultiTenant",  // or "Private"
    "EnableSuperAdmin": true,
    "OrganizationLimit": 100,
    "DefaultOrganizationId": null  // Set for Private mode
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MedicalSystemDb;Trusted_Connection=True;TrustServerCertificate=True",
    "RedisConnection": "localhost:6379"
  },
  "JwtSettings": {
    "SecretKey": "YOUR_SECRET_KEY_HERE_MINIMUM_32_CHARACTERS",
    "Issuer": "MedicalSystem",
    "Audience": "MedicalSystemUsers",
    "TokenExpirationMinutes": 60,
    "RefreshTokenExpirationDays": 7
  },
  "CacheSettings": {
    "DefaultExpirationMinutes": 30,
    "UserSessionExpirationMinutes": 60,
    "OrganizationSettingsExpirationHours": 24
  },
  "FileStorage": {
    "StorageType": "Local",  // or "Network" or "AzureBlob" (V2.0)
    "LocalPath": "C:\\MedicalSystemFiles",
    "MaxFileSizeMB": 10,
    "AllowedExtensions": [".jpg", ".jpeg", ".png", ".pdf", ".docx"]
  },
  "SecuritySettings": {
    "MaxFailedLoginAttempts": 5,
    "AccountLockoutMinutes": 30,
    "PasswordMinLength": 8,
    "PasswordRequireUppercase": true,
    "PasswordRequireDigit": true,
    "PasswordRequireSpecialChar": true
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    },
    "Serilog": {
      "MinimumLevel": "Information",
      "WriteTo": [
        { "Name": "Console" },
        { "Name": "File", "Args": { "path": "Logs/log-.txt", "rollingInterval": "Day" } }
      ]
    }
  },
  "AllowedHosts": "*"
}
```

#### 10.3.2 Frontend Configuration (environment.ts)

```typescript
export const environment = {
  production: false,
  apiUrl: 'https://localhost:7001/api',
  apiTimeout: 30000,
  cacheExpirationMinutes: 30,
  dateFormat: 'dd/MM/yyyy',
  timeFormat: 'HH:mm',
  currency: 'USD',
  paginationPageSize: 10,
  paginationPageSizeOptions: [10, 25, 50, 100],
  maxFileUploadSizeMB: 10,
  allowedFileTypes: ['.jpg', '.jpeg', '.png', '.pdf'],
  sessionTimeoutMinutes: 60,
  enableDebugMode: true
};
```

### 10.4 Database Configuration & Strategy

#### 10.4.1 Connection String Format

**SQL Server**:
```
Server=localhost;Database=MedicalSystemDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True;
```

**PostgreSQL**:
```
Host=localhost;Database=medicalsystemdb;Username=postgres;Password=YourPassword
```

#### 10.4.2 Entity Framework Core Configuration Strategy

**Global Query Filter for Multi-Tenancy**:
```
Applied to all entities with OrganizationId property
Automatically filters all queries by current tenant context
Cannot be bypassed without explicit IgnoreQueryFilters() call
```

**Audit Fields on All Entities**:
```
CreatedAt: DateTime (automatically set on insert)
UpdatedAt: DateTime (automatically set on update)
CreatedBy: int? (user ID who created)
UpdatedBy: int? (user ID who last updated)
```

**Soft Delete Strategy**:
```
IsActive/IsDeleted boolean field on entities
Never physically delete records (except for GDPR compliance requests)
Filter out deleted records in queries
```

### 10.5 API Endpoint Structure

**Base URL**: `https://api.medicalsystem.com/api` (or configured domain)

**Endpoint Naming Convention**:
```
GET    /api/{resource}                    # Get all (paginated)
GET    /api/{resource}/{id}               # Get by ID
POST   /api/{resource}                    # Create
PUT    /api/{resource}/{id}               # Update (full)
PATCH  /api/{resource}/{id}               # Update (partial)
DELETE /api/{resource}/{id}               # Delete (soft)
GET    /api/{resource}/search?query={}    # Search
GET    /api/{resource}/{id}/{sub-resource}  # Nested resources
```

### 10.6 Manual Mapping Strategy

**Reason for Manual Mapping**: Avoids AutoMapper dependency, provides explicit control, better performance

**Mapping Location**: `Application Layer -> Mappers folder`

**Example Mapper Pattern**:
```
PatientMapper.cs:
- ToDto(Patient entity) -> PatientDto
- ToEntity(CreatePatientDto dto) -> Patient
- ToEntity(UpdatePatientDto dto, Patient existing) -> Patient
- ToDtoList(List<Patient> entities) -> List<PatientDto>
```

### 10.7 FluentValidation Strategy

**Validator Location**: `Application Layer -> Validators folder`

**Validation Rules**:
- Required fields
- Length constraints
- Format validation (email, phone)
- Business rules
- Cross-field validation

**Example Validator**:
```
CreatePatientDtoValidator:
- FirstName: Required, MaxLength 100
- LastName: Required, MaxLength 100
- DateOfBirth: Required, Must be past date
- Phone: Required, Valid phone format
- Email: Optional, Valid email format
```

### 10.8 Error Handling Strategy

**Exception Hierarchy**:
1. BusinessException - Business rule violations
2. NotFoundException - Resource not found
3. ValidationException - Input validation failures
4. UnauthorizedException - Authentication failures
5. ForbiddenException - Authorization failures

**Error Response Format**:
```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "Email",
      "message": "Email is required"
    }
  ],
  "timestamp": "2026-01-10T10:30:00Z"
}
```

### 10.9 Deployment Considerations

#### Single Server Deployment (Small Clinics):
- Windows Server 2019+ or Ubuntu 22.04+
- 8GB RAM minimum
- 4 CPU cores
- 100GB SSD storage
- IIS or Nginx
- SQL Server Express or PostgreSQL
- Redis (optional for small deployments)

#### Two-Tier Deployment (Medium Clinics):
- Application Server: 8GB RAM, 4 cores
- Database Server: 16GB RAM, 4 cores
- Separate Redis server (optional)

#### Scaled Deployment (Hospitals / SaaS):
- Multiple application servers behind load balancer
- Dedicated database server with replicas
- Dedicated Redis cluster
- Separate file storage server

---

**End of System Design Document**

This comprehensive document covers all aspects of the Medical Management System design for Version 1.0, focusing on monolithic architecture with clean code principles, manual mapping, FluentValidation, and support for both SaaS multi-tenant and private deployment models without external service dependencies in the initial version.
