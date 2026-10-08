# TechCare Backend

TechCare is a healthcare platform that connects patients with doctors, nurses, pharmacies, laboratories, and blood donors through a secure, role-aware backend.

The backend is the authoritative source for authentication, authorization, provider verification, bookings, medical records, pricing, payments, notifications, ratings, complaints, location-based functionality, search, and administrative operations.

> **Architecture principle:** The frontend communicates intent; the backend decides what is allowed and what is correct; the database protects data integrity.

---

## 1. Backend Goals

The backend must be:

- Secure
- Modular
- Maintainable
- Testable
- Scalable
- Consistent
- API-first
- Easy for five developers to work on in parallel

The current project should favor a **modular monolith** over premature microservices.

```text
Frontend
   ↓
REST API
   ↓
ASP.NET Core
   ├── API / Controllers
   ├── Application / Use Cases
   ├── Domain / Business Rules
   └── Infrastructure
          ├── Database
          ├── File Storage
          ├── Notifications
          ├── Payments
          ├── Geo Services
          └── Other Integrations
```

---

## 2. Backend Technology

Primary platform:

- ASP.NET Core Web API
- C#
- REST
- JSON

Recommended supporting technologies:

- Entity Framework Core
- Relational SQL database
- JWT authentication
- Refresh-token/session mechanism
- ASP.NET Core Authorization
- Dependency Injection
- OpenAPI / Swagger
- Automated testing
- Docker for reproducible development/deployment where adopted

Exact framework/package versions and infrastructure providers should come from the actual solution configuration rather than being hardcoded in this README.

---

## 3. Recommended Repository Structure

```text
backend/
│
├── src/
│   ├── TechCare.API/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Filters/
│   │   ├── Extensions/
│   │   ├── Configuration/
│   │   ├── Program.cs
│   │   └── appsettings*.json
│   │
│   ├── TechCare.Application/
│   │   ├── Common/
│   │   ├── DTOs/
│   │   ├── Interfaces/
│   │   ├── Validators/
│   │   └── Features/
│   │
│   ├── TechCare.Domain/
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Enums/
│   │   ├── Exceptions/
│   │   ├── Rules/
│   │   └── Events/
│   │
│   └── TechCare.Infrastructure/
│       ├── Persistence/
│       │   ├── DbContext/
│       │   ├── Configurations/
│       │   ├── Migrations/
│       │   └── Seed/
│       ├── Identity/
│       ├── Services/
│       ├── Storage/
│       ├── Notifications/
│       ├── Payments/
│       ├── Location/
│       └── Search/
│
├── tests/
│   ├── TechCare.UnitTests/
│   ├── TechCare.IntegrationTests/
│   └── TechCare.API.Tests/
│
├── docs/
├── Dockerfile
├── docker-compose.yml
└── README.md
```

Names can be adjusted to match the actual solution. Responsibilities should remain separated.

---

## 4. Layer Responsibilities

### API

Responsible for:

- HTTP endpoints
- Routing
- Authentication integration
- Authorization integration
- Model binding
- Response contracts
- HTTP status codes
- API-level validation coordination
- Global exception middleware integration

Controllers should be thin.

### Application

Responsible for:

- Use cases
- DTO orchestration
- Business workflow coordination
- Validation coordination
- Authorization checks that belong to the use case
- Transactions where appropriate
- Calling domain rules and infrastructure abstractions

### Domain

Responsible for:

- Core entities
- Invariants
- State transitions
- Business rules
- Value objects
- Domain exceptions
- Domain events where useful

The domain should not depend on HTTP, controllers, UI, or concrete infrastructure services.

### Infrastructure

Responsible for concrete implementations of:

- Database persistence
- EF Core
- File storage
- Email/SMS
- Notifications
- Payment provider integration
- Geolocation services
- Search infrastructure
- Other external integrations

---

## 5. Dependency Direction

Preferred dependency direction:

```text
API
 ↓
Application
 ↓
Domain

Infrastructure
 └── implements abstractions required by Application/Domain
```

Avoid:

```text
Domain → API
Domain → UI
Domain → Controller
Domain → concrete Infrastructure
```

---

# 6. Core Modules

```text
Authentication
Authorization
User Management
Patient
Doctor
Nurse
Pharmacy
Laboratory
Blood Donor
Provider Verification
Booking / Appointment
Medical Records / Consultation
Payment / Financial
Notifications
Ratings
Complaints
Location / Geo
Search / Discovery
Admin
```

Each module must have a clear responsibility and a clear ownership/access boundary.

---

# 7. Authentication

Authentication is centralized across all roles.

Capabilities:

