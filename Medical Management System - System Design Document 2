# Medical Management System - System Design Document

## 1. System Overview

### 1.1 Purpose
The Medical Management System is a comprehensive healthcare platform designed to serve three distinct operational models:
- **Personal Clinic**: Single doctor practice
- **Multi-Doctor Clinic**: Clinic with multiple doctors
- **Hospital**: Large facility with multiple departments and doctors

### 1.2 Scope
The system manages patient records, appointments, medical history, prescriptions, billing, and reporting across different organizational structures while maintaining data privacy and security compliance.

### 1.3 Key Features
- Patient registration and management
- Appointment scheduling
- Electronic Medical Records (EMR)
- Prescription management
- Billing and invoicing
- Reporting and analytics
- Multi-tenant architecture
- Role-based access control

---

## 2. Stakeholders & User Types

### 2.1 Primary Stakeholders
- **Healthcare Providers**: Doctors, Nurses, Medical Staff
- **Patients**: Individuals seeking medical care
- **Administrative Staff**: Receptionists, Billing personnel
- **System Administrators**: IT staff managing the platform

### 2.2 User Roles

```mermaid
graph TD
    A[Users] --> B[Super Admin]
    A --> C[Organization Admin]
    A --> D[Doctor]
    A --> E[Nurse]
    A --> F[Receptionist]
    A --> G[Billing Staff]
    A --> H[Patient]
    
    B --> I[System Configuration]
    C --> J[Organization Management]
    D --> K[Medical Services]
    E --> L[Patient Care]
    F --> M[Appointment Management]
    G --> N[Financial Operations]
    H --> O[Self-Service Portal]
```

### 2.3 Role Permissions

| Role | Permissions |
|------|-------------|
| Super Admin | Full system access, multi-tenant management |
| Organization Admin | Organization settings, user management, reporting |
| Doctor | Patient records, prescriptions, appointments, medical history |
| Nurse | Patient vitals, basic records, assist doctor |
| Receptionist | Appointments, patient registration, basic info |
| Billing Staff | Invoicing, payments, financial reports |
| Patient | View own records, book appointments, view prescriptions |

---

## 3. Functional Requirements

### 3.1 Organization Management
- **FR-ORG-001**: System shall support multiple organization types (Personal Clinic, Multi-Doctor Clinic, Hospital)
- **FR-ORG-002**: Each organization shall have configurable settings (working hours, departments, services)
- **FR-ORG-003**: Hospital type shall support department hierarchy
- **FR-ORG-004**: System shall allow organization-specific branding

### 3.2 User Management
- **FR-USER-001**: System shall support role-based access control
- **FR-USER-002**: Users shall be assigned to specific organizations
- **FR-USER-003**: Doctors can work across multiple organizations
- **FR-USER-004**: Authentication shall support SSO and 2FA

### 3.3 Patient Management
- **FR-PAT-001**: Register new patients with complete demographics
- **FR-PAT-002**: Maintain comprehensive medical history
- **FR-PAT-003**: Support patient search and filtering
- **FR-PAT-004**: Track patient visits and encounters
- **FR-PAT-005**: Manage patient documents and files

### 3.4 Appointment Management
- **FR-APT-001**: Schedule appointments with specific doctors
- **FR-APT-002**: Support recurring appointments
- **FR-APT-003**: Send automated reminders (SMS/Email)
- **FR-APT-004**: Handle appointment cancellations and rescheduling
- **FR-APT-005**: Show doctor availability calendar

### 3.5 Medical Records
- **FR-MED-001**: Create and update medical records per visit
- **FR-MED-002**: Record diagnosis using ICD-10 codes
- **FR-MED-003**: Create electronic prescriptions
- **FR-MED-004**: Track vital signs and lab results
- **FR-MED-005**: Support medical imaging integration

