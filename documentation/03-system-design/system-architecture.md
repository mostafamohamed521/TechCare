# 🏗️ TechCare — System Architecture

## 1. Purpose

This document defines the technical architecture of the TechCare platform.

It explains how the system is organized, how its main components communicate, where business logic belongs, how data flows through the application, and how shared concerns such as authentication, authorization, security, logging, and validation are handled.

The goal is to provide one shared technical blueprint for the entire development team.

---

# 2. Architecture Goals

The architecture is designed to achieve:

- Clear separation of responsibilities.
- Maintainable code.
- Low coupling between modules.
- Reusable shared services.
- Secure handling of sensitive data.
- Testable business logic.
- Scalable backend structure.
- Consistent API behavior.
- Clear ownership boundaries.
- Easier future expansion.

---

# 3. High-Level Architecture

TechCare follows a layered backend architecture with a separated frontend.

```text
┌──────────────────────────────────────────────┐
│                  FRONTEND                    │
│              HTML / CSS / JS                │
└──────────────────────┬───────────────────────┘
                       │
                       │ HTTPS / REST API
                       ▼
┌──────────────────────────────────────────────┐
│                TechCare.API                 │
│ Controllers / Middleware / HTTP             │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│           TechCare.Application               │
│ Use Cases / Commands / Queries / DTOs       │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│              TechCare.Domain                │
│ Entities / Rules / Value Objects / Enums    │
└──────────────────────┬───────────────────────┘
                       ▲
                       │
┌──────────────────────┴───────────────────────┐
│          TechCare.Infrastructure             │
│ EF Core / Persistence / External Services    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
                 ┌────────────┐
                 │  Database  │
                 └────────────┘
4. Architecture Style

The backend follows a layered architecture inspired by Clean Architecture principles.

The main layers are:

API
Application
Domain
Infrastructure

The key principle is:

Business rules should not depend directly on infrastructure details.

5. Main Layers
5.1 API Layer

Project:

TechCare.API
Responsibility

The API layer is responsible for handling HTTP communication.

It contains concerns such as:

Controllers
Middleware
API configuration
Authentication pipeline configuration
Exception handling
Request/response handling
Dependency injection setup
The API Layer Should
Receive HTTP requests.
Validate basic request structure.
Call application use cases.
Return HTTP responses.
The API Layer Should Not
Contain complex business rules.
Directly perform complex database operations.
Implement authentication logic manually inside every controller.
Duplicate business rules.

Example:

POST /api/bookings
        ↓
BookingController
        ↓
Application Use Case
        ↓
Domain Rules
        ↓
Infrastructure
6. Application Layer

Project:

TechCare.Application

The Application layer contains application-specific business use cases.

It coordinates the execution of business operations without depending directly on infrastructure implementation details.

It may contain:

Commands
Queries
DTOs
Validators
Application services
Interfaces
Results
Exceptions
Mapping
Feature-specific use cases

Example structure:

Features/
├── Auth/
├── Patient/
├── Doctor/
├── Nurse/
├── Pharmacy/
├── Laboratory/
└── BloodDonor/
7. Application Layer Responsibilities

The Application layer is responsible for:

Executing use cases.
Coordinating domain operations.
Validating application input.
Calling domain logic.
Using repository/service interfaces.
Returning application results.
Controlling workflow orchestration.

Example:

Create Booking
       ↓
Validate Input
       ↓
Check Provider
       ↓
Check Availability
       ↓
Calculate Price
       ↓
Create Booking
       ↓
Save
       ↓
Notify Provider
8. Domain Layer

Project:

TechCare.Domain

The Domain layer represents the core business concepts of TechCare.

It should be independent from:

ASP.NET Core
EF Core
Database providers
HTTP
JWT implementation
External services

It contains concepts such as:

Entities
Value Objects
Enums
Domain rules
Domain interfaces
Constants

Examples:

User
Patient
Doctor
Nurse
Pharmacy
Laboratory
BloodDonor
Booking
Consultation
MedicalRecord
Payment
Notification
Complaint
Rating
9. Domain Layer Principle

The Domain layer answers:

What are the actual business concepts and rules of TechCare?

For example:

A booking cannot normally move from:

Completed
   ↓
Pending

because that violates the business lifecycle.

That rule belongs to business/domain logic rather than frontend code.

10. Infrastructure Layer

Project:

TechCare.Infrastructure

The Infrastructure layer contains technical implementations.

Examples include:

Entity Framework Core
Database context
Repositories
Authentication implementation
External services
Email services
SMS services
File storage
Payment integrations
Location provider integrations
Notification infrastructure

The Infrastructure layer implements interfaces defined by inner layers when appropriate.

11. Dependency Direction

The architecture should follow a controlled dependency direction.

API
 ↓
Application
 ↓
Domain

Infrastructure
 ↓
Implements required interfaces

The key idea is:

Domain
   ↑
Application
   ↑
API

while Infrastructure provides implementations for required technical services.

Business logic must not become tightly coupled to infrastructure implementations.

12. Dependency Rule

A feature should depend on abstractions where appropriate.

Example:

Application
   ↓
IEmailService

Infrastructure then provides:

EmailService

This makes it possible to change the technical implementation without rewriting business use cases.

13. Frontend Architecture

The frontend is built using:

HTML5
CSS3
JavaScript

Recommended structure:

frontend/
├── pages/
├── css/
├── js/
├── assets/
└── components/
14. Frontend Responsibilities

The frontend is responsible for:

Displaying interfaces.
Collecting user input.
Client-side validation.
Calling APIs.
Rendering API results.
Navigation.
UI state.
User feedback.
Shared components.

The frontend is not trusted as a security boundary.

Important authorization decisions must always be enforced by the backend.

15. Frontend API Communication

The frontend communicates with the backend through HTTP REST APIs.

General flow:

HTML / JS
   ↓
API Service
   ↓
HTTP Request
   ↓
ASP.NET Core API
   ↓
Response
   ↓
JavaScript
   ↓
UI Update
16. Backend Request Lifecycle

A typical request follows:

Client
   ↓
HTTP Request
   ↓
Middleware
   ↓
Authentication
   ↓
Authorization
   ↓
Controller
   ↓
Application Use Case
   ↓
Domain Rules
   ↓
Infrastructure
   ↓
Database / External Service
   ↓
Result
   ↓
Application
   ↓
Controller
   ↓
HTTP Response
   ↓
Client
17. Middleware Layer

Middleware provides cross-cutting request processing.

Possible responsibilities include:

Authentication
Exception handling
Logging
Request tracing
CORS
Rate limiting
Other infrastructure concerns

The middleware pipeline should be centralized rather than duplicated across controllers.

18. Authentication Architecture

Authentication is centralized.

All roles use the same core authentication mechanism.

Patient
Doctor
Nurse
Pharmacy
Laboratory
Blood Donor
Admin
      │
      ▼
Central Authentication
      │
      ▼
Identity
      │
      ▼
Role / Permissions

The login page does not require role selection.

19. Authentication Flow
User
 ↓
Login
 ↓
Credentials
 ↓
Authentication Service
 ↓
Credential Validation
 ↓
Account Validation
 ↓
Token Generation
 ↓
Role / Claims
 ↓
Authenticated Request
20. Registration Architecture

Registration begins with role selection.

Register
   ↓
Choose Role
   ├── Patient
   ├── Doctor
   ├── Nurse
   ├── Pharmacist
   ├── Laboratory
   └── Blood Donor

Each role then follows a specialized registration workflow while using the same centralized identity system.

21. Authorization Architecture

Authorization is implemented centrally.

Conceptually:

Authentication
      ↓
Who is the user?
      ↓
Role
      ↓
Permissions / Policies
      ↓
Resource Authorization
      ↓
Allow / Deny

Authorization should be enforced on the backend.

22. Resource Authorization

Role authorization alone is not always enough.

Example:

Patient Role
      ↓
Medical Record
      ↓
Whose Record?

The system must also determine whether the patient is authorized to access that specific resource.

Therefore:

Role Check
    +
Resource Ownership / Relationship Check

may be required.

23. Provider Verification Architecture

Healthcare providers may require verification.

General architecture:

Provider
   ↓
Registration
   ↓
Documents
   ↓
Verification Request
   ↓
Admin Review
   ↓
Verification Status
   ↓
Provider Capability

Provider capabilities should follow verification and account-state rules.

24. Role Module Architecture

Each major business role has its own application area.

Features/
├── Auth/
├── Patient/
├── Doctor/
├── Nurse/
├── Pharmacy/
├── Laboratory/
└── BloodDonor/

The modules are independent in responsibility but connected through shared services.

25. Shared Services

TechCare requires shared platform capabilities.

Examples:

Authentication
Authorization
Location
Notifications
Payments
File Storage
Search
Logging

Shared services should be implemented once and reused when appropriate.

26. Authentication Is a Shared Service

Authentication must not be duplicated per role.

Incorrect approach:

PatientAuth
DoctorAuth
NurseAuth
PharmacyAuth
LaboratoryAuth

Correct approach:

                Auth System
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
   Patient        Doctor        Nurse
      │             │             │
      └─────────────┼─────────────┘
                    ▼
              Other Roles
27. Location Architecture

Location is also a shared capability.

It can support:

Current location
Manual addresses
Coordinates
Distance calculation
Nearby search
Service radius
Distance-based pricing

General flow:

User Location
      ↓
Location Service
      ↓
Coordinates
      ↓
Distance Calculation
      ↓
Search / Booking / Pricing
28. Search Architecture

Search is another cross-module capability.

It can operate on different provider/service domains.

Search
  ├── Doctors
  ├── Nurses
  ├── Pharmacies
  ├── Medicines
  ├── Laboratories
  └── Other Supported Services

Search should use appropriate filtering, pagination, indexing, and authorization rules.

29. Notification Architecture

Modules should not implement unrelated notification mechanisms independently.

Instead:

Business Event
      ↓
Notification Service
      ↓
Notification Record
      ↓
Delivery Channel

Examples:

Booking Accepted
      ↓
Notification Service
      ↓
Patient Notification
30. Payment Architecture

Financial operations should follow a dedicated shared workflow.

Service
   ↓
Price Calculation
   ↓
Financial Transaction
   ↓
Payment
   ↓
Settlement
   ↓
Provider Earnings
   ↓
Withdrawal

Business modules should not directly manipulate unrelated financial records without going through defined financial services/use cases.

31. Medical Information Architecture

Medical information requires stronger authorization boundaries.

Conceptually:

Patient
   ↓
Medical Profile
   ├── Medical History
   ├── Conditions
   ├── Medications
   └── Consultations

Doctors may access relevant information based on authorized relationships and supported workflows.

Other users should not automatically gain access.

32. Booking Architecture

Booking is a shared business capability.

Patient
   ↓
Provider
   ↓
Service
   ↓
Availability
   ↓
Request
   ↓
Booking
   ↓
Appointment
   ↓
Completion

The booking service should enforce valid state transitions and concurrency rules.

33. Booking State Management

General booking lifecycle:

Pending
   ↓
Accepted
   ↓
Scheduled
   ↓
In Progress
   ↓
Completed

Alternative states:

Rejected
Cancelled
Expired

Invalid transitions must be rejected.

34. Concurrency Architecture

Some operations may be performed by multiple users simultaneously.

Example:

Patient A ─┐
           ├──► Same Doctor Slot
Patient B ─┘

The backend must protect critical operations against race conditions and conflicting updates.

This applies especially to:

Booking
Availability
Financial operations
Limited inventory
Request acceptance
35. Database Architecture

The backend uses Entity Framework Core to communicate with the relational database.

General flow:

Application
     ↓
Repository / Data Access Abstraction
     ↓
EF Core
     ↓
DbContext
     ↓
Database

The application should not make database decisions in the frontend.

36. Entity Framework Core Boundary

EF Core implementation belongs in Infrastructure.

Domain entities should not depend on EF Core-specific infrastructure.

Infrastructure maps the domain model to the database.

37. Database Transactions

Critical multi-step operations should use appropriate transaction boundaries.

Example:

Create Booking
      +
Create Financial Record
      +
Update Availability

When required, these operations should succeed or fail consistently.

38. External Services Architecture

TechCare may depend on external services such as:

Email
SMS / OTP
Maps
Payment Provider
File Storage
Other Integrations

External dependencies should be accessed through abstraction boundaries.

Example:

Application
     ↓
IEmailService
     ↓
Infrastructure
     ↓
Email Provider

This prevents business logic from becoming tied directly to one external provider.

39. File Storage Architecture

Provider verification documents may require file storage.

General flow:

Provider
   ↓
Upload Document
   ↓
Validation
   ↓
Storage Service
   ↓
Private Storage
   ↓
Document Reference

Documents should not be publicly accessible by default.

40. Error Handling Architecture

Errors should be handled consistently.

General flow:

Exception / Failure
        ↓
Central Error Handling
        ↓
Structured Error Response
        ↓
Frontend

Production responses should not expose sensitive internal implementation details.

41. Validation Architecture

Validation can occur at multiple levels.

Frontend Validation
        ↓
API Validation
        ↓
Application Validation
        ↓
Domain Rules
        ↓
Database Constraints

Frontend validation improves user experience.

Backend validation is mandatory for security and correctness.

42. Logging Architecture

Logging is a cross-cutting concern.

Important events may include:

Authentication attempts
Authorization failures
Important administrative actions
Provider verification actions
Critical errors
Important business events

Sensitive values must not be logged unnecessarily.

Do not log:

Passwords
OTP Values
Access Tokens
Refresh Tokens
Secrets
Sensitive Medical Data
43. Security Boundaries

The system contains several security boundaries:

Browser
   │
   │ Untrusted
   ▼
API
   │
   │ Authenticated / Authorized
   ▼
Application
   │
   ▼
Domain
   │
   ▼
Infrastructure
   │
   ▼
Database

The backend is the main trust boundary for business authorization.

44. Data Flow Example — Doctor Booking
Patient Browser
      ↓
Booking Page
      ↓
POST /booking
      ↓
Authentication
      ↓
Authorization
      ↓
Booking Controller
      ↓
Booking Use Case
      ↓
Validate Provider
      ↓
Validate Availability
      ↓
Calculate Price
      ↓
Create Booking
      ↓
Database
      ↓
Notification
      ↓
Response
      ↓
Patient UI
45. Data Flow Example — Provider Verification
Doctor
   ↓
Upload Documents
   ↓
API
   ↓
Authentication
   ↓
Authorization
   ↓
File Validation
   ↓
Private Storage
   ↓
Verification Record
   ↓
Admin Dashboard
   ↓
Admin Review
   ↓
Approve / Reject
   ↓
Provider Status Updated
   ↓
Provider Notification
46. Data Flow Example — Medicine Search
Patient
   ↓
Medicine Search
   ↓
Search API
   ↓
Search Service
   ↓
Medicine Data
   ↓
Pharmacy Branches
   ↓
Location / Distance
   ↓
Availability
   ↓
Rank Results
   ↓
Return Results
47. Data Flow Example — Medical Consultation
Doctor
   ↓
Open Eligible Appointment
   ↓
Authorization Check
   ↓
Patient Information
   ↓
Consultation
   ↓
Diagnosis / Treatment
   ↓
Save Consultation
   ↓
Medical Record
   ↓
Completion
   ↓
Notification / Financial Events
48. Cross-Cutting Concerns

The following concerns apply to many modules:

Authentication
Authorization
Validation
Logging
Error Handling
Security
Notifications
Location
Transactions
Monitoring

These should be implemented consistently.

49. Module Communication Principle

Business modules should communicate through clearly defined application contracts rather than directly accessing each other's internal implementation.

Bad:

DoctorController
   ↓
Directly access PharmacyDbContext

Better:

Doctor Feature
   ↓
Defined Application Service / Contract
   ↓
Required Capability

This reduces coupling.

50. Module Ownership

Current ownership:

Mostafa
├── Authentication & Authorization
└── Laboratory

Ahmed
└── Nurse

Ahd
└── Doctor

Alaa
└── Pharmacy

Eman
├── Patient
└── Blood Donor

Ownership defines the primary development responsibility, not permission to violate shared architecture.

51. Shared Code Principle

Shared functionality should be centralized when it is genuinely shared.

Examples:

Auth
Authorization
Location
Notifications
Common Errors
Common Results

However, business-specific logic should remain inside the corresponding feature.

52. Architecture Rules

The following rules should be followed:

Controllers should remain thin.
Business logic should not live in controllers.
Domain logic should remain independent.
Infrastructure should contain implementation details.
Authentication should be centralized.
Authorization should be enforced on the backend.
Modules should have clear ownership.
Shared services should not be duplicated.
Sensitive data should have strict access control.
Database access should remain behind the appropriate architectural boundary.
Features should be testable independently.
Cross-module dependencies should remain explicit.
53. What Developers Should Avoid

The following patterns should be avoided:

❌ Business logic inside Controllers
❌ Business logic inside JavaScript
❌ Duplicate JWT systems
❌ Duplicate authentication per role
❌ Direct database access from Controllers
❌ Exposing private documents publicly
❌ Trusting frontend authorization
❌ Copy-pasting shared services
❌ Direct access to another module's internal implementation
❌ Hard-coded secrets
❌ Hard-coded business rules where configuration is appropriate
54. Architecture and Security Relationship

Security is not a separate final step.

It exists across the architecture:

Frontend
   ↓
API Security
   ↓
Authentication
   ↓
Authorization
   ↓
Application Validation
   ↓
Domain Rules
   ↓
Infrastructure Security
   ↓
Database Protection

Every layer contributes to the overall security posture.

55. Architecture and Scalability

The architecture is designed to allow future growth.

Possible future growth areas include:

More Users
More Providers
More Bookings
More Medical Records
More Notifications
More Pharmacies
More Laboratories
More Services
Mobile Apps
External Integrations

The platform should grow without requiring a complete rewrite of the core domain.

56. Architecture and Future Mobile Support

The backend is API-driven.

Therefore:

                    TechCare API
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Web App   Mobile App   Future Client

The backend should not depend on the current web frontend implementation.

57. Architecture and Testing

The architecture supports multiple levels of testing.

Domain
   ↓
Unit Tests

Application
   ↓
Unit / Integration Tests

API
   ↓
Integration Tests

Database
   ↓
Integration Tests

Frontend
   ↓
UI / Functional Tests

The goal is to test business logic independently from external infrastructure wherever practical.

58. Architecture Decision Principles

When making a technical architecture decision, the team should evaluate:

Security
Maintainability
Simplicity
Testability
Performance
Scalability
Reusability
Project Scope

The simplest architecture that satisfies the requirements should be preferred over unnecessary complexity.

59. Architecture Summary

The TechCare architecture can be summarized as:

                   CLIENT
                     │
                     ▼
                 FRONTEND
                     │
                     ▼
                 REST API
                     │
                     ▼
                   API
                     │
                     ▼
               APPLICATION
                     │
                     ▼
                  DOMAIN
                     ▲
                     │
              INFRASTRUCTURE
                     │
            ┌────────┴────────┐
            ▼                 ▼
        DATABASE       EXTERNAL SERVICES

Shared concerns surround the architecture:

Authentication
Authorization
Security
Validation
Logging
Notifications
Monitoring
60. Final Architecture Principle

The most important architecture principle of TechCare is:

Keep business logic independent, keep responsibilities separated, keep shared capabilities centralized, and keep infrastructure replaceable.

The architecture exists to help the five developers build one consistent system instead of five unrelated implementations.

🩺 TechCare Architecture

One platform. Clear boundaries. Shared standards. Independent responsibilities.