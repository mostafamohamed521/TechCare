# 🧩 TechCare — System Components

> **This document defines the major technical and business components of the TechCare platform, their responsibilities, boundaries, dependencies, interactions, inputs, outputs, and shared capabilities.**

---

# 1. Document Purpose

The purpose of this document is to describe the major components that make up the TechCare platform.

The document answers the following questions:

- What are the main system components?
- What does each component do?
- What responsibility belongs to each component?
- Which components are shared?
- Which components belong to specific business modules?
- How do components communicate?
- What are the main dependencies?
- Which components handle sensitive information?
- Which components interact with the database?
- Which components interact with external services?

This document complements:

```text
system-architecture.md

The architecture document explains the overall structure, while this document explains the individual components in more detail.

2. Component Classification

TechCare components are divided into several categories:

                    TECHCARE COMPONENTS
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
   Presentation         Business            Technical
   Components           Components          Components
        │                   │                   │
        ▼                   ▼                   ▼
   Frontend            Healthcare          Database
   API                  Modules             Infrastructure

The main classifications are:

Presentation Components
Frontend
Pages
Shared UI components
API client layer
Platform Components
Authentication
Authorization
User Management
Notification
Location
Search
File Management
Business Components
Patient
Doctor
Nurse
Pharmacy
Laboratory
Blood Donor
Booking
Medical Records
Payment
Ratings
Complaints
Provider Verification
Infrastructure Components
Database
EF Core
External Services
Email
SMS
File Storage
Logging
Monitoring
3. High-Level Component Map
                              TECHCARE
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
    Frontend                ASP.NET Core API          External Services
        │                         │                         │
        │                         │                         ├── Email
        │                         │                         ├── SMS / OTP
        │                         │                         ├── Maps
        │                         │                         ├── Payment
        │                         │                         └── Storage
        │                         │
        │                         ▼
        │                  Platform Services
        │                         │
        │        ┌────────────────┼────────────────┐
        │        │                │                │
        │        ▼                ▼                ▼
        │   Authentication   Authorization    Notifications
        │
        │
        └───────────────────────┬───────────────────────────
                                │
                                ▼
                        Business Components
                                │
       ┌────────┬────────┬──────┼──────┬────────┬──────────┐
       ▼        ▼        ▼      ▼      ▼        ▼          ▼
    Patient  Doctor   Nurse  Pharmacy  Lab   Blood Donor  Admin
                                │
                                ▼
                     Shared Business Services
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
          Booking          Medical Records      Payments
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ▼
                         Database / Storage
4. Frontend Component
Component Name
TechCare Frontend
Technology
HTML5
CSS3
JavaScript
Responsibility

The frontend is responsible for providing the user interface through which users interact with TechCare.

It handles:

Pages
Forms
Navigation
User input
UI validation
API communication
Rendering API responses
Error display
Loading states
Notifications display
Role-specific interfaces
Main Areas
frontend/
├── pages/
├── css/
├── js/
├── assets/
└── components/
Input

Frontend receives:

User actions
Form input
API responses
Authentication state
Notifications
Output

Frontend produces:

HTTP requests
User interface updates
Navigation
Validation feedback
Security Boundary

The frontend is not trusted as the security boundary.

A hidden button does not mean the operation is secure.

Backend authorization remains mandatory.

5. Frontend API Client Component
Responsibility

The API client layer handles communication between JavaScript and the ASP.NET Core backend.

Typical responsibilities:

HTTP requests
Authentication headers
Request serialization
Response handling
Common error handling
Token/session handling
API endpoint abstraction

Conceptual flow:

Page
 ↓
JavaScript Service
 ↓
API Client
 ↓
HTTP Request
 ↓
ASP.NET Core API
6. API Component
Component Name
TechCare.API
Responsibility

The API component is the main HTTP entry point into the backend.

It handles:

Routing
Controllers
HTTP requests
HTTP responses
Middleware
Authentication pipeline
Authorization pipeline
Model binding
Basic request processing
Input

Examples:

POST
GET
PUT
PATCH
DELETE
Output

Examples:

200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
500 Internal Server Error

The exact status-code strategy is defined by the API conventions.

7. Middleware Component
Responsibility

Middleware performs cross-cutting request processing.

Possible middleware includes:

Exception handling
Authentication
Logging
Request tracing
CORS
Rate limiting
Other common pipeline concerns

General flow:

Incoming Request
      ↓
Middleware Pipeline
      ↓
Controller
8. Global Exception Handling Component
Responsibility

The global exception handling mechanism converts unexpected application failures into consistent API responses.

Exception
   ↓
Global Handler
   ↓
Log Error
   ↓
Safe API Response

The response must not expose sensitive implementation details.

9. Authentication Component
Component Name
Authentication
Responsibility

Authentication verifies the identity of users.

It handles concepts such as:

Registration
Login
Password validation
OTP
Verification
JWT
Access tokens
Refresh tokens
Logout
Session management
Password recovery
Users

The component serves:

Patient
Doctor
Nurse
Pharmacist
Laboratory
Blood Donor
Admin
Important Rule

There is one centralized authentication system.

The system must not create separate authentication systems for each role.

10. Registration Component
Responsibility

The Registration component handles account creation.

General process:

Visitor
   ↓
Register
   ↓
Choose Role
   ↓
Role-Specific Form
   ↓
Validate Information
   ↓
Create Identity
   ↓
Create Role Profile
   ↓
Verification
Supported Public Registration Roles
Patient
Doctor
Nurse
Pharmacist / Pharmacy
Laboratory
Blood Donor

Admin is not part of the public registration workflow.

11. Authorization Component
Responsibility

Authorization determines whether an authenticated user is allowed to perform an operation.

It handles:

Role checks
Permission checks
Policy checks
Resource ownership
Resource relationships
Administrative permissions

General flow:

Authenticated User
       ↓
Role
       ↓
Permission
       ↓
Resource Authorization
       ↓
Allow / Deny
12. Identity Component
Responsibility

The Identity component represents the common identity of users across the entire system.

It provides a shared identity reference for:

Patient
Doctor
Nurse
Pharmacist
Laboratory
Blood Donor
Admin

The important principle is:

One Account
     ↓
One Identity
     ↓
One Role Model
     ↓
Role-Specific Profile

This prevents separate identities from being created for every module.

13. Patient Component
Component Name
Patient
Responsibility

Handles patient-specific functionality.

Main Responsibilities
Patient profile
Address
Location
Medical profile
Medical history
Current medications
Provider discovery
Requests
Bookings
Service history
Notifications
Ratings
Complaints
Main Dependencies
Authentication
Authorization
Location
Search
Booking
Medical Records
Notifications
Payments
Ratings
Complaints
14. Doctor Component
Component Name
Doctor
Responsibility

Handles doctor-specific functionality.

Main Responsibilities
Doctor profile
Specialty
Experience
Services
Availability
Documents
Verification
Requests
Appointments
Consultation
Diagnosis
Treatment
Earnings
Withdrawals
Ratings
Complaints
Main Dependencies
Authentication
Authorization
Verification
Location
Booking
Medical Records
Notifications
Payments
Ratings
Complaints
File Management
15. Nurse Component
Component Name
Nurse
Responsibility

Handles nurse-specific functionality.

Main Responsibilities
Nurse profile
Professional information
Skills
Nursing services
Service area
Pricing
Availability
Requests
Home visits
Service completion
Earnings
Withdrawals
Ratings
Complaints
Notifications
Main Dependencies
Authentication
Authorization
Verification
Location
Booking
Payments
Notifications
Ratings
Complaints
File Management
16. Pharmacy Component
Component Name
Pharmacy
Responsibility

Handles pharmacy and medicine discovery functionality.

Main Responsibilities
Pharmacy profile
Pharmacist information
Branches
Locations
Working hours
Medicines
Availability
Inventory/status
Main Dependencies
Authentication
Authorization
Location
Search
Notifications
Verification
17. Pharmacy Branch Component
Component Name
Pharmacy Branch
Responsibility

Represents an individual physical pharmacy location.

A pharmacy can contain multiple branches.

Pharmacy
   ├── Branch A
   ├── Branch B
   └── Branch C

Each branch can have:

Address
Coordinates
Working hours
Medicine availability
Inventory/status

This component is important because pharmacy information is not necessarily identical across all branches.

18. Medicine Component
Component Name
Medicine
Responsibility

Represents searchable medicine information.

It can be used by:

Pharmacy management
Medicine discovery
Availability search

The component should separate:

Medicine Definition

from:

Medicine Availability at Branch

Example:

Medicine
   ↓
Pharmacy Branch
   ↓
Availability
19. Laboratory Component
Component Name
Laboratory
Responsibility

Handles laboratory-specific functionality.

Main Responsibilities
Laboratory profile
Responsible person
Location
Working hours
Tests
Services
Verification
Requests
Result-related workflows
Main Dependencies
Authentication
Authorization
Verification
Location
Search
Booking
Notifications
File Management
20. Laboratory Test Component
Component Name
Laboratory Test
Responsibility

Represents supported laboratory tests/services.

It can contain concepts such as:

Test name
Description
Availability
Laboratory association
Other configured test information

Patients can use this component through search and laboratory discovery.

21. Blood Donor Component
Component Name
Blood Donor
Responsibility

Handles donor profiles and donation-related workflows.

Main Responsibilities
Donor profile
Blood type
Availability
Location/area
Matching
Donation notifications
Privacy protection
Main Dependencies
Authentication
Authorization
Location
Search / Matching
Notifications
22. Blood Matching Component
Component Name
Blood Matching
Responsibility

Identifies potential donors based on supported matching criteria.

Possible inputs:

Blood Information
Location / Area
Availability
Compatibility Rules

Flow:

Blood Request
     ↓
Matching Rules
     ↓
Eligible Candidates
     ↓
Location Filtering
     ↓
Availability
     ↓
Potential Donors

The matching component should not expose unnecessary private donor information.

23. Admin Component
Component Name
Admin
Responsibility

Provides authorized platform-level management.

Main Responsibilities
User management
Provider verification
Document review
Complaint management
Rating moderation
Account management
Operational monitoring

Admin capabilities must be protected using strict authorization.

24. Booking Component
Component Name
Booking
Responsibility

The Booking component manages the lifecycle of healthcare service requests and appointments.

Main Responsibilities
Request creation
Provider selection
Service selection
Availability validation
Slot selection
Accept/reject
Cancellation
Expiration
Appointment management
Service completion

General state flow:

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
25. Availability Component
Component Name
Availability
Responsibility

Controls provider availability.

It handles:

Working days
Working hours
Available slots
Unavailable periods
Temporary blocked times

Providers using this component:

Doctor
Nurse
Other Eligible Providers
26. Appointment Component
Component Name
Appointment
Responsibility

Represents scheduled healthcare interactions.

It is related to:

Patient
Provider
Service
Date
Time
Location
Booking

The appointment component works closely with Booking and Notification.

27. Service Component
Component Name
Healthcare Service
Responsibility

Represents services provided through TechCare.

Examples:

Doctor Consultation
Home Nursing
Laboratory Test
Pharmacy / Medicine Discovery

The service component allows the platform to distinguish:

Provider
     ↓
Service
     ↓
Price
     ↓
Availability
28. Location Component
Component Name
Location
Responsibility

Provides shared geographic functionality.

It handles:

Addresses
Coordinates
Current location
Manual location
Saved addresses
Distance
Nearby discovery
Service radius

General flow:

Location
   ↓
Coordinates
   ↓
Distance
   ↓
Nearby Search
29. Geocoding Component
Component Name
Geocoding
Responsibility

Converts between addresses and geographic coordinates.

Address
   ↓
Geocoding
   ↓
Coordinates

Reverse:

Coordinates
   ↓
Reverse Geocoding
   ↓
Address

This functionality may use an external map/location provider.

30. Distance Calculation Component
Component Name
Distance Calculation
Responsibility

Calculates the relevant distance between locations.

Example:

Patient Location
       ↕
Distance
       ↕
Provider Location

Distance information can be used by:

Search
Provider eligibility
Service radius
Travel pricing
31. Search Component
Component Name
Search & Discovery
Responsibility

Provides centralized search capabilities.

It may support:

Doctors
Nurses
Pharmacies
Medicines
Laboratories
Tests
Other supported services

Search responsibilities include:

Query processing
Filtering
Sorting
Pagination
Location-aware results
Relevance
Ranking
32. Search Filter Component
Responsibility

Provides supported filters for discovery.

Examples:

Specialty
Service
Location
Distance
Rating
Price
Availability

Filters can be combined where the business workflow supports it.

33. Search Ranking Component
Responsibility

Determines the ordering of search results.

Possible signals include:

Relevance
Distance
Rating
Availability
Price

The ranking strategy should remain configurable and evolve independently from the search API where possible.

34. Medical Records Component
Component Name
Medical Records
Responsibility

Manages authorized healthcare-related patient information.

Possible data includes:

Medical history
Conditions
Medications
Consultations
Diagnoses
Treatments
Notes

This is one of the most sensitive components in the platform.

It must use strict authorization.

35. Consultation Component
Component Name
Consultation
Responsibility

Represents doctor-patient healthcare encounters.

A consultation may contain:

Patient
Doctor
Encounter
Complaint
Symptoms
Diagnosis
Treatment
Medication
Notes

The Consultation component works closely with:

Booking
Medical Records
Notifications
Payments
36. Medication Component
Component Name
Medication
Responsibility

Represents medication information associated with healthcare workflows.

It may be used by:

Patient medical information
Doctor consultation
Pharmacy search

The system should distinguish between:

Medication Information

and:

Pharmacy Inventory
37. Payment Component
Component Name
Payment & Financial
Responsibility

Manages financial workflows.

It handles concepts such as:

Price calculation
Charges
Payment
Transactions
Provider earnings
Withdrawals
Financial status

General flow:

Service
   ↓
Price
   ↓
Payment
   ↓
Transaction
   ↓
Provider Earnings
   ↓
Withdrawal
38. Pricing Component
Component Name
Pricing
Responsibility

Calculates applicable service pricing.

Possible components:

Base Price
+
Travel Fee
+
Other Configured Charges

The pricing logic should be implemented through configurable business rules where appropriate.

39. Financial Transaction Component
Component Name
Financial Transaction
Responsibility

Represents a financial event generated by the platform.

Potential states include:

Pending
Confirmed
Completed
Failed
Cancelled
Settled

The exact state model depends on the selected financial implementation.

40. Withdrawal Component
Component Name
Withdrawal
Responsibility

Manages provider withdrawal requests.

Supported operations may include:

Create withdrawal request
Validate eligibility
Track status
Display withdrawal history

Providers include:

Doctor
Nurse
Other Eligible Providers
41. Notification Component
Component Name
Notification
Responsibility

Provides a shared notification mechanism.

Notification sources may include:

Booking
Verification
Payment
Appointment
Rating
Complaint
Laboratory results
Blood donation requests

General flow:

Business Event
      ↓
Notification Service
      ↓
Notification
      ↓
Delivery Channel
42. Notification Delivery Component
Responsibility

Handles delivery through supported channels.

Possible channels:

In-App
Email
SMS

The delivery implementation should be separated from business logic.

43. Rating Component
Component Name
Ratings & Reviews
Responsibility

Manages service feedback.

It handles:

Rating creation
Review creation
Rating aggregation
Eligibility validation
Visibility
Moderation support

Ratings should be connected to eligible completed services.

44. Complaint Component
Component Name
Complaints
Responsibility

Manages user complaints.

It handles:

Complaint creation
Category
Description
Status
Administrative review
Resolution

General states:

Created
   ↓
Under Review
   ↓
Investigating
   ↓
Resolved
   ↓
Closed
45. Provider Verification Component
Component Name
Provider Verification
Responsibility

Manages verification of healthcare providers.

Supported provider types include:

Doctor
Nurse
Pharmacy
Laboratory

General lifecycle:

Registration
   ↓
Documents
   ↓
Pending
   ↓
Admin Review
   ↓
Approved / Rejected
46. Document Management Component
Component Name
File & Document Management
Responsibility

Manages professional and verification documents.

Responsibilities include:

Upload
Validation
Storage
Association
Replacement
Access control
47. File Validation Component
Responsibility

Validates uploaded files.

Possible checks:

File Type
File Size
Extension
Content
Storage Rules

The exact validation rules depend on the application's security policy.

48. File Storage Component
Responsibility

Stores uploaded files securely.

General flow:

Upload
   ↓
Validation
   ↓
Private Storage
   ↓
Reference

Sensitive professional documents should not be publicly accessible by default.

49. User Profile Component
Component Name
Profile
Responsibility

Represents user-specific profile information.

Profile behavior depends on role.

Examples:

Patient Profile
Doctor Profile
Nurse Profile
Pharmacy Profile
Laboratory Profile
Blood Donor Profile

The profile layer should remain separate from the central identity where appropriate.

50. Account Status Component
Responsibility

Manages whether an account is currently allowed to operate.

Possible statuses include:

Active
Inactive
Suspended
Blocked
Pending

The exact states depend on the final account lifecycle.

Account status affects:

Authentication
Authorization
Provider Access
Administrative Actions
51. Session Management Component
Responsibility

Tracks supported user authentication sessions.

Possible functionality:

Active sessions
Session revocation
Logout all devices
Token/session expiration
52. OTP Component
Responsibility

Handles one-time passwords used in supported verification workflows.

It manages:

OTP generation
OTP delivery
Expiration
Validation
Attempt limits
Re-request rules

The OTP implementation must not expose OTP values through logs or normal API responses in production.

53. Email Component
Responsibility

Provides email delivery capabilities.

Potential uses:

Account verification
Password recovery
Important notifications
Verification updates

Business modules should communicate through an application-level abstraction rather than depending directly on a concrete email provider.

54. SMS Component
Responsibility

Provides SMS delivery where required.

Possible uses:

OTP
Verification
Important notifications

SMS provider implementation should remain behind a replaceable abstraction.

55. Database Component
Component Name
Database
Responsibility

Persistent storage for TechCare data.

Possible domains include:

Users
Roles
Permissions
Patients
Doctors
Nurses
Pharmacies
Branches
Medicines
Laboratories
Tests
Blood Donors
Bookings
Availability
Consultations
Medical Records
Payments
Transactions
Withdrawals
Notifications
Ratings
Complaints
Verification Documents
56. EF Core Component
Component Name
Entity Framework Core
Responsibility

Provides database access through the backend infrastructure.

It handles:

DbContext
Entity mapping
Migrations
Queries
Persistence
Transactions

EF Core belongs in Infrastructure.

57. Repository / Data Access Component
Responsibility

Provides an abstraction for retrieving and persisting required application data where the architecture uses repositories.

General flow:

Application
    ↓
Abstraction
    ↓
Infrastructure
    ↓
EF Core
    ↓
Database

The exact repository strategy should be used only where it provides real value and should not introduce unnecessary abstraction.

58. Cache Component
Responsibility

A future or optional caching layer may improve performance for frequently requested data.

Potential cached information:

Read-heavy reference data
Search metadata
Configuration
Other safe-to-cache data

Sensitive information should not be cached without an appropriate security and invalidation strategy.

59. Logging Component
Responsibility

Provides application and operational logging.

Possible events:

Authentication attempts
Authorization failures
Important admin operations
Provider verification
Errors
Important business events

Sensitive information must not be logged unnecessarily.

60. Audit Component
Responsibility

Tracks important security or administrative operations when auditability is required.

Possible events:

Provider Approved
Provider Rejected
Account Suspended
Sensitive Administrative Change
Important Security Action

Audit records should be protected from unauthorized modification.

61. Monitoring Component
Responsibility

Provides visibility into application health and operational behavior.

Potential signals include:

API response time
Error rate
Request volume
Authentication failures
Database health
Infrastructure health
62. Configuration Component
Responsibility

Centralizes application configuration.

Configuration may include:

Database connection
JWT settings
OTP lifetime
Request timeout
Pagination defaults
External service settings
Pricing configuration
Environment-specific settings

Secrets should not be hard-coded in source code.

63. Validation Component
Responsibility

Provides consistent validation.

Validation can happen at:

Frontend
   ↓
API
   ↓
Application
   ↓
Domain
   ↓
Database Constraints

The backend remains responsible for authoritative validation.

64. Business Rules Component

Business rules should be kept close to the business layer rather than scattered throughout the system.

Examples:

Booking State Transitions
Provider Eligibility
Service Radius
Pricing Rules
Rating Eligibility
Complaint Rules
Verification Rules
65. Cross-Cutting Components

The following components affect many modules:

Authentication
Authorization
Validation
Logging
Error Handling
Notifications
Location
Search
Configuration
Monitoring
File Management

These should be implemented consistently across the platform.

66. Component Dependency Model

A simplified dependency model is:

                       Frontend
                           │
                           ▼
                          API
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
      Authentication  Authorization    Validation
            │              │              │
            └──────────────┼──────────────┘
                           ▼
                    Application Layer
                           │
       ┌───────────────────┼────────────────────┐
       ▼                   ▼                    ▼
    Patient             Doctor                Nurse
       │                   │                    │
       └───────────────────┼────────────────────┘
                           ▼
                Shared Business Services
                           │
       ┌─────────┬─────────┼─────────┬─────────┐
       ▼         ▼         ▼         ▼         ▼
    Booking   Location   Search   Payment   Notification
       │         │         │         │         │
       └─────────┴─────────┼─────────┴─────────┘
                           ▼
                    Infrastructure
                           │
              ┌────────────┼─────────────┐
              ▼            ▼             ▼
           Database      Storage      External APIs
67. Component Input / Output Model

Each component should have clearly defined boundaries.

General pattern:

Input
  ↓
Validation
  ↓
Component Logic
  ↓
Dependencies
  ↓
Output

Example:

Booking Input
     ↓
Booking Validation
     ↓
Booking Component
     ↓
Availability
     ↓
Pricing
     ↓
Persistence
     ↓
Booking Result
68. Component Communication Rules

Components should communicate through clearly defined contracts.

Preferred:

Component
   ↓
Application Contract
   ↓
Required Capability

Avoid unnecessary direct coupling.

Example of an undesirable dependency:

Doctor Feature
      ↓
Directly Manipulates
Pharmacy Database

Preferred:

Doctor Feature
      ↓
Defined Contract
      ↓
Required Pharmacy Capability
69. Shared Component Rules

A component should be shared when:

Multiple modules need the same capability.
The behavior should remain consistent.
Centralized control provides a real benefit.

Examples:

Authentication
Authorization
Notifications
Location
Search Infrastructure
File Management

A feature should not become shared merely because two modules contain similar-looking code.

70. Business Module Independence

Business modules should remain independently understandable.

Examples:

Patient
Doctor
Nurse
Pharmacy
Laboratory
Blood Donor

Each module should own its domain-specific behavior.

71. Cross-Module Interaction Example
Doctor Booking
Patient
   ↓
Search
   ↓
Doctor
   ↓
Availability
   ↓
Booking
   ↓
Notification
   ↓
Payment
   ↓
Medical Record
   ↓
Rating

Each component performs its own responsibility.

72. Cross-Module Interaction Example
Home Nursing
Patient
   ↓
Location
   ↓
Nurse Search
   ↓
Service Area
   ↓
Availability
   ↓
Pricing
   ↓
Booking
   ↓
Notification
   ↓
Nursing Service
   ↓
Payment
   ↓
Rating
73. Cross-Module Interaction Example
Medicine Search
Patient
   ↓
Medicine Search
   ↓
Pharmacy
   ↓
Branch
   ↓
Location
   ↓
Availability
   ↓
Search Results
74. Cross-Module Interaction Example
Laboratory Service
Patient
   ↓
Test Search
   ↓
Laboratory
   ↓
Location
   ↓
Availability
   ↓
Booking / Request
   ↓
Laboratory Service
   ↓
Result
   ↓
Notification
75. Cross-Module Interaction Example
Blood Donation
Blood Request
   ↓
Blood Matching
   ↓
Location
   ↓
Donor Availability
   ↓
Potential Matches
   ↓
Notification
76. Sensitive Components

The following components contain especially sensitive information:

Authentication
Medical Records
Patient Profile
Location
Provider Documents
Payments
Blood Donor
Admin

These components should have stricter authorization and privacy controls.

77. Components Requiring Strong Authorization

At minimum:

Medical Records
Consultation
Provider Documents
Financial Information
Admin
Precise Location
Account Management

Access should be evaluated according to:

Identity
+
Role
+
Permission
+
Resource Ownership
+
Business Relationship
+
Account Status
78. Components Requiring Concurrency Protection

Concurrency protection is especially important for:

Booking
Availability
Payment
Withdrawal
Pharmacy Inventory
Request Acceptance

Two users attempting conflicting changes simultaneously must not leave the system in an invalid state.

79. Components Requiring Transactions

Potential transactional workflows include:

Booking + Availability
Payment + Transaction
Service Completion + Financial Update
Critical Medical Record Update

The exact transaction boundaries are determined by the business requirements.

80. External Dependency Components

External services may include:

Email Provider
SMS Provider
Map Provider
Payment Provider
File Storage Provider

They should be isolated behind appropriate interfaces or service abstractions.

81. Component Ownership

Primary team ownership:

Mostafa
├── Authentication
├── Authorization
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

Shared components should follow the team's agreed ownership and integration rules.

82. Component Ownership Does Not Mean Isolation

A developer owning a module is responsible for implementing it, but the module must still integrate with shared platform components.

For example:

Ahmed — Nurse
      ↓
Uses
      ↓
Mostafa — Authentication / Authorization

The Nurse module should not create its own authentication mechanism.

83. Component Development Lifecycle

A new component or feature should follow:

Requirement
   ↓
User Story
   ↓
Workflow
   ↓
Component Design
   ↓
API Contract
   ↓
Database Design
   ↓
Implementation
   ↓
Testing
   ↓
Documentation
84. Component Completion Criteria

A component should not be considered complete merely because its UI exists.

The required parts should be implemented where applicable:

[ ] Frontend
[ ] API
[ ] Business Logic
[ ] Validation
[ ] Authorization
[ ] Database
[ ] Error Handling
[ ] Notifications
[ ] Tests
[ ] Documentation
85. Component Design Principles

Every component should aim for:

High Cohesion

Related responsibilities should stay together.

Low Coupling

Unnecessary dependencies should be avoided.

Clear Ownership

Every business responsibility should have a clear owner.

Reusability

Shared behavior should be reusable when appropriate.

Testability

Components should be testable independently.

Security

Sensitive operations should enforce authorization.

Replaceability

External infrastructure dependencies should be replaceable where practical.

86. Component Anti-Patterns

Avoid:

❌ God Component
❌ Huge Controller
❌ Business Logic in Frontend
❌ Business Logic in Controller
❌ Duplicate Authentication
❌ Duplicate Notification Logic
❌ Direct Cross-Module Database Access
❌ Shared Global State Without Boundaries
❌ Hard-Coded Secrets
❌ Hard-Coded Environment Configuration
❌ Public Sensitive Documents
87. Component Relationship With Documentation

Each component should have supporting documentation when necessary.

Component
   │
   ├── Requirement
   ├── User Story
   ├── Workflow
   ├── API
   ├── Database
   └── Tests

This creates traceability throughout the project.

88. Complete Component Inventory

The major TechCare components are:

Frontend
API
Middleware
Exception Handling
Authentication
Registration
Identity
Authorization
Patient
Doctor
Nurse
Pharmacy
Pharmacy Branch
Medicine
Laboratory
Laboratory Test
Blood Donor
Blood Matching
Admin
Booking
Availability
Appointment
Healthcare Service
Location
Geocoding
Distance Calculation
Search & Discovery
Search Filtering
Search Ranking
Medical Records
Consultation
Medication
Payment
Pricing
Financial Transaction
Withdrawal
Notification
Notification Delivery
Ratings & Reviews
Complaints
Provider Verification
File Management
File Validation
File Storage
Profile
Account Status
Session Management
OTP
Email
SMS
Database
EF Core
Repository / Data Access
Caching
Logging
Audit
Monitoring
Configuration
Validation
Business Rules
89. Complete Component Ecosystem
                              TECHCARE
                                  │
                 ┌────────────────┼────────────────┐
                 │                │                │
                 ▼                ▼                ▼
             Frontend            API        External Services
                 │                │                │
                 └────────────────┼────────────────┘
                                  ▼
                       Platform Services
                                  │
       ┌──────────────┬───────────┼───────────┬──────────────┐
       ▼              ▼           ▼           ▼              ▼
Authentication   Authorization  Location   Search      Notification
       │              │           │           │              │
       └──────────────┼───────────┼───────────┼──────────────┘
                      ▼
                 Business Modules
                      │
       ┌──────────────┼─────────────────────────────────────┐
       │              │              │            │          │
       ▼              ▼              ▼            ▼          ▼
    Patient         Doctor         Nurse      Pharmacy      Lab
       │              │              │            │          │
       └──────────────┼──────────────┼────────────┼──────────┘
                      │
                      ▼
                Blood Donor / Admin
                      │
                      ▼
                Shared Workflows
                      │
       ┌──────────────┼──────────────┬────────────────┐
       ▼              ▼              ▼                ▼
    Booking        Medical        Payment         Verification
                   Records
       │              │              │                │
       └──────────────┼──────────────┼────────────────┘
                      ▼
                 Infrastructure
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
    Database       Storage      External APIs
90. Final Component Principle

The core component principle of TechCare is:

Every component should have one clear responsibility, a defined boundary, controlled dependencies, explicit inputs and outputs, and a clear relationship with the rest of the platform.

The goal is not to create as many components as possible.

The goal is to create clear and meaningful boundaries that make TechCare easier to understand, develop, test, and maintain.

🩺 TechCare

Clear components. Clear responsibilities. Controlled dependencies. One connected platform.