### 3.6 Billing & Invoicing
- **FR-BILL-001**: Generate invoices for services rendered
- **FR-BILL-002**: Support multiple payment methods
- **FR-BILL-003**: Track insurance claims
- **FR-BILL-004**: Generate financial reports
- **FR-BILL-005**: Handle partial payments and refunds

---

## 4. Non-Functional Requirements

### 4.1 Performance
- **NFR-PERF-001**: System shall support 1000+ concurrent users
- **NFR-PERF-002**: Page load time shall be < 2 seconds
- **NFR-PERF-003**: API response time shall be < 500ms for 95% of requests

### 4.2 Security
- **NFR-SEC-001**: All data shall be encrypted at rest and in transit (TLS 1.3)
- **NFR-SEC-002**: Comply with HIPAA/GDPR regulations
- **NFR-SEC-003**: Implement audit logging for all data access
- **NFR-SEC-004**: Support data isolation between organizations

### 4.3 Scalability
- **NFR-SCALE-001**: Horizontal scaling capability
- **NFR-SCALE-002**: Support 100+ organizations
- **NFR-SCALE-003**: Database partitioning by organization

### 4.4 Availability
- **NFR-AVAIL-001**: System uptime of 99.9%
- **NFR-AVAIL-002**: Automated backup every 6 hours
- **NFR-AVAIL-003**: Disaster recovery plan with RPO < 1 hour

### 4.5 Usability
- **NFR-USE-001**: Responsive design for desktop, tablet, and mobile
- **NFR-USE-002**: Support multiple languages
- **NFR-USE-003**: Accessibility compliance (WCAG 2.1 Level AA)

---

## 5. User Scenarios

### 5.1 Scenario: Patient Booking Appointment

**Actor**: Patient
**Goal**: Book an appointment with a doctor

**Steps**:
1. Patient logs into the system
2. Selects desired clinic/hospital
3. Chooses specialty or specific doctor
4. Views available time slots
5. Selects preferred date and time
6. Confirms appointment
7. Receives confirmation via email/SMS

### 5.2 Scenario: Doctor Consultation

**Actor**: Doctor
**Goal**: Complete a patient consultation

**Steps**:
1. Doctor views daily appointment schedule
2. Calls patient from waiting room
3. Reviews patient medical history
4. Records vitals (delegated to nurse)
5. Documents symptoms and examination
6. Creates diagnosis
7. Prescribes medications
8. Orders lab tests if needed
9. Schedules follow-up appointment
10. Completes visit documentation

### 5.3 Scenario: Billing Process

**Actor**: Billing Staff
**Goal**: Generate and process invoice

**Steps**:
1. Views completed consultations
2. Generates invoice with service charges
3. Applies insurance coverage if applicable
4. Presents invoice to patient
5. Processes payment
6. Issues receipt
7. Updates financial records

---

## 6. High-Level Design (HLD)

### 6.1 System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        A[Web Application - Angular]
        B[Mobile App]
        C[Admin Portal]
    end
    
    subgraph "API Gateway"
        D[API Gateway / Load Balancer]
    end
    
    subgraph "Application Layer - .NET"
        E[Authentication Service]
        F[Organization Service]
        G[Patient Service]
        H[Appointment Service]
        I[Medical Records Service]
        J[Billing Service]
        K[Notification Service]
    end
    
    subgraph "Data Layer"
        L[(SQL Database)]
        M[(Document Store)]
        N[(Cache - Redis)]
    end
    
    subgraph "External Services"
        O[Email Service]
        P[SMS Gateway]
        Q[Payment Gateway]
        R[Cloud Storage]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    D --> F
    D --> G
    D --> H
    D --> I
    D --> J
    D --> K
    
    E --> L
    F --> L
    G --> L
    H --> L
    I --> L
    J --> L
    
    E --> N
    F --> N
    G --> N
    
    I --> M
    I --> R
    
    K --> O
    K --> P
    J --> Q
```

---

## 7. Entity Relationship Diagrams (ERD)

### 7.1 Core ERD

```mermaid
erDiagram
    ORGANIZATION ||--o{ DEPARTMENT : has
    ORGANIZATION ||--o{ USER : employs
    ORGANIZATION ||--o{ PATIENT : serves
    ORGANIZATION {
        int Id PK
        string Name
        string Type
        string Address
        string Phone
        string Email
        datetime CreatedAt
    }
    
    DEPARTMENT ||--o{ USER : contains
    DEPARTMENT {
        int Id PK
        int OrganizationId FK
        string Name
        string Description
    }
    
    USER ||--o{ APPOINTMENT : schedules
    USER ||--o{ MEDICAL_RECORD : creates
    USER {
        int Id PK
        int OrganizationId FK
        int DepartmentId FK
        string FirstName
        string LastName
        string Email
        string Role
        bool IsActive
    }
    
    PATIENT ||--o{ APPOINTMENT : books
    PATIENT ||--o{ MEDICAL_RECORD : has
    PATIENT ||--o{ INVOICE : receives
    PATIENT {
        int Id PK
        int OrganizationId FK
        string FirstName
        string LastName
        date DateOfBirth
        string Gender
        string Phone
        string Email
        string Address
        string BloodGroup
        string MedicalHistory
    }
    
    APPOINTMENT ||--|| MEDICAL_RECORD : generates
    APPOINTMENT {
        int Id PK
        int PatientId FK
        int DoctorId FK
        int OrganizationId FK
        datetime AppointmentDate
        int Duration
        string Status
        string Notes
    }
    
    MEDICAL_RECORD ||--o{ PRESCRIPTION : contains
    MEDICAL_RECORD ||--o{ LAB_TEST : orders
    MEDICAL_RECORD {
        int Id PK
        int PatientId FK
        int DoctorId FK
        int AppointmentId FK
        datetime VisitDate
        string ChiefComplaint
        string Diagnosis
        string Notes
        string VitalSigns
    }
    
    PRESCRIPTION ||--o{ PRESCRIPTION_ITEM : contains
    PRESCRIPTION {
        int Id PK
        int MedicalRecordId FK
        datetime PrescribedDate
        string Notes
    }
    
    PRESCRIPTION_ITEM {
        int Id PK
        int PrescriptionId FK
        string MedicationName
        string Dosage
        string Frequency
        int Duration
        string Instructions
    }
    
    LAB_TEST {
        int Id PK
        int MedicalRecordId FK
        string TestName
        string TestType
        datetime OrderedDate
        datetime ResultDate
        string Results
        string Status
    }
    
    MEDICAL_RECORD ||--|| INVOICE : generates
    INVOICE ||--o{ INVOICE_ITEM : contains
    INVOICE ||--o{ PAYMENT : receives
    INVOICE {
        int Id PK
        int PatientId FK
        int MedicalRecordId FK
        string InvoiceNumber
        datetime InvoiceDate
        decimal TotalAmount
        decimal PaidAmount
        string Status
    }
    
    INVOICE_ITEM {
        int Id PK
        int InvoiceId FK
        string ServiceName
        int Quantity
        decimal UnitPrice
        decimal TotalPrice
    }
    
    PAYMENT {
        int Id PK
        int InvoiceId FK
        datetime PaymentDate
        decimal Amount
        string PaymentMethod
        string TransactionId
    }
```

---

## 8. Sequence Diagrams

### 8.1 Patient Appointment Booking Flow

```mermaid
sequenceDiagram
    actor Patient
    participant WebApp
    participant API Gateway
    participant Auth Service
    participant Appointment Service
    participant Notification Service
    participant Database
    
    Patient->>WebApp: Access Booking Page
    WebApp->>API Gateway: Request Available Slots
    API Gateway->>Auth Service: Validate Token
    Auth Service-->>API Gateway: Token Valid
    API Gateway->>Appointment Service: Get Available Slots
    Appointment Service->>Database: Query Doctor Schedule
    Database-->>Appointment Service: Return Available Slots
    Appointment Service-->>API Gateway: Return Slots
    API Gateway-->>WebApp: Display Available Slots
    WebApp-->>Patient: Show Time Slots
    
    Patient->>WebApp: Select Slot & Confirm
    WebApp->>API Gateway: Create Appointment
    API Gateway->>Appointment Service: Process Booking
    Appointment Service->>Database: Save Appointment
    Database-->>Appointment Service: Appointment Saved
    Appointment Service->>Notification Service: Send Confirmation
    Notification Service-->>Patient: Email/SMS Confirmation
    Appointment Service-->>API Gateway: Booking Successful
    API Gateway-->>WebApp: Confirmation
    WebApp-->>Patient: Display Success Message
```

### 8.2 Doctor Consultation Flow

```mermaid
sequenceDiagram
    actor Doctor
    participant WebApp
    participant API Gateway
    participant Patient Service
    participant Medical Record Service
    participant Billing Service
    participant Database
    
    Doctor->>WebApp: View Patient List
    WebApp->>API Gateway: Get Today's Appointments
    API Gateway->>Patient Service: Fetch Appointments
    Patient Service->>Database: Query Appointments
    Database-->>Patient Service: Return List
    Patient Service-->>WebApp: Display Patient List
    
    Doctor->>WebApp: Select Patient
    WebApp->>API Gateway: Get Patient History
    API Gateway->>Medical Record Service: Fetch Medical History
    Medical Record Service->>Database: Query Records
    Database-->>Medical Record Service: Return History
    Medical Record Service-->>WebApp: Display History
    
    Doctor->>WebApp: Create Consultation Record
    Doctor->>WebApp: Add Diagnosis & Prescription
    WebApp->>API Gateway: Save Medical Record
    API Gateway->>Medical Record Service: Create Record
    Medical Record Service->>Database: Save Record
    Database-->>Medical Record Service: Record Saved
    
    Medical Record Service->>Billing Service: Generate Invoice
    Billing Service->>Database: Create Invoice
    Database-->>Billing Service: Invoice Created
    Billing Service-->>Medical Record Service: Invoice Generated
    Medical Record Service-->>WebApp: Consultation Complete
    WebApp-->>Doctor: Success Confirmation
```

### 8.3 Multi-Tenant Data Access Flow

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant API Gateway
    participant Auth Service
    participant Tenant Context
    participant Service Layer
    participant Database
    
    User->>Frontend: Login
    Frontend->>API Gateway: Authentication Request
    API Gateway->>Auth Service: Validate Credentials
    Auth Service->>Database: Check User & Organization
    Database-->>Auth Service: User Details + Org ID
    Auth Service-->>API Gateway: JWT Token (with Org ID)
    API Gateway-->>Frontend: Token with Claims
    
    User->>Frontend: Request Data
    Frontend->>API Gateway: API Call + JWT Token
    API Gateway->>Auth Service: Validate Token
    Auth Service-->>API Gateway: Extract Org ID from Token
    API Gateway->>Tenant Context: Set Organization Context
    Tenant Context->>Service Layer: Pass Request with Org ID
    Service Layer->>Database: Query with Org Filter
    Note over Database: WHERE OrganizationId = {OrgId}
    Database-->>Service Layer: Filtered Results
    Service Layer-->>Frontend: Return Data
```

---

## 9. Flow Charts

### 9.1 User Authentication Flow

```mermaid
flowchart TD
    A[Start: User Login] --> B{Valid Credentials?}
    B -->|No| C[Show Error Message]
    C --> D[Retry Count < 3?]
    D -->|Yes| A
    D -->|No| E[Lock Account]
    E --> F[End]
    
    B -->|Yes| G{2FA Enabled?}
    G -->|Yes| H[Send OTP]
    H --> I{Valid OTP?}
    I -->|No| C
    I -->|Yes| J[Generate JWT Token]
    
    G -->|No| J
    J --> K[Set Organization Context]
    K --> L[Load User Dashboard]
    L --> F
```

### 9.2 Appointment Booking Flow

```mermaid
flowchart TD
    A[Start: Book Appointment] --> B[Select Organization]
    B --> C{Organization Type?}
    
    C -->|Personal Clinic| D[Show Single Doctor]
    C -->|Multi-Doctor Clinic| E[Select Doctor from List]
    C -->|Hospital| F[Select Department]
    F --> G[Select Doctor]
    
    D --> H[Check Availability]
    E --> H
    G --> H
    
    H --> I{Slots Available?}
    I -->|No| J[Show Alternative Dates]
    J --> K{Accept Alternative?}
    K -->|No| L[End]
    K -->|Yes| H
    
    I -->|Yes| M[Select Time Slot]
    M --> N[Enter Patient Details]
    N --> O{Existing Patient?}
    O -->|No| P[Create Patient Record]
    O -->|Yes| Q[Link to Existing]
    
    P --> R[Confirm Booking]
    Q --> R
    R --> S[Create Appointment]
    S --> T[Send Confirmation]
    T --> U{Payment Required?}
    U -->|Yes| V[Process Payment]
    U -->|No| W[Complete]
    V --> W
    W --> L
```

### 9.3 Medical Record Creation Flow

```mermaid
flowchart TD
    A[Start: Patient Visit] --> B[Check-In Patient]
    B --> C[Nurse Records Vitals]
    C --> D[Update Waiting Queue]
    D --> E[Doctor Calls Patient]
    E --> F[Review Medical History]
    F --> G[Record Chief Complaint]
    G --> H[Physical Examination]
    H --> I[Enter Diagnosis]
    I --> J{Need Lab Tests?}
    
    J -->|Yes| K[Order Lab Tests]
    K --> L{Need Prescription?}
    
    J -->|No| L
    L -->|Yes| M[Create Prescription]
    M --> N[Add Medications]
    N --> O{Need Follow-Up?}
    
    L -->|No| O
    O -->|Yes| P[Schedule Follow-Up]
    O -->|No| Q[Generate Invoice]
    P --> Q
    
    Q --> R[Save Medical Record]
    R --> S[Print Documents]
    S --> T[Complete Visit]
    T --> U[End]
```

### 9.4 Billing Process Flow

```mermaid
flowchart TD
    A[Start: Generate Bill] --> B[Retrieve Medical Record]
    B --> C[Fetch Service Charges]
    C --> D[Add Consultation Fee]
    D --> E{Lab Tests Ordered?}
    E -->|Yes| F[Add Lab Charges]
    E -->|No| G{Procedures Done?}
    F --> G
    
    G -->|Yes| H[Add Procedure Charges]
    G -->|No| I[Calculate Total]
    H --> I
    
    I --> J{Insurance Applicable?}
    J -->|Yes| K[Verify Coverage]
    K --> L[Apply Insurance Discount]
    L --> M[Calculate Patient Share]
    
    J -->|No| M
    M --> N[Generate Invoice]
    N --> O[Display to Patient]
    O --> P{Payment Method?}
    
    P -->|Cash| Q[Accept Cash]
    P -->|Card| R[Process Card Payment]
    P -->|Insurance| S[Submit Insurance Claim]
    
    Q --> T[Issue Receipt]
    R --> T
    S --> U{Approved?}
    U -->|Yes| T
    U -->|No| V[Patient Pays Difference]
    V --> T
    
    T --> W[Update Financial Records]
    W --> X[Close Invoice]
    X --> Y[End]
```

---

## 10. Technical Implementation Details

### 10.1 Technology Stack

#### Frontend (Angular)
- **Framework**: Angular 17+
- **State Management**: NgRx or Akita
- **UI Framework**: Angular Material or PrimeNG
- **HTTP Client**: Angular HttpClient with Interceptors
- **Authentication**: JWT with HTTP Interceptors
- **Routing**: Angular Router with Guards
- **Forms**: Reactive Forms with Validation

#### Backend (.NET)
- **Framework**: ASP.NET Core 8.0 Web API
- **Architecture**: Clean Architecture / Onion Architecture
- **ORM**: Entity Framework Core 8.0
- **Database**: SQL Server 2022 / PostgreSQL
- **Caching**: Redis
- **Authentication**: JWT Bearer Token with IdentityServer or Auth0
- **API Documentation**: Swagger/OpenAPI
- **Logging**: Serilog
- **Mapping**: AutoMapper

### 10.2 Database Schema (SQL Server)

```sql
-- Organization Table
CREATE TABLE Organizations (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(200) NOT NULL,
    Type NVARCHAR(50) NOT NULL CHECK (Type IN ('PersonalClinic', 'MultiDoctorClinic', 'Hospital')),
    Address NVARCHAR(500),
    Phone NVARCHAR(20),
    Email NVARCHAR(100),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    IsActive BIT DEFAULT 1
);

-- Users Table
CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY(1,1),
    OrganizationId INT NOT NULL FOREIGN KEY REFERENCES Organizations(Id),
    DepartmentId INT NULL FOREIGN KEY REFERENCES Departments(Id),
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(100) UNIQUE NOT NULL,
    PasswordHash NVARCHAR(255) NOT NULL,
    Role NVARCHAR(50) NOT NULL,
    Phone NVARCHAR(20),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    IsActive BIT DEFAULT 1,
    INDEX IX_Users_OrganizationId (OrganizationId),
    INDEX IX_Users_Email (Email)
);

-- Patients Table
CREATE TABLE Patients (
    Id INT PRIMARY KEY IDENTITY(1,1),
    OrganizationId INT NOT NULL FOREIGN KEY REFERENCES Organizations(Id),
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    DateOfBirth DATE NOT NULL,
    Gender NVARCHAR(10),
    Phone NVARCHAR(20),
    Email NVARCHAR(100),
    Address NVARCHAR(500),
    BloodGroup NVARCHAR(10),
    EmergencyContact NVARCHAR(200),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    INDEX IX_Patients_OrganizationId (OrganizationId),
    INDEX IX_Patients_Email (Email)
);

-- Appointments Table
CREATE TABLE Appointments (
    Id INT PRIMARY KEY IDENTITY(1,1),
    PatientId INT NOT NULL FOREIGN KEY REFERENCES Patients(Id),
    DoctorId INT NOT NULL FOREIGN KEY REFERENCES Users(Id),
    OrganizationId INT NOT NULL FOREIGN KEY REFERENCES Organizations(Id),
    AppointmentDate DATETIME2 NOT NULL,
    Duration INT DEFAULT 30,
    Status NVARCHAR(50) DEFAULT 'Scheduled',
    Notes NVARCHAR(1000),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    INDEX IX_Appointments_Date (AppointmentDate),
    INDEX IX_Appointments_Doctor (DoctorId, AppointmentDate),
    INDEX IX_Appointments_Patient (PatientId)
);
```

### 10.3 .NET Project Structure

```
MedicalSystem.Solution/
│
├── MedicalSystem.API/                    # Web API Project
│   ├── Controllers/
│   ├── Middleware/
│   ├── Filters/
│   └── Program.cs
│
├── MedicalSystem.Application/            # Business Logic
│   ├── Services/
│   │   ├── PatientService.cs
│   │   ├── AppointmentService.cs
│   │   └── MedicalRecordService.cs
│   ├── DTOs/
│   ├── Interfaces/
│   └── Validators/
│
├── MedicalSystem.Domain/                 # Domain Models
│   ├── Entities/
│   │   ├── Organization.cs
│   │   ├── Patient.cs
│   │   ├── Appointment.cs
│   │   └── MedicalRecord.cs
│   ├── Enums/
│   └── ValueObjects/
│
├── MedicalSystem.Infrastructure/         # Data Access
│   ├── Data/
│   │   ├── ApplicationDbContext.cs
│   │   └── Configurations/
│   ├── Repositories/
│   ├── Migrations/
│   └── Services/
│
└── MedicalSystem.Shared/                 # Shared Resources
    ├── Constants/
    ├── Exceptions/
    └── Helpers/
```

### 10.4 Angular Project Structure

```
medical-system-app/
│
├── src/
│   ├── app/
│   │   ├── core/                        # Singleton Services
│   │   │   ├── services/
│   │   │   │   ├── auth.service.ts
│   │   │   │   └── api.service.ts
│   │   │   ├── guards/
│   │   │   ├── interceptors/
│   │   │   └── models/
│   │   │
│   │   ├── shared/                      # Shared Components
│   │   │   ├── components/
│   │   │   ├── directives/
│   │   │   └── pipes/
│   │   │
│   │   ├── features/                    # Feature Modules
│   │   │   ├── patients/
│   │   │   │   ├── patient-list/
│   │   │   │   ├── patient-detail/
│   │   │   │   └── patients.module.ts
│   │   │   ├── appointments/
│   │   │   ├── medical-records/
│   │   │   └── billing/
│   │   │
│   │   ├── layout/                      # Layout Components
│   │   │   ├── header/
│   │   │   ├── sidebar/
│   │   │   └── footer/
│   │   │
│   │   └── app-routing.module.ts
│   │
│   └── environments/
```

### 10.5 Key API Endpoints

```
Authentication:
POST   /api/auth/login
POST   /api/auth/register
POST   /api/auth/refresh-token
POST   /api/auth/logout

Organizations:
GET    /api/organizations
GET    /api/organizations/{id}
POST   /api/organizations
PUT    /api/organizations/{id}

Patients:
GET    /api/patients
GET    /api/patients/{id}
POST   /api/patients
PUT    /api/patients/{id}
GET    /api/patients/{id}/history

Appointments:
GET    /api/appointments
GET    /api/appointments/{id}
POST   /api/appointments
PUT    /api/appointments/{id}
DELETE /api/appointments/{id}
GET    /api/appointments/available-slots

Medical Records:
GET    /api/medical-records/{patientId}
POST   /api/medical-records
GET    /api/medical-records/{id}
PUT    /api/medical-records/{id}

Prescriptions:
GET    /api/prescriptions/{medicalRecordId}
POST   /api/prescriptions

Billing:
GET    /api/invoices
GET    /api/invoices/{id}
POST   /api/invoices
POST   /api/payments
```

### 10.6 Multi-Tenancy Implementation

```csharp
// Tenant Context Service
public class TenantContext
{
    public int OrganizationId { get; set; }
    public string OrganizationType { get; set; }
}

// Middleware to Extract Tenant
public class TenantMiddleware
{
    private readonly RequestDelegate _next;
    
    public async Task InvokeAsync(HttpContext context, TenantContext tenantContext)
    {
        var token = context.Request.Headers["Authorization"].ToString();
        var claims = ExtractClaimsFromToken(token);
        
        tenantContext.OrganizationId = int.Parse(claims["OrganizationId"]);
        tenantContext.OrganizationType = claims["OrganizationType"];
        
        await _next(context);
    }
}

// Global Query Filter in DbContext
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Patient>()
        .HasQueryFilter(p => p.OrganizationId == _tenantContext.OrganizationId);
    
    modelBuilder.Entity<Appointment>()
        .HasQueryFilter(a => a.OrganizationId == _tenantContext.OrganizationId);
}
```

### 10.7 Security Implementation

```typescript
// Angular HTTP Interceptor
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`,
          'X-Organization-Id': this.authService.getOrganizationId()
        }
      });
    }
    
    return next.handle(req);
  }
}
```

### 10.8 Caching Strategy

```csharp
// Redis Caching in .NET
public class CacheService : ICacheService
{
    private readonly IDistributedCache _cache;
    
    public async Task<T> GetOrSetAsync<T>(
        string key, 
        Func