- Registration entry
- Login
- Contact verification
- OTP generation/verification
- OTP resend/cooldown
- Forgot password
- Password reset
- Password hashing
- Access token creation
- Refresh-token/session management
- Logout/session invalidation where supported

General registration:

```text
Create Account
   ↓
Verify Contact
   ↓
Create/Complete Role Profile
   ↓
Account / Provider Status
```

Login:

```text
Credentials
   ↓
Validate Identity
   ↓
Check Account Status
   ↓
Issue Access Token
   +
Refresh Token / Session
```

---

# 8. Password and OTP Security

Passwords:

- Never store plain-text passwords.
- Never return passwords in API responses.
- Never log passwords.
- Never put passwords in URLs.

OTP:

- Short expiration
- Limited attempts
- Resend cooldown
- Rate limiting
- Secure random generation
- No OTP values in normal logs
- Expired/invalid code handling

Authentication endpoints are high-risk endpoints and should be protected against brute force.

---

# 9. Authorization

Roles:

```text
Patient
Doctor
Nurse
Pharmacy / Pharmacist
Laboratory
Blood Donor
Admin
```

Use roles and/or policies for coarse and fine-grained access.

Examples:

```text
Patient
 └── Manage own patient data

Doctor
 └── Manage own professional services

Nurse
 └── Manage own nursing services

Admin
 └── Verify providers and manage platform operations
```

The frontend can hide UI elements for convenience, but **only the backend provides security**.

---

# 10. Ownership / IDOR Protection

Every protected object must be checked against the current user's permissions and/or ownership.

Never assume this is safe:

```text
GET /api/v1/patients/{id}/records
```

because the requester is authenticated.

The backend must explicitly verify:

```text
Current user
      ↓
Authorized relationship
      ↓
Requested resource
```

This applies especially to:

- Patients
- Medical records
- Bookings
- Provider profiles
- Pharmacy branches
- Documents
- Payments
- Complaints

---

# 11. User and Role Profiles

Keep core account identity separate from role-specific profile data where appropriate.

```text
User
 ├── Authentication
 ├── Contact
 ├── Status
 └── Role/Profile
       ├── PatientProfile
       ├── DoctorProfile
       ├── NurseProfile
       ├── PharmacistProfile
       ├── LaboratoryProfile
       └── BloodDonorProfile
```

For pharmacy, the account holder and business entity should be modeled distinctly:

```text
Pharmacist / Account
       ↓
Pharmacy
       ↓
Branch
       ↓
Medicine Availability / Inventory
```

---

# 12. Patient Module

Responsibilities:

- Patient profile
- Personal information
- Medical profile
- Emergency contact
- Location
- Bookings
- Medical records access
- Notifications
- Ratings
- Complaints
- Payment/transaction history where applicable

Patient access is primarily owner-scoped.

---

# 13. Doctor Module

Responsibilities:

- Doctor profile
- Qualifications
- Specialty/sub-specialty
- Professional license
- Experience
- Services
- Pricing
- Availability
- Location
- Home visits where applicable
- Booking requests
- Appointments
- Authorized consultation/medical record operations
- Ratings
- Complaints
- Financial operations
- Verification status

Only approved/active providers should be able to offer services.

---

# 14. Nurse Module

Responsibilities:

- Nurse profile
- Nursing qualification
- Professional license
- Skills
- Services
- Pricing
- Location
- Service radius
- Availability
- Home visits
- Booking requests
- Completed services
- Ratings
- Complaints
- Financial operations where applicable
- Verification status

Nurse permissions must not automatically inherit doctor-only clinical capabilities.

---

# 15. Pharmacy Module

Separate:

```text
Pharmacist
   +
Pharmacy
   +
Branch
   +
Medicine Inventory
```

Pharmacy responsibilities may include:

### Pharmacist

- Identity
- Qualification
- Professional license
- Verification

### Pharmacy

- Name
- Type
- License
- Contact details
- Status

### Branch

- Name
- Phone
- Address
- Coordinates
- Operating hours
- Active/inactive state

### Inventory

- Medicine
- Branch
- Availability
- Quantity where supported
- Inventory status

Registration creates the pharmacy and initial branch; inventory management belongs to later operational features.

---

# 16. Laboratory Module

Responsibilities:

- Laboratory profile
- License
- Services
- Tests
- Pricing where applicable
- Location
- Working hours
- Service coverage
- Verification documents
- Verification status

Future capabilities such as home sample collection and result delivery must use explicit authorization and protected medical data access.

---

# 17. Blood Donor Module

Responsibilities:

- Donor profile
- Blood type
- Rh factor
- Location
- Availability
- Donation-related information
- Notification preferences
- Matching participation
- Privacy controls

The backend must not treat frontend registration as medical eligibility approval.

---

# 18. Provider Verification

Provider verification is separate from authentication.

Typical professional providers:

```text
Doctor
Nurse
Pharmacist / Pharmacy
Laboratory
```

Lifecycle:

```text
Registration Submitted
        ↓
PENDING_VERIFICATION
        ↓
Admin Review
        ↓
APPROVED
or
REJECTED
or
ADDITIONAL_INFORMATION_REQUIRED
        ↓
Possible SUSPENDED state when needed
```

Provider search visibility and service activation must depend on authoritative status.

---

# 19. Verification Documents

Documents may include:

- National ID
- Qualification/graduation certificate
- Professional license
- Pharmacy/laboratory license
- Supporting documents

Security requirements:

- Validate type
- Validate size
- Validate content where possible
- Store securely
- Do not expose public file URLs
- Enforce ownership/role access
- Audit sensitive actions where appropriate

Sensitive documents must not be stored as unrestricted static assets.

---

# 20. Booking / Appointment Module

Core flow:

```text
Patient Search
    ↓
Select Provider
    ↓
Select Service
    ↓
Choose Location
    ↓
Check Availability
    ↓
Choose Slot
    ↓
Calculate Price
    ↓
Create Booking Request
    ↓
PENDING
    ↓
ACCEPTED / REJECTED / EXPIRED
    ↓
SCHEDULED
    ↓
IN_PROGRESS
    ↓
COMPLETED
```

Possible states:

```text
PENDING
ACCEPTED
REJECTED
EXPIRED
SCHEDULED
IN_PROGRESS
COMPLETED
CANCELLED
```

The actual enum values must be defined once and reused everywhere.

---

# 21. Booking Concurrency

Booking is a concurrency-sensitive operation.

Potential race:

```text
Patient A ──┐
            ├── same slot
Patient B ──┘
```

The backend must guarantee that an exclusive slot cannot be incorrectly booked twice.

Use appropriate database transactions, constraints, locking, and/or concurrency mechanisms.

Frontend availability checks are not authoritative.

---

# 22. Booking Expiration

Pending provider requests may expire after a configured response window.

```text
PENDING
   ↓
response window elapsed
   ↓
EXPIRED
```

The backend is authoritative for expiration.

Do not trust the browser's local countdown.

---

# 23. Price Calculation

Authoritative pricing is backend-owned.

General model:

```text
Base Service Price
       +
Travel / Distance Fee
       +
Additional Fees
       =
Server-Validated Total
```

Possible travel calculation:

```text
Distance × Configured Price Per KM = Travel Fee
```

Never trust:

```text
clientTotal
clientPrice
clientProviderBalance
```

The backend must calculate or validate financial values.

---

# 24. Location / Geo Module

Responsibilities:

- Address
- Governorate
- City
- Latitude
- Longitude
- Service radius
- Distance calculation
- Coverage validation
- Nearby search

Validate coordinate ranges:

```text
Latitude  : -90 to 90
Longitude : -180 to 180
```

Do not trust arbitrary client coordinates without validation.

---

# 25. Search / Discovery

Search may include:

```text
Doctors
Nurses
Pharmacies
Medicines
Laboratories
Blood Donation
```

General flow:

```text
Search
 ↓
Validate Filters
 ↓
Apply Status / Visibility Rules
 ↓
Apply Location
 ↓
Apply Specialty / Service / Price / Availability
 ↓
Sort
 ↓
Paginate
 ↓
Return DTOs
```

Public search should not expose unapproved or suspended providers when the business rules say they are hidden.

---

# 26. Pagination

Use pagination for large collections:

- Search results
- Notifications
- Bookings
- Complaints
- Ratings
- Transactions
- Admin tables
- Inventory

Example:

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalCount": 120,
  "totalPages": 6
}
```

Use one response convention across the API.

---

# 27. Medical Records / Consultation

Medical data is highly sensitive.

A consultation may contain:

- Symptoms/complaint
- Diagnosis
- Treatment
- Medication information
- Notes
- Date
- Patient
- Booking/encounter

Flow:

```text
Completed / Authorized Encounter
       ↓
Consultation
       ↓
Medical Record Entry
       ↓
Authorized Retrieval
```

Access must be explicitly authorized.

The frontend must never be able to gain access simply by changing a URL or record ID.

---

# 28. Financial Module

Responsibilities may include:

- Service price
- Travel fee
- Additional fee
- Final amount
- Payment status
- Transactions
- Provider earnings
- Withdrawals
- Refund/cancellation rules where supported

Example:

```text
Booking
  ↓
Server Price Calculation
  ↓
Payment
  ↓
Payment Confirmation
  ↓
Financial State Update
```

Financial transitions should be transactional.

---

# 29. Payment Security

Never trust client-controlled:

```text
Amount
Total
Payment Status
Provider Balance
Refund Amount
```

Recalculate/validate on the backend.

External payment callbacks/webhooks must be authenticated or verified according to the selected provider's integration requirements.

---

# 30. Notifications

Notifications may be triggered by events such as:

- Account/contact verification
- Booking created
- Booking accepted/rejected/expired
- Appointment reminders
- Provider verification updates
- Payment updates
- Complaint updates
- Blood donation requests

General flow:

```text
Business Event
      ↓
Notification Service
      ↓
Notification Record
      ↓
Channel Delivery
      ↓
User
```

Possible channels:

```text
In-App
Email
SMS
```

Only configured channels should be implemented.

---

# 31. Ratings

Typical rules:

```text
Eligible Completed Booking
        ↓
Submit Rating
        ↓
Store Score + Optional Comment
```

The backend must enforce:

- Eligibility
- Ownership of the booking
- Duplicate-rating rules
- Valid rating range

---

# 32. Complaints

Complaint data may include:

- Creator
- Related provider/user
- Related booking
- Description
- Attachments where supported
- Status
- Admin handling
- Resolution

Possible statuses:

```text
OPEN
UNDER_REVIEW
RESOLVED
REJECTED
```

The actual enum must be centralized.

---

# 33. Admin Module

Admin capabilities may include:

```text
User Management
Provider Verification
Provider Suspension
Complaint Management
Rating Moderation
Booking Monitoring
Payment Monitoring
Notification Operations
Reports
Platform Configuration
```

All admin endpoints must be protected server-side.

---

# 34. Database Design

The backend should use a relational model with clear relationships.

Potential entities:

```text
User
Role
Permission
Session / RefreshToken
Otp
PatientProfile
DoctorProfile
NurseProfile
PharmacistProfile
Pharmacy
PharmacyBranch
LaboratoryProfile
BloodDonorProfile
ProviderVerification
VerificationDocument
Service
ProviderService
Availability
Booking
Appointment
Consultation
MedicalRecord
Payment
Transaction
Notification
Rating
Complaint
Location
Medicine
InventoryItem / BranchMedicine
AuditLog
```

The final model should reflect the actual approved requirements.

Do not create entities simply to make the architecture look complex.

---

# 35. DTOs

Do not expose persistence entities directly through the public API.

Use dedicated request and response DTOs.

```text
Entity
  ↓
Application Mapping
  ↓
Response DTO
  ↓
JSON
```

Examples:

```text
CreateBookingRequest
DoctorSummaryDto
ProviderVerificationDto
PatientProfileDto
```

Requests should only accept fields that the current operation is allowed to change.

---

# 36. Mass Assignment Protection

Never accept arbitrary entity properties from clients.

For example, a client must not be able to submit:

```json
{
  "role": "Admin",
  "isVerified": true,
  "balance": 999999
}
```

Use explicit DTOs and server-side rules.

---

# 37. Validation

Backend validation is mandatory.

Validate:

```text
Required fields
Formats
Lengths
Ranges
Dates
Relationships
Ownership
Business rules
State transitions
File metadata
Coordinates
```

Examples:

```text
Provider must be approved before offering services.
Service must belong to provider.
Booking slot must be valid and available.
License must be unique.
User can only modify allowed resources.
```

---

# 38. Business Rules and State Machines

Important state transitions must be controlled by the backend.

Example:

```text
PENDING
 ├── ACCEPTED
 ├── REJECTED
 ├── EXPIRED
 └── CANCELLED
```

Invalid transitions should be rejected.

Useful domain/application methods can include:

```text
CanAcceptBooking()
CanCancelBooking()
CanRateBooking()
CanAccessMedicalRecord()
CanOfferService()
CanAppearInSearch()
```

---

# 39. Database Integrity

Use database constraints where they matter.

Examples:

- Unique email where required
- Unique professional license
- Unique verification identifiers where required
- Foreign key constraints
- Booking integrity
- Transaction consistency

Application checks alone are not enough when concurrent requests are possible.

---

# 40. Transactions

Use transactions for operations that must be atomic.

Example:

```text
Accept Booking
   +
Create Appointment
   +
Update Booking Status
```

or:

```text
Confirm Payment
   +
Create Transaction
   +
Update Financial State
```

Avoid holding long-running database transactions across slow external calls.

---

# 41. Idempotency

Design retry-sensitive operations carefully.

Especially:

```text
Payment callbacks
Webhooks
Registration submission
Booking creation
Notification retries
File finalization
```

A repeated request should not accidentally create duplicate business effects.

---

# 42. Time and Dates

Use a consistent timezone strategy.

A common approach:

```text
Persist timestamps in UTC
        ↓
Convert for display/business context
```

Do not use browser clock values as the source of truth for expiration.

Booking and OTP expiration must use server-side timestamps.

---

# 43. File Storage

Secure file flow:

```text
Upload
  ↓
Validate
  ↓
Secure Storage
  ↓
Persist Metadata
  ↓
Authorized Retrieval
```

Store metadata such as:

```text
DocumentId
OwnerId
DocumentType
OriginalFileName
StorageKey
ContentType
Size
CreatedAt
```

Use private storage for sensitive documents.

---

# 44. Database Migrations

Schema changes should be versioned through migrations.

```text
Change Entity
   ↓
Create Migration
   ↓
Review Migration
   ↓
Apply
   ↓
Run Tests
```

Do not rely on undocumented manual production schema edits.

---

# 45. Seed Data

Suitable reference seed data can include:

```text
Roles
Permissions
Governorates
Cities
Medical Specialties
Service Categories
Blood Groups
Document Types
```

Do not seed real sensitive user data into production.

Development seeds should be obviously fake.

---

# 46. Configuration and Secrets

Configuration may include:

```text
Database
JWT
Email
SMS
Payment Provider
Storage
Location Services
External API Keys
```

Never commit production secrets.

Never put secrets in:

```text
Source Code
README
Frontend JavaScript
Public Configuration
Git History
```

Use environment variables or an appropriate secret store.

---

# 47. Environments

Recommended:

```text
Development
Testing
Production
```

Configuration should be environment-aware.

Example:

```text
appsettings.json
appsettings.Development.json
appsettings.Testing.json
appsettings.Production.json
```

Production secrets must be injected securely.

---

# 48. CORS

Configure CORS explicitly.

Development can allow trusted local frontend origins.

Production should allow only the intended frontend origins.

Avoid unrestricted CORS in production for a sensitive healthcare API.

---

# 49. HTTPS

Production should use HTTPS.

Especially sensitive operations:

```text
Login
Password Reset
Medical Records
Documents
Location
Payments
```

must use protected transport.

---

# 50. Rate Limiting

Apply rate limits to high-risk or abuse-prone endpoints.

Examples:

```text
Login
OTP
Password Recovery
Password Reset
Public Search
File Upload
```

This reduces brute-force and abuse risks.

---

# 51. API Versioning

Use a consistent API versioning strategy.

Recommended route style:

```text
/api/v1/...
```

Examples:

```text
/api/v1/auth/login
/api/v1/auth/register
/api/v1/auth/verify-otp

/api/v1/patients/me
/api/v1/doctors
/api/v1/nurses
/api/v1/pharmacies
/api/v1/laboratories
/api/v1/blood-donors

/api/v1/bookings
/api/v1/notifications
/api/v1/ratings
/api/v1/complaints
```

Exact endpoint names should be defined by the API contract.

---

# 52. HTTP Status Codes

Use meaningful status codes:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity where adopted
429 Too Many Requests
500 Internal Server Error
```

Do not use `200 OK` for every failure.

---

# 53. API Error Contract

Use one consistent error structure.

Example:

```json
{
  "status": 400,
  "title": "Validation failed",
  "errors": {
    "email": [
      "A valid email address is required."
    ]
  },
  "traceId": "..."
}
```

Never expose stack traces, SQL details, secrets, or internal infrastructure information.

---

# 54. Global Exception Handling

Use centralized exception handling middleware.

```text
Request
  ↓
Controller
  ↓
Application
  ↓
Exception
  ↓
Global Handler
  ↓
Safe API Error
```

Internally:

```text
Log diagnostics
Keep trace/correlation information
```

Externally:

```text
Return safe, useful client message
```

---

# 55. Logging

Use structured logs.

Good examples:

```text
Booking created
Provider verification approved
Payment state changed
```

Never log:

```text
Passwords
OTP values
Access tokens
Refresh tokens
National IDs
Medical record contents
Private verification documents
Payment secrets
```

---

# 56. Audit Logging

Audit logging is appropriate for sensitive operations such as:

```text
Provider approval/rejection
Provider suspension
Sensitive medical record access
Financial state changes
Complaint decisions
Administrative actions
```

Audit data must itself be protected.

---

# 57. Background Work

Potential background tasks:

```text
Expire pending bookings
Expire OTPs
Send appointment reminders
Retry notifications
Clean temporary sessions
Process asynchronous operations
```

Do not rely on users keeping the frontend browser open.

Use the team's approved background-job approach.

---

# 58. API Architecture

Recommended request flow:

```text
HTTP Request
    ↓
Middleware
    ↓
Authentication
    ↓
Authorization
    ↓
Controller / Endpoint
    ↓
Application Use Case
    ↓
Domain Rules
    ↓
Infrastructure
    ↓
Database / External Service
    ↓
DTO Response
```

---

# 59. API Documentation

Swagger/OpenAPI should describe:

- Endpoint
- Method
- Auth requirement
- Authorization
- Parameters
- Request body
- Validation
- Success response
- Error responses
- Business rules

Example:

```text
POST /api/v1/bookings

Auth:
Required

Role:
Patient

Rules:
Provider must be active.
Service must belong to provider.
Slot must be available.
Location must be supported.
```

---

# 60. Testing Strategy

Use several test levels:

```text
Unit Tests
    +
Integration Tests
    +
API Tests
    +
Security/Authorization Tests
```

---

# 61. Unit Tests

Test:

- Domain rules
- Validation
- Price calculation
- Booking transitions
- Availability
- Matching
- Permission decisions

Examples:

```text
Reject invalid booking transition.
Reject booking outside provider coverage.
Reject rating for incomplete booking.
Do not activate unverified provider.
```

---

# 62. Integration Tests

Test:

- Authentication
- Database persistence
- Registration
- Provider verification
- Booking creation
- Medical record access
- Payment state
- Search
- Notification persistence

---

# 63. Security Tests

Must cover:

```text
Unauthenticated → 401
Authenticated but forbidden → 403
Unknown resource → 404 where appropriate
Cross-user access → denied
Cross-provider access → denied
Mass assignment → blocked
Invalid state transition → blocked
Rate limit → enforced
```

---

# 64. Performance

Use:

- Pagination
- DTO projections
- Appropriate database indexes
- Efficient queries
- Avoidance of N+1 queries
- Controlled external calls
- Appropriate caching for stable reference data
- Background processing for non-interactive tasks

Review expensive search and location queries.

Do not optimize by adding infrastructure before there is a real need.

---

# 65. Search Query Performance

Potentially indexed/reference fields may include:

```text
Email
Phone
Provider status
Verification status
Specialty
City
Booking status
Booking date
Notification recipient
```

Spatial indexing/querying can be used when supported by the chosen database and required by actual search patterns.

---

# 66. API Contract With Frontend

The backend and frontend must share stable conventions for:

```text
Role values
Status values
Request DTOs
Response DTOs
Error format
Pagination
Document upload responses
Location model
Authentication responses
```

When an API contract changes:

```text
Backend
  ↓
API Documentation
  ↓
Frontend Services
  ↓
Frontend UI
  ↓
Tests
```

All relevant layers must be updated.

---

# 67. Authorization Matrix

This is a conceptual starting point and must be finalized with the actual product rules.

| Capability | Patient | Doctor | Nurse | Pharmacy | Lab | Donor | Admin |
|---|---:|---:|---:|---:|---:|---:|---:|
| Manage own profile | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Search providers | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Create patient booking | ✓ | - | - | - | - | - | Controlled |
| Manage own services | - | ✓ | ✓ | ✓ | ✓ | - | Controlled |
| Access own medical data | ✓ | - | - | - | - | - | Controlled |
| Provider verification | - | - | - | - | - | - | ✓ |
| Admin operations | - | - | - | - | - | - | ✓ |

The final matrix should be maintained as project documentation and enforced on the backend.

---

# 68. Registration Lifecycle

Provider registration:

```text
Account
  ↓
Verification
  ↓
Role Profile
  ↓
Documents
  ↓
Verification Application
  ↓
PENDING_VERIFICATION
  ↓
Admin Review
```

Patient registration:

```text
Account
  ↓
Verification
  ↓
Patient Profile
  ↓
Account Ready
```

Professional activation must remain separate from normal authentication.

---

# 69. Provider Visibility

A provider should normally become publicly discoverable only when appropriate conditions are met, such as:

```text
Account Active
+
Verification Approved
+
Profile Complete
+
Service Active
+
Not Suspended
```

Keep these visibility rules centralized.

---

# 70. Pharmacy Visibility

A pharmacy/branch may require:

```text
Approved Pharmacy
+
Active Branch
+
Valid Location
+
Active Service
```

Medicine availability should be linked to the appropriate branch.

---

# 71. Blood Donor Matching

High-level backend flow:

```text
Blood Need
   ↓
Compatibility Rules
   ↓
Potential Donors
   ↓
Location Filter
   ↓
Availability Filter
   ↓
Privacy Rules
   ↓
Notification
```

Do not expose unnecessary donor personal information.

---

# 72. Timeouts and Expiration

Use server-side timestamps for:

```text
OTP expiration
Booking expiration
Token/session expiration
Temporary upload/session expiration
```

Frontend timers are informational only.

---

# 73. Soft Delete and Status

Use soft delete only when business/audit requirements justify it.

Where status matters, make it explicit:

```text
ACTIVE
INACTIVE
SUSPENDED
DELETED
PENDING
```

Do not use ambiguous flags everywhere.

---

# 74. Coding Standards

Use:

- Meaningful names
- Small focused methods
- Dependency Injection
- Async I/O
- Explicit DTOs
- Clear interfaces
- Centralized error handling
- Clear module boundaries

Avoid:

- Giant controllers
- God services
- Duplicated business logic
- Magic strings
- Hardcoded secrets
- Entity exposure
- Global mutable state

---

# 75. C# Naming

Use normal C# conventions:

```text
BookingService
DoctorController
ProviderVerification
CreateBookingAsync()
GetProviderAsync()
```

Private fields:

```text
_logger
_bookingRepository
```

Avoid meaningless names such as:

```text
x
temp
data1
thing
```

when they reduce readability.

---

# 76. Async Programming

Prefer asynchronous database and external I/O.

```csharp
await service.ExecuteAsync(cancellationToken);
```

Avoid blocking asynchronous operations unnecessarily.

Use `CancellationToken` where appropriate for long-running or I/O-bound operations.

---

# 77. Dependency Injection

Prefer constructor injection:

```csharp
public class BookingService
{
    private readonly IBookingRepository _bookingRepository;

    public BookingService(IBookingRepository bookingRepository)
    {
        _bookingRepository = bookingRepository;
    }
}
```

Keep dependencies explicit.

---

# 78. Repository / Data Access

Do not create repositories automatically for every table just for the sake of having repositories.

Use EF Core directly or repository abstractions where they provide meaningful value:

- Complex queries
- Aggregate boundaries
- Testability
- Persistence abstraction
- Module boundaries

The goal is clarity, not maximum abstraction.

---

# 79. Database Query Rules

For important queries:

- Return only needed fields
- Use projection when useful
- Avoid N+1 behavior
- Paginate
- Add indexes based on real query patterns
- Avoid unnecessary eager loading
- Review expensive joins and location queries

---

# 80. Local Development Workflow

Typical setup:

```text
Clone Repository
     ↓
Configure Environment
     ↓
Restore Dependencies
     ↓
Configure Database
     ↓
Apply Migrations
     ↓
Run API
     ↓
Open Swagger
     ↓
Connect Frontend
```

Exact commands depend on the actual solution name and database configuration.

---

# 81. Docker

When containerization is used:

```text
backend/
├── Dockerfile
└── docker-compose.yml
```

Typical conceptual environment:

```text
TechCare API
     +
Database
     +
Optional supporting services
```

Pass configuration through environment variables.

Never bake secrets into Docker images.

---

# 82. Git Workflow

Recommended flow:

```text
Update Main
   ↓
Feature Branch
   ↓
Implement
   ↓
Test
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
Review
   ↓
Merge
```

Avoid unrelated changes in one PR.

---

# 83. Commit Convention

Recommended examples:

```text
feat(auth): add OTP verification endpoint
feat(booking): create booking workflow
feat(nurse): add nurse profile endpoint
fix(provider): block unverified service activation
fix(payment): reject client-controlled total
test(booking): add duplicate-slot test
refactor(search): extract provider query service
docs(api): update booking contract
```

Avoid vague commits such as:

```text
update
fix
changes
final
final2
new
```

---

# 84. Pull Request Checklist

```text
[ ] Business rules defined
[ ] Validation implemented
[ ] Authorization implemented
[ ] Ownership checks implemented
[ ] DTOs used
[ ] Persistence handled
[ ] Tests added/updated
[ ] Migration included if required
[ ] Swagger/API docs updated
[ ] No secrets committed
[ ] Logging is safe
[ ] No unrelated changes
```

---

# 85. Security Checklist

```text
[ ] HTTPS
[ ] Authentication
[ ] Authorization
[ ] Ownership checks
[ ] Password hashing
[ ] OTP protection
[ ] Rate limiting
[ ] Input validation
[ ] Secure file storage
[ ] Safe logging
[ ] Externalized secrets
[ ] Restricted CORS
[ ] Sanitized errors
[ ] Protected admin endpoints
[ ] Protected medical records
[ ] Protected financial operations
[ ] Audit-sensitive actions reviewed
```

---

# 86. Data Integrity Checklist

```text
[ ] Unique constraints
[ ] Foreign keys
[ ] Booking concurrency protection
[ ] Transactional financial changes
[ ] Controlled provider status transitions
[ ] Controlled booking transitions
[ ] Location validation
[ ] Server-side pricing
[ ] Server-side expiration
[ ] Idempotent retry-sensitive operations
```

---

# 87. Backend Definition of Done

A backend feature is complete when it has:

```text
Requirement
+
Business Rules
+
Domain/Application Logic
+
Validation
+
Authorization
+
Persistence
+
API Endpoint
+
Error Handling
+
Tests
+
Documentation
+
Security Review
```

"Endpoint returns 200" is not a complete feature definition.

---

# 88. Recommended Development Order

A practical dependency-oriented order:

```text
1. Solution / Project Structure
        ↓
2. Database + Core Entities
        ↓
3. Authentication
        ↓
4. Authorization
        ↓
5. Role Profiles / Registration APIs
        ↓
6. Provider Verification
        ↓
7. Location / Search
        ↓
8. Booking
        ↓
9. Medical Records / Consultation
        ↓
10. Payments
        ↓
11. Notifications
        ↓
12. Ratings / Complaints
        ↓
13. Admin
        ↓
14. Security / Performance / QA
```

Independent work can proceed in parallel once interfaces and contracts are agreed.

---

# 89. Team Collaboration

The team has five full-stack .NET developers.

Frontend registration ownership is:

| Team Member | Registration |
|---|---|
| Mostafa | Authentication & Authorization frontend integration + Laboratory |
| Ahmed | Nurse |
| Ahd | Doctor |
| Alaa | Pharmacy |
| Eman | Patient + Blood Donor |

Backend feature ownership should be agreed explicitly before implementation. Shared backend areas such as authentication infrastructure, common middleware, API conventions, core database abstractions, and authorization must be coordinated before major changes are merged.

---

# 90. Architecture Rules for Parallel Work

To minimize conflicts:

```text
Shared Layer
   ↓
Stable Interfaces / Contracts
   ↓
Module Work
   ↓
Independent Tests
   ↓
Small PR
```

Avoid having multiple developers simultaneously rewrite:

```text
Program.cs
DbContext
Core User entity
Authentication configuration
Shared middleware
```

without coordination.

---

# 91. MVP Scope

Current core backend scope:

```text
Authentication
Authorization
Patient
Doctor
Nurse
Pharmacy
Laboratory
Blood Donor
Provider Verification
Booking / Appointment
Medical Records / Consultation
Payment / Financial
Notifications
Ratings
Complaints
Location / Geo
Search / Discovery
Admin
```

Future concepts should remain outside the core MVP unless explicitly activated:

```text
AI Symptom Assistant
Emergency Live Location
Hospital Integration
Advanced Telemedicine
Insurance
Advanced Messaging
Mobile Applications
Advanced Analytics
External Healthcare Integrations
```

---

# 92. Future-Proofing

The backend should be ready for growth through:

```text
Clear module boundaries
Stable DTO contracts
API versioning
Domain rules
Service abstractions
Configuration
Tests
Documentation
```

Avoid premature microservices, distributed infrastructure, or complex event systems without a real requirement.

---

# 93. Final Backend Flow

```text
                         CLIENTS
                            │
                            ▼
                     ASP.NET Core API
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
            Authentication       Authorization
                  │                   │
                  └─────────┬─────────┘
                            ▼
                      Application
                            │
                ┌───────────┼────────────┐
                ▼           ▼            ▼
             Profiles    Providers     Services
                │           │            │
                └───────────┼────────────┘
                            ▼
                         Booking
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
          Payments      Medical Data   Notifications
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                         Admin
                            │
                            ▼
                      Infrastructure
                            │
                ┌───────────┼────────────┐
                ▼           ▼            ▼
            Database     Storage     External APIs
```

---

# 94. Final Principles

TechCare should be developed as a serious healthcare backend.

Always prioritize:

```text
Security
Correctness
Data Integrity
Clear Business Rules
Maintainability
Testability
Consistency
```

The backend must be the authoritative source for:

```text
Identity
Roles
Permissions
Provider status
Bookings
Availability
Prices
Financial state
Medical access
Location validation
Search visibility
```

The frontend can improve UX, but it must never be the security boundary.

---

## End of TechCare Backend README
