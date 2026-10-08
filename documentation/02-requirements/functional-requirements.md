# ⚙️ TechCare — Functional Requirements

> **This document defines what the TechCare system must do from a functional and business perspective.**
>
> Each requirement describes a specific behavior, capability, or operation that the system is expected to support.

---

# 1. Document Purpose

The purpose of this document is to define the functional requirements of the TechCare healthcare platform.

Functional requirements describe:

- What the system must do.
- What users can do.
- How the system should respond to user actions.
- How different modules interact.
- What business rules should be enforced.
- What should happen under successful and unsuccessful conditions.
- Which role is allowed to perform each operation.

This document is intended to be a shared reference for:

- Developers
- Frontend developers
- Backend developers
- Testers
- Project members
- Reviewers
- Future contributors

---

# 2. Requirement Definition

A functional requirement defines a behavior that the system must provide.

Each requirement follows the general structure:

```text
Requirement ID
      ↓
Requirement Name
      ↓
Actor
      ↓
Description
      ↓
Business Rules
      ↓
Expected System Behavior
3. Requirement ID Convention

Requirements use the following ID pattern:

FR-AUTH-XXX
FR-PAT-XXX
FR-DOC-XXX
FR-NUR-XXX
FR-PHA-XXX
FR-LAB-XXX
FR-BLD-XXX
FR-ADM-XXX
FR-BOOK-XXX
FR-LOC-XXX
FR-SEARCH-XXX
FR-MED-XXX
FR-PAY-XXX
FR-NOT-XXX
FR-RATE-XXX
FR-COMP-XXX
FR-VER-XXX
FR-FILE-XXX

Where:

Prefix	Module
AUTH	Authentication & Authorization
PAT	Patient
DOC	Doctor
NUR	Nurse
PHA	Pharmacy
LAB	Laboratory
BLD	Blood Donor
ADM	Admin
BOOK	Booking
LOC	Location
SEARCH	Search & Discovery
MED	Medical Information
PAY	Payment & Financial
NOT	Notifications
RATE	Ratings & Reviews
COMP	Complaints
VER	Provider Verification
FILE	File and Document Management
4. Actors

The main actors in the system are:

Visitor
Patient
Doctor
Nurse
Pharmacist / Pharmacy
Laboratory
Blood Donor
Admin
System
5. Global Functional Rules
FR-SYS-001 — Role-Aware Platform

Actor: System

The system shall identify the authenticated user's role and apply the appropriate permissions and available functionality.

FR-SYS-002 — Authentication Before Protected Operations

Actor: System

The system shall require authentication before allowing access to protected resources.

FR-SYS-003 — Authorization Before Sensitive Operations

Actor: System

The system shall verify that the authenticated user has permission to perform the requested operation.

FR-SYS-004 — Resource-Level Authorization

Actor: System

The system shall verify whether the authenticated user is authorized to access or modify the requested resource.

Example:

Patient A
   ↓
Own Medical Record
   ✅ Allowed

Patient A
   ↓
Patient B Medical Record
   ❌ Denied
FR-SYS-005 — Account Status Enforcement

Actor: System

The system shall prevent suspended, blocked, or otherwise unauthorized accounts from performing restricted operations according to account status rules.

FR-SYS-006 — Validation

Actor: System

The system shall validate submitted information before processing it.

Validation shall be performed server-side even when client-side validation exists.

FR-SYS-007 — Consistent Error Handling

Actor: System

The system shall return an appropriate response when an operation fails.

Examples:

Invalid input.
Unauthorized access.
Forbidden operation.
Resource not found.
Duplicate data.
Invalid state transition.
Business rule violation.
6. Authentication & Authorization Requirements
6.1 Registration
FR-AUTH-001 — Open Registration Page

Actor: Visitor

The system shall allow unauthenticated visitors to access the registration process.

FR-AUTH-002 — Role Selection During Registration

Actor: Visitor

The system shall allow a new user to select a supported registration role.

Supported public roles:

Patient
Doctor
Nurse
Pharmacist / Pharmacy
Laboratory
Blood Donor
FR-AUTH-003 — Role-Specific Registration Form

Actor: System

The system shall display a registration flow appropriate to the selected role.

FR-AUTH-004 — Patient Registration

Actor: Visitor

The system shall allow a user to register as a Patient.

The registration process shall collect the information required by the patient requirements.

FR-AUTH-005 — Doctor Registration

Actor: Visitor

The system shall allow a user to register as a Doctor.

The registration process shall collect personal, professional, and verification-related information.

FR-AUTH-006 — Nurse Registration

Actor: Visitor

The system shall allow a user to register as a Nurse.

The registration process shall collect personal, professional, service, and verification-related information.

FR-AUTH-007 — Pharmacy Registration

Actor: Visitor

The system shall allow a user to register as a Pharmacist / Pharmacy provider.

The registration process shall collect the required pharmacist and pharmacy information.

FR-AUTH-008 — Laboratory Registration

Actor: Visitor

The system shall allow a user to register a Laboratory provider account.

The registration process shall collect the required laboratory and responsible-person information.

FR-AUTH-009 — Blood Donor Registration

Actor: Visitor

The system shall allow a user to register as a Blood Donor.

FR-AUTH-010 — Admin Public Registration Restriction

Actor: System

The system shall not provide normal public registration for the Admin role.

6.2 Login
FR-AUTH-011 — Common Login

Actor: Registered User

The system shall provide one common login interface for registered users.

The user shall not be required to select their role during login.

FR-AUTH-012 — Credential Validation

Actor: System

The system shall validate submitted login credentials.

FR-AUTH-013 — Invalid Login

Actor: System

The system shall reject invalid credentials and return an appropriate error.

FR-AUTH-014 — Account Status Check During Login

Actor: System

The system shall verify the account status before completing login.

FR-AUTH-015 — Role Identification

Actor: System

After successful authentication, the system shall identify the user's role and permissions.

FR-AUTH-016 — Role-Based Navigation

Actor: System

The system shall provide access to the appropriate role-specific area after successful authentication.

Example:

Doctor Login
    ↓
Doctor Area

Patient Login
    ↓
Patient Area
6.3 Logout
FR-AUTH-017 — Logout

Actor: Authenticated User

The system shall allow the user to log out.

FR-AUTH-018 — Session / Token Revocation

Actor: System

The system shall invalidate or revoke the relevant authentication state according to the token/session strategy.

FR-AUTH-019 — Logout All Devices

Actor: Authenticated User

The system shall support ending active sessions on multiple devices where session management supports this feature.

6.4 Verification
FR-AUTH-020 — OTP Generation

Actor: System

The system shall generate a verification OTP for supported verification workflows.

FR-AUTH-021 — OTP Delivery

Actor: System

The system shall deliver the OTP through the configured verification channel.

FR-AUTH-022 — OTP Validation

Actor: User

The system shall allow the user to submit the received OTP.

FR-AUTH-023 — OTP Expiration

Actor: System

The system shall reject expired OTPs.

FR-AUTH-024 — Invalid OTP

Actor: System

The system shall reject invalid OTP values.

FR-AUTH-025 — Verification Status

Actor: System

The system shall maintain the verification status of the relevant account or verification method.

FR-AUTH-026 — Email Verification

Actor: User

The system shall support email verification where email verification is part of the configured authentication flow.

FR-AUTH-027 — Phone Verification

Actor: User

The system shall support phone verification where phone verification is part of the configured authentication flow.

6.5 Password Management
FR-AUTH-028 — Forgot Password

Actor: User

The system shall allow users to initiate a password recovery process.

FR-AUTH-029 — Password Recovery Verification

Actor: System

The system shall verify the user's identity before allowing password reset.

FR-AUTH-030 — Reset Password

Actor: User

The system shall allow an authorized user to create a new password.

FR-AUTH-031 — Change Password

Actor: Authenticated User

The system shall allow an authenticated user to change their password.

FR-AUTH-032 — Password Confirmation

Actor: System

The system shall require appropriate confirmation when setting or changing a password.

6.6 Authorization
FR-AUTH-033 — Role-Based Permissions

Actor: System

The system shall enforce permissions according to the authenticated user's role.

FR-AUTH-034 — Protected Administrative Operations

Actor: System

The system shall restrict administrative operations to authorized Admin accounts.

FR-AUTH-035 — Protected Provider Operations

Actor: System

The system shall restrict provider-specific operations to the relevant provider role.

7. Patient Requirements
7.1 Patient Profile
FR-PAT-001 — View Patient Profile

Actor: Patient

The system shall allow a patient to view their own profile.

FR-PAT-002 — Edit Patient Profile

Actor: Patient

The system shall allow a patient to update editable profile information.

FR-PAT-003 — Manage Contact Information

Actor: Patient

The system shall allow the patient to manage supported contact information.

FR-PAT-004 — Manage Address

Actor: Patient

The system shall allow the patient to create or update their address information.

FR-PAT-005 — Manage Location

Actor: Patient

The system shall allow the patient to provide and manage supported location information.

7.2 Patient Healthcare Information
FR-PAT-006 — Medical Profile

Actor: Patient

The system shall allow the patient to manage supported healthcare-related profile information.

FR-PAT-007 — Medical History

Actor: Patient

The system shall allow the patient to maintain supported medical history information.

FR-PAT-008 — Previous Conditions

Actor: Patient

The system shall allow the patient to record supported previous health conditions.

FR-PAT-009 — Current Medications

Actor: Patient

The system shall allow the patient to maintain supported information about current medications.

FR-PAT-010 — View Medical Information

Actor: Authorized User

The system shall allow authorized users to access relevant patient healthcare information according to authorization rules.

7.3 Doctor Discovery
FR-PAT-011 — Search Doctors

Actor: Patient

The system shall allow patients to search for doctors.

FR-PAT-012 — Filter Doctors by Specialty

Actor: Patient

The system shall allow patients to filter doctors by specialty.

FR-PAT-013 — Filter Doctors by Location

Actor: Patient

The system shall allow patients to filter or prioritize doctors based on location.

FR-PAT-014 — Filter Doctors by Distance

Actor: Patient

The system shall allow patients to filter or sort doctors based on distance where location data is available.

FR-PAT-015 — Filter Doctors by Rating

Actor: Patient

The system shall allow patients to filter or sort doctors based on rating where supported.

FR-PAT-016 — Filter Doctors by Price

Actor: Patient

The system shall allow patients to filter doctors based on applicable service price.

FR-PAT-017 — Filter Doctors by Availability

Actor: Patient

The system shall allow patients to discover doctors based on supported availability information.

FR-PAT-018 — View Doctor Profile

Actor: Patient

The system shall allow the patient to view the public provider information permitted by the platform.

7.4 Nurse Discovery
FR-PAT-019 — Search Nurses

Actor: Patient

The system shall allow patients to search for nurses.

FR-PAT-020 — Filter Nurses by Service

Actor: Patient

The system shall allow patients to filter nurses according to supported service types.

FR-PAT-021 — Filter Nurses by Location

Actor: Patient

The system shall allow patients to discover nurses based on location.

FR-PAT-022 — Filter Nurses by Distance

Actor: Patient

The system shall allow patients to filter or sort nurses according to distance where supported.

FR-PAT-023 — Filter Nurses by Rating

Actor: Patient

The system shall allow patients to filter or sort nurses according to rating.

FR-PAT-024 — Filter Nurses by Price

Actor: Patient

The system shall allow patients to filter nurses according to applicable pricing.

FR-PAT-025 — View Nurse Profile

Actor: Patient

The system shall allow patients to view permitted nurse profile information.

7.5 Pharmacy and Medicine Discovery
FR-PAT-026 — Search Medicine

Actor: Patient

The system shall allow patients to search for supported medicines.

FR-PAT-027 — Find Nearby Pharmacies

Actor: Patient

The system shall allow patients to discover relevant nearby pharmacy branches.

FR-PAT-028 — View Medicine Availability

Actor: Patient

The system shall display supported medicine availability information.

FR-PAT-029 — View Pharmacy Branch Information

Actor: Patient

The system shall allow patients to view permitted pharmacy branch information.

7.6 Laboratory Discovery
FR-PAT-030 — Search Laboratories

Actor: Patient

The system shall allow patients to search for laboratories.

FR-PAT-031 — Search Laboratory Tests

Actor: Patient

The system shall allow patients to search for supported laboratory tests or services.

FR-PAT-032 — Filter Laboratories by Location

Actor: Patient

The system shall allow patients to discover laboratories according to location.

FR-PAT-033 — Filter Laboratories by Availability

Actor: Patient

The system shall allow patients to filter laboratories according to supported availability information.

7.7 Patient Requests and Bookings
FR-PAT-034 — Create Service Request

Actor: Patient

The system shall allow a patient to submit a healthcare service request to an eligible provider.

FR-PAT-035 — Select Provider

Actor: Patient

The system shall allow a patient to select a provider before submitting a provider-specific request.

FR-PAT-036 — Select Service

Actor: Patient

The system shall allow the patient to select an applicable healthcare service.

FR-PAT-037 — Select Time Slot

Actor: Patient

The system shall allow the patient to select an available time slot where scheduling is required.

FR-PAT-038 — View Price Before Confirmation

Actor: Patient

The system shall display applicable pricing information before confirmation whenever the business flow supports it.

FR-PAT-039 — Request Status

Actor: Patient

The system shall allow the patient to view the current status of their request.

FR-PAT-040 — Request History

Actor: Patient

The system shall allow the patient to view their previous requests.

FR-PAT-041 — Cancel Request

Actor: Patient

The system shall allow cancellation when the request is in a state that permits cancellation.

8. Doctor Requirements
8.1 Doctor Profile
FR-DOC-001 — View Doctor Profile

Actor: Doctor

The system shall allow doctors to view their own professional profile.

FR-DOC-002 — Edit Doctor Profile

Actor: Doctor

The system shall allow doctors to update editable professional information.

FR-DOC-003 — Manage Specialty

Actor: Doctor

The system shall allow doctors to specify supported specialty information.

FR-DOC-004 — Manage Experience

Actor: Doctor

The system shall allow doctors to provide professional experience information.

FR-DOC-005 — Manage Bio

Actor: Doctor

The system shall allow doctors to manage supported profile biography information.

FR-DOC-006 — Manage Services

Actor: Doctor

The system shall allow doctors to manage the services they provide.

FR-DOC-007 — Manage Pricing

Actor: Doctor

The system shall allow doctors to manage supported service pricing.

FR-DOC-008 — Manage Availability

Actor: Doctor

The system shall allow doctors to define and update their availability.

8.2 Doctor Requests
FR-DOC-009 — View Incoming Requests

Actor: Doctor

The system shall allow doctors to view requests assigned or sent to them.

FR-DOC-010 — View Request Details

Actor: Doctor

The system shall allow doctors to view permitted information about a request.

FR-DOC-011 — Accept Request

Actor: Doctor

The system shall allow doctors to accept an eligible request.

FR-DOC-012 — Reject Request

Actor: Doctor

The system shall allow doctors to reject an eligible request.

FR-DOC-013 — Prevent Invalid State Changes

Actor: System

The system shall prevent doctors from performing actions that are not valid for the current request state.

8.3 Doctor Consultation
FR-DOC-014 — Start Consultation

Actor: Doctor

The system shall allow an eligible doctor to start the relevant consultation workflow.

FR-DOC-015 — Record Consultation Information

Actor: Doctor

The system shall allow the doctor to record supported consultation information.

FR-DOC-016 — Record Diagnosis

Actor: Doctor

The system shall allow the doctor to record diagnosis information where supported.

FR-DOC-017 — Record Treatment

Actor: Doctor

The system shall allow the doctor to record treatment information where supported.

FR-DOC-018 — Record Medication Information

Actor: Doctor

The system shall allow authorized doctors to record medication information where applicable.

FR-DOC-019 — Complete Consultation

Actor: Doctor

The system shall allow the doctor to complete a consultation according to the defined workflow.

8.4 Doctor Financial Management
FR-DOC-020 — View Earnings

Actor: Doctor

The system shall allow the doctor to view supported earnings information.

FR-DOC-021 — View Transactions

Actor: Doctor

The system shall allow the doctor to view relevant financial transactions.

FR-DOC-022 — Request Withdrawal

Actor: Doctor

The system shall allow the doctor to submit a withdrawal request when eligible.

9. Nurse Requirements
9.1 Nurse Profile
FR-NUR-001 — View Nurse Profile

Actor: Nurse

The system shall allow nurses to view their own professional profile.

FR-NUR-002 — Edit Nurse Profile

Actor: Nurse

The system shall allow nurses to edit supported professional information.

FR-NUR-003 — Manage Skills

Actor: Nurse

The system shall allow nurses to maintain supported skills information.

FR-NUR-004 — Manage Services

Actor: Nurse

The system shall allow nurses to define supported nursing services.

FR-NUR-005 — Manage Service Area

Actor: Nurse

The system shall allow nurses to define their supported service area.

FR-NUR-006 — Manage Pricing

Actor: Nurse

The system shall allow nurses to manage supported pricing.

FR-NUR-007 — Manage Availability

Actor: Nurse

The system shall allow nurses to manage service availability.

9.2 Nurse Requests
FR-NUR-008 — View Incoming Requests

Actor: Nurse

The system shall allow nurses to view incoming eligible service requests.

FR-NUR-009 — View Request Details

Actor: Nurse

The system shall allow nurses to view permitted request information.

FR-NUR-010 — Accept Request

Actor: Nurse

The system shall allow nurses to accept eligible requests.

FR-NUR-011 — Reject Request

Actor: Nurse

The system shall allow nurses to reject eligible requests.

9.3 Nurse Visits
FR-NUR-012 — View Scheduled Visits

Actor: Nurse

The system shall allow nurses to view scheduled visits.

FR-NUR-013 — Start Nursing Service

Actor: Nurse

The system shall allow the nurse to start an eligible service visit.

FR-NUR-014 — Complete Nursing Service

Actor: Nurse

The system shall allow the nurse to complete an eligible service.

9.4 Nurse Financials
FR-NUR-015 — View Earnings

Actor: Nurse

The system shall allow nurses to view supported earnings information.

FR-NUR-016 — Request Withdrawal

Actor: Nurse

The system shall allow eligible nurses to submit withdrawal requests.

10. Pharmacy Requirements
10.1 Pharmacy Profile
FR-PHA-001 — Manage Pharmacy Profile

Actor: Pharmacist / Pharmacy

The system shall allow authorized pharmacy users to manage supported pharmacy information.

FR-PHA-002 — Manage Contact Information

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to manage pharmacy contact information.

FR-PHA-003 — Manage Working Hours

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to manage supported working hours.

10.2 Branch Management
FR-PHA-004 — Create Pharmacy Branch

Actor: Pharmacist / Pharmacy

The system shall allow authorized pharmacy users to create supported branches.

FR-PHA-005 — Edit Pharmacy Branch

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to update branch information.

FR-PHA-006 — Manage Branch Location

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to manage branch location information.

FR-PHA-007 — Manage Branch Working Hours

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to manage branch working hours where applicable.

10.3 Medicine Management
FR-PHA-008 — Add Medicine

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to add supported medicine information.

FR-PHA-009 — Update Medicine Availability

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to update medicine availability.

FR-PHA-010 — Manage Inventory Status

Actor: Pharmacist / Pharmacy

The system shall allow authorized users to manage supported inventory/status information.

FR-PHA-011 — Branch-Level Availability

Actor: System

The system shall maintain medicine availability at the appropriate pharmacy branch level when branch-level inventory is supported.

11. Laboratory Requirements
11.1 Laboratory Profile
FR-LAB-001 — Manage Laboratory Profile

Actor: Laboratory

The system shall allow authorized laboratory users to manage laboratory profile information.

FR-LAB-002 — Manage Laboratory Location

Actor: Laboratory

The system shall allow authorized laboratory users to manage laboratory location information.

FR-LAB-003 — Manage Working Hours

Actor: Laboratory

The system shall allow authorized laboratory users to manage supported working hours.

11.2 Laboratory Services
FR-LAB-004 — Add Laboratory Test

Actor: Laboratory

The system shall allow authorized laboratory users to add supported tests.

FR-LAB-005 — Update Laboratory Test

Actor: Laboratory

The system shall allow authorized laboratory users to update supported test information.

FR-LAB-006 — Manage Laboratory Services

Actor: Laboratory

The system shall allow authorized laboratory users to manage supported laboratory services.

FR-LAB-007 — Manage Test Availability

Actor: Laboratory

The system shall allow authorized users to manage test availability where supported.

11.3 Laboratory Requests
FR-LAB-008 — Receive Service Request

Actor: Laboratory

The system shall allow laboratories to receive eligible service requests.

FR-LAB-009 — Manage Laboratory Request

Actor: Laboratory

The system shall allow authorized laboratory users to process eligible requests according to the laboratory workflow.

FR-LAB-010 — Result Availability Workflow

Actor: Laboratory

The system shall support the defined workflow for making a laboratory result available to an authorized patient where this functionality is included in the active scope.

12. Blood Donor Requirements
FR-BLD-001 — Manage Donor Profile

Actor: Blood Donor

The system shall allow donors to manage supported donor profile information.

FR-BLD-002 — Manage Blood Type

Actor: Blood Donor

The system shall allow the donor to provide supported blood type information.

FR-BLD-003 — Manage Availability

Actor: Blood Donor

The system shall allow donors to manage supported availability information.

FR-BLD-004 — Blood Donation Request

Actor: Authorized User / System

The system shall support creation and processing of supported blood donation requests.

FR-BLD-005 — Donor Matching

Actor: System

The system shall identify potential donors using supported matching criteria.

Potential criteria may include:

Blood information.
Compatibility rules.
Location/area.
Availability.
FR-BLD-006 — Donor Notification

Actor: System

The system shall notify relevant donors according to the supported donation workflow.

FR-BLD-007 — Donor Privacy

Actor: System

The system shall prevent unnecessary public exposure of sensitive donor information.

13. Admin Requirements
13.1 User Management
FR-ADM-001 — View Users

Actor: Admin

The system shall allow authorized administrators to view users.

FR-ADM-002 — Search Users

Actor: Admin

The system shall allow administrators to search for users.

FR-ADM-003 — View User Account State

Actor: Admin

The system shall allow authorized administrators to view relevant account status information.

FR-ADM-004 — Manage User Account Status

Actor: Admin

The system shall allow authorized administrators to perform supported account status actions.

13.2 Provider Verification
FR-ADM-005 — View Verification Requests

Actor: Admin

The system shall allow administrators to view provider verification requests.

FR-ADM-006 — View Submitted Documents

Actor: Admin

The system shall allow authorized administrators to review submitted provider documents.

FR-ADM-007 — Approve Provider

Actor: Admin

The system shall allow an authorized administrator to approve an eligible provider.

FR-ADM-008 — Reject Provider

Actor: Admin

The system shall allow an authorized administrator to reject a provider verification request.

FR-ADM-009 — Request Additional Information

Actor: Admin

The system shall allow the administrator to request additional information or documents where supported.

13.3 Complaints
FR-ADM-010 — View Complaints

Actor: Admin

The system shall allow administrators to view complaints.

FR-ADM-011 — Review Complaint

Actor: Admin

The system shall allow administrators to review complaint details.

FR-ADM-012 — Update Complaint Status

Actor: Admin

The system shall allow authorized administrators to update complaint status.

FR-ADM-013 — Resolve Complaint

Actor: Admin

The system shall allow administrators to record or apply supported complaint resolutions.

13.4 Ratings and Reviews Moderation
FR-ADM-014 — Review Reported Content

Actor: Admin

The system shall allow administrators to review reported ratings and reviews where moderation is supported.

FR-ADM-015 — Apply Moderation Action

Actor: Admin

The system shall allow authorized administrators to apply supported moderation actions.

14. Booking & Appointment Requirements
FR-BOOK-001 — Create Booking Request

Actor: Patient

The system shall allow a patient to create an eligible booking request.

FR-BOOK-002 — Validate Provider Eligibility

Actor: System

The system shall verify that the selected provider is eligible to receive the requested booking.

FR-BOOK-003 — Validate Availability

Actor: System

The system shall verify provider availability before confirming a booking slot.

FR-BOOK-004 — Prevent Double Booking

Actor: System

The system shall prevent conflicting bookings for the same provider and time slot according to scheduling rules.

FR-BOOK-005 — Create Pending Request

Actor: System

The system shall create the booking request with the appropriate initial state.

FR-BOOK-006 — Notify Provider

Actor: System

The system shall notify the relevant provider when a new request is created.

FR-BOOK-007 — Accept Booking

Actor: Provider

The system shall allow the relevant provider to accept an eligible request.

FR-BOOK-008 — Reject Booking

Actor: Provider

The system shall allow the relevant provider to reject an eligible request.

FR-BOOK-009 — Expire Request

Actor: System

The system shall automatically expire requests that exceed the configured response period without the required provider response.

FR-BOOK-010 — Update Booking Status

Actor: System

The system shall update the booking status according to valid state transitions.

FR-BOOK-011 — Cancel Booking

Actor: Authorized User

The system shall allow cancellation when the booking state and business rules permit cancellation.

FR-BOOK-012 — Complete Service

Actor: Authorized Provider

The system shall allow an eligible provider to mark a service as completed.

FR-BOOK-013 — Record Completion Time

Actor: System

The system shall record relevant completion information for completed services.

15. Availability Requirements
FR-BOOK-014 — Create Availability

Actor: Provider

The system shall allow eligible providers to define availability.

FR-BOOK-015 — Update Availability

Actor: Provider

The system shall allow providers to update availability.

FR-BOOK-016 — Remove Availability

Actor: Provider

The system shall allow providers to remove or disable available periods where the business rules permit it.

FR-BOOK-017 — Respect Unavailable Periods

Actor: System

The system shall prevent new bookings from being created for periods that are unavailable.

16. Location & Geo Requirements
FR-LOC-001 — Capture Current Location

Actor: Patient / Provider

The system shall support capturing current device location when the user grants the required permission.

FR-LOC-002 — Manual Address

Actor: User

The system shall allow users to enter a location manually.

FR-LOC-003 — Save Location

Actor: User

The system shall allow users to save supported address/location information.

FR-LOC-004 — Store Geographic Coordinates

Actor: System

The system shall store supported geographic coordinates when geographic location is part of the relevant business process.

FR-LOC-005 — Calculate Distance

Actor: System

The system shall calculate or retrieve supported distance between relevant locations.

FR-LOC-006 — Nearby Provider Discovery

Actor: System

The system shall support discovery of nearby providers using location information.

FR-LOC-007 — Service Radius

Actor: Provider / System

The system shall support provider service radius rules where applicable.

FR-LOC-008 — Service Area Validation

Actor: System

The system shall determine whether a provider can serve a relevant location according to service-area rules.

FR-LOC-009 — Location Privacy

Actor: System

The system shall restrict access to precise location information according to authorization and privacy rules.

17. Search & Discovery Requirements
FR-SEARCH-001 — Search Providers

Actor: User

The system shall allow users to search supported healthcare providers.

FR-SEARCH-002 — Search Services

Actor: User

The system shall allow users to search supported healthcare services.

FR-SEARCH-003 — Filter Results

Actor: User

The system shall allow users to filter search results using supported filters.

FR-SEARCH-004 — Sort Results

Actor: User

The system shall allow users to sort or rank supported search results.

FR-SEARCH-005 — Location-Aware Results

Actor: System

The system shall use available location information when location-aware search is requested.

FR-SEARCH-006 — Search by Specialty

Actor: User

The system shall support specialty-based provider search where relevant.

FR-SEARCH-007 — Search by Service

Actor: User

The system shall support service-based discovery.

FR-SEARCH-008 — Search by Medicine

Actor: User

The system shall support medicine search where pharmacy search is applicable.

FR-SEARCH-009 — Search by Laboratory Test

Actor: User

The system shall support discovery of laboratories or services based on supported laboratory test information.

FR-SEARCH-010 — Provider Profile Result

Actor: System

The system shall provide access to relevant provider profile information from search results.

FR-SEARCH-011 — Pagination

Actor: System

The system shall support paginated results where result volume requires pagination.

18. Medical Information Requirements
FR-MED-001 — Create Medical Profile

Actor: Patient

The system shall allow the patient to maintain a supported medical profile.

FR-MED-002 — Add Medical History

Actor: Patient / Authorized Provider

The system shall allow authorized users to add supported medical history information.

FR-MED-003 — Update Medical Information

Actor: Authorized User

The system shall allow authorized users to update medical information according to access rules.

FR-MED-004 — Record Consultation

Actor: Doctor

The system shall allow an eligible doctor to create a consultation record.

FR-MED-005 — Associate Consultation with Patient

Actor: System

The system shall associate the consultation with the correct patient and relevant encounter.

FR-MED-006 — Record Diagnosis

Actor: Doctor

The system shall allow diagnosis information to be recorded where applicable.

FR-MED-007 — Record Treatment

Actor: Doctor

The system shall allow treatment information to be recorded where applicable.

FR-MED-008 — Record Medication Information

Actor: Authorized Doctor

The system shall allow supported medication information to be recorded.

FR-MED-009 — Medical Record Access Control

Actor: System

The system shall prevent unauthorized users from accessing patient medical information.

FR-MED-010 — Medical Record History

Actor: Authorized User

The system shall allow authorized users to view supported historical medical records.

19. Payment & Financial Requirements
FR-PAY-001 — Calculate Service Price

Actor: System

The system shall calculate the applicable service price according to configured pricing rules.

FR-PAY-002 — Base Service Price

Actor: System

The system shall support a configured base service price.

FR-PAY-003 — Distance-Based Pricing

Actor: System

The system shall support distance-based charges where applicable.

FR-PAY-004 — Display Applicable Charges

Actor: Patient

The system shall display supported pricing components before confirmation where the workflow requires it.

FR-PAY-005 — Create Financial Transaction

Actor: System

The system shall create a financial transaction record when an applicable financial event occurs.

FR-PAY-006 — Payment Status

Actor: System

The system shall maintain the status of a supported payment transaction.

FR-PAY-007 — Provider Earnings

Actor: Provider

The system shall allow eligible providers to view supported earnings.

FR-PAY-008 — Withdrawal Request

Actor: Provider

The system shall allow eligible providers to submit withdrawal requests.

FR-PAY-009 — Withdrawal Status

Actor: System

The system shall track supported withdrawal statuses.

FR-PAY-010 — Financial History

Actor: Provider

The system shall allow providers to view supported financial history.

20. Notification Requirements
FR-NOT-001 — Create Notification

Actor: System

The system shall create notifications for configured important platform events.

FR-NOT-002 — Booking Notification

Actor: System

The system shall notify relevant users about important booking events.

FR-NOT-003 — Request Notification

Actor: System

The system shall notify providers about relevant incoming service requests.

FR-NOT-004 — Verification Notification

Actor: System

The system shall notify providers about relevant verification status changes.

FR-NOT-005 — Payment Notification

Actor: System

The system shall notify relevant users about supported payment and financial events.

FR-NOT-006 — Appointment Reminder

Actor: System

The system shall support appointment reminders where enabled by the platform.

FR-NOT-007 — Notification History

Actor: User

The system shall allow users to view their supported notification history.

FR-NOT-008 — Notification Read Status

Actor: User

The system shall allow supported notifications to be marked as read.

21. Ratings & Reviews Requirements
FR-RATE-001 — Rate Completed Service

Actor: Patient

The system shall allow eligible patients to rate a completed service.

FR-RATE-002 — Written Review

Actor: Patient

The system shall allow eligible patients to submit a written review where supported.

FR-RATE-003 — Prevent Invalid Ratings

Actor: System

The system shall prevent ratings from being created when the required eligibility conditions are not met.

FR-RATE-004 — Provider Rating Summary

Actor: System

The system shall calculate or maintain supported provider rating information.

FR-RATE-005 — Review Visibility

Actor: System

The system shall display reviews according to visibility and moderation rules.

FR-RATE-006 — Rating Moderation

Actor: Admin

The system shall allow authorized administrators to moderate reported ratings or reviews where supported.

22. Complaint Requirements
FR-COMP-001 — Create Complaint

Actor: User

The system shall allow users to create complaints for supported categories.

FR-COMP-002 — Select Complaint Category

Actor: User

The system shall allow the user to select an appropriate complaint category.

FR-COMP-003 — Complaint Description

Actor: User

The system shall allow the user to provide a complaint description.

FR-COMP-004 — Complaint Status

Actor: System

The system shall maintain the complaint status.

Possible states may include:

Created
Under Review
Investigating
Resolved
Closed
FR-COMP-005 — Admin Complaint Review

Actor: Admin

The system shall allow authorized administrators to review complaints.

FR-COMP-006 — Complaint Resolution

Actor: Admin

The system shall allow authorized administrators to record supported complaint resolution information.

FR-COMP-007 — Complaint History

Actor: Authorized User

The system shall allow users to view complaints they are authorized to access.

23. Provider Verification Requirements
FR-VER-001 — Provider Verification Status

Actor: System

The system shall maintain a verification status for supported providers.

FR-VER-002 — Upload Verification Documents

Actor: Provider

The system shall allow providers to upload supported verification documents.

FR-VER-003 — Verification Pending

Actor: System

The system shall place newly submitted verification data into the appropriate verification state until review is complete.

FR-VER-004 — Admin Review

Actor: Admin

The system shall allow authorized administrators to review provider verification information.

FR-VER-005 — Approve Verification

Actor: Admin

The system shall allow an administrator to approve an eligible provider.

FR-VER-006 — Reject Verification

Actor: Admin

The system shall allow an administrator to reject an eligible verification request.

FR-VER-007 — Additional Information Request

Actor: Admin

The system shall allow an administrator to request additional information or documents where supported.

FR-VER-008 — Provider Activation

Actor: System

The system shall activate provider functionality according to verification and account rules.

24. File & Document Requirements
FR-FILE-001 — Upload File

Actor: Authorized User

The system shall allow authorized users to upload supported files.

FR-FILE-002 — File Validation

Actor: System

The system shall validate uploaded files according to configured file rules.

FR-FILE-003 — Document Association

Actor: System

The system shall associate uploaded documents with the correct entity and purpose.

FR-FILE-004 — Document Access Control

Actor: System

The system shall restrict document access to authorized users.

FR-FILE-005 — Replace Document

Actor: Authorized User

The system shall allow supported documents to be replaced where the workflow permits replacement.

25. Provider Service Area Requirements
FR-LOC-010 — Define Service Radius

Actor: Doctor / Nurse / Other Eligible Provider

The system shall allow eligible providers to define a supported service radius where applicable.

FR-LOC-011 — Check Service Eligibility

Actor: System

The system shall determine whether the requested patient location is within the provider's supported service area.

FR-LOC-012 — Travel Fee Calculation

Actor: System

The system shall apply supported travel/distance pricing rules when applicable.

26. Service Completion Requirements
FR-BOOK-018 — Service Start

Actor: Authorized Provider

The system shall allow a provider to begin an eligible service.

FR-BOOK-019 — Service Completion

Actor: Authorized Provider

The system shall allow the provider to mark the service as completed.

FR-BOOK-020 — Completion Validation

Actor: System

The system shall validate that the service is in a state that can be completed.

FR-BOOK-021 — Completion Event

Actor: System

The system shall trigger relevant post-completion actions after a successful service completion.

Possible actions include:

Payment update.
Financial record update.
Medical/service record update.
Notification.
Rating eligibility.
27. Cross-Module Requirements

Some requirements affect multiple modules.

FR-CROSS-001 — Authentication Integration

Actor: System

All supported role modules shall use the centralized authentication system.

FR-CROSS-002 — Authorization Integration

Actor: System

All modules shall use centralized authorization policies and role/permission rules.

FR-CROSS-003 — Location Integration

Actor: System

Supported modules shall use the shared location services when location-based operations are required.

FR-CROSS-004 — Notification Integration

Actor: System

Modules shall use the common notification mechanism for configured events.

FR-CROSS-005 — Financial Integration

Actor: System

Paid services shall integrate with the shared financial workflow where applicable.

FR-CROSS-006 — Rating Integration

Actor: System

Eligible completed services shall integrate with the common rating system.

FR-CROSS-007 — Complaint Integration

Actor: System

Supported services and roles shall integrate with the common complaint workflow.

28. Business Rules
FR-BR-001 — One Common Login

All registered users shall use the same common login experience.

Role selection shall not be required during login.

FR-BR-002 — Role Selected During Registration

The public registration flow shall use role selection to determine the appropriate registration process.

FR-BR-003 — Admin Controlled Access

Administrative functionality shall only be available to authorized administrative accounts.

FR-BR-004 — Provider Verification

Provider functionality that requires verification shall only become available according to the provider verification rules.

FR-BR-005 — No Unauthorized Medical Access

A user shall not access medical information unless explicitly authorized.

FR-BR-006 — No Unauthorized Profile Modification

A user shall not modify another user's private profile information unless the role and permission model explicitly allows it.

FR-BR-007 — Booking Availability

A booking shall only be created when the requested service and provider are eligible and available.

FR-BR-008 — Valid Booking State Transitions

A booking status shall only move through valid state transitions.

FR-BR-009 — Request Expiration

Requests that exceed the configured response period may automatically move to an expired state.

FR-BR-010 — Rating Eligibility

A rating shall only be created for an eligible completed service.

FR-BR-011 — Complaint Ownership

Users shall only access complaints they are authorized to view.

FR-BR-012 — Sensitive Location Protection

Precise location information shall only be exposed to authorized functionality.

FR-BR-013 — Donor Privacy

Blood donor information shall only be shared to the minimum extent required by the supported donation workflow.

FR-BR-014 — Branch-Level Pharmacy Availability

Where branch-level inventory is supported, medicine availability shall be associated with the appropriate branch.

29. Main Booking Statuses

The platform may use the following general booking states:

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

The exact states can vary by service type.

30. Provider Verification Statuses

The platform may use:

Pending
Approved
Rejected
Additional Information Required
Suspended

The exact final state model is determined by the provider verification workflow.

31. Complaint Statuses

The platform may use:

Created
Under Review
Investigating
Resolved
Closed
32. Financial Statuses

Financial entities may use supported states such as:

Pending
Confirmed
Completed
Failed
Cancelled
Settled
Withdrawn

The final states depend on the financial implementation.

33. Functional Dependency Overview

Many features depend on shared services.

                         Authentication
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
       Patient             Providers              Admin
          │                    │                    │
          └──────────────┬─────┴──────────────┬─────┘
                         ▼                    ▼
                    Location              Verification
                         │
                         ▼
                   Search & Discovery
                         │
                         ▼
                       Booking
                         │
              ┌──────────┼───────────┐
              ▼          ▼           ▼
          Doctor       Nurse      Laboratory
              │          │           │
              └──────────┼───────────┘
                         ▼
                      Service
                         │
          ┌──────────────┼───────────────┐
          ▼              ▼               ▼
       Medical        Payment        Notification
       Record
          │
          └──────────────┬────────────────┘
                         ▼
                       Rating
                         │
                         ▼
                     Complaint
34. Functional Requirements Summary

The TechCare functional requirements cover the following areas:

Authentication
Authorization
Registration
Verification
Patient Management
Doctor Management
Nurse Management
Pharmacy Management
Laboratory Management
Blood Donor Management
Admin Management
Search
Discovery
Location
Availability
Booking
Appointments
Medical Information
Consultation
Payments
Transactions
Withdrawals
Notifications
Ratings
Reviews
Complaints
Provider Verification
File Management
Cross-Module Integration
35. Core End-to-End Functional Flow

The main functional sequence of TechCare is:

Visitor
   ↓
Register
   ↓
Role Selection
   ↓
Role-Specific Registration
   ↓
Verification
   ↓
Login
   ↓
Role Identification
   ↓
Profile
   ↓
Location
   ↓
Search
   ↓
Provider / Service Selection
   ↓
Availability
   ↓
Price
   ↓
Booking / Request
   ↓
Provider Notification
   ↓
Accept / Reject / Expire
   ↓
Appointment
   ↓
Healthcare Service
   ↓
Service Completion
   ↓
Medical / Service Record
   ↓
Payment / Financial Update
   ↓
Notification
   ↓
Rating / Review
   ↓
Complaint if Necessary
36. Definition of Functional Completeness

A functional requirement should not be considered complete merely because a page exists.

For a feature to be considered functionally implemented, the related process should have:

User Action
   ↓
Frontend Handling
   ↓
API Request
   ↓
Authentication
   ↓
Authorization
   ↓
Validation
   ↓
Business Logic
   ↓
Database Operation
   ↓
Response
   ↓
Frontend Update
   ↓
Notification / Side Effects

where applicable.

37. Requirement Traceability

Every major feature should be traceable to:

Requirement
    ↓
User Story
    ↓
Workflow
    ↓
API
    ↓
Database
    ↓
Implementation
    ↓
Test Case

This allows the development team to confirm that the documented requirements are actually implemented and tested.

38. Requirement Change Rule

When an approved functional requirement changes, the following documentation may also need to be updated:

User stories.
Workflows.
Database documentation.
API documentation.
Security documentation.
Testing documentation.
Project scope.
Relevant frontend/backend implementation.

A requirement change should not be treated as isolated when it affects other system components.

39. Functional Requirements Completion Criteria

The functional requirements are considered covered when:

Every major user role has defined functionality.
Major workflows have defined requirements.
Business rules are documented.
Authorization requirements are documented.
Sensitive-data access rules are documented.
Shared services are documented.
Main state transitions are documented.
Cross-module dependencies are identified.
40. Final Requirement Principle

The functional requirements of TechCare should answer one central question:

What must the system do?

Every implementation decision should be traceable back to the documented requirements and the approved business workflows.

🩺 TechCare

From requirements to workflows, from workflows to implementation.