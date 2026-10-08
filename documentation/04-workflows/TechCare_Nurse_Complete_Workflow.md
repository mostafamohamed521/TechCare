# TechCare — Complete Nurse Workflow
## End-to-End Product, Functional, Backend, Frontend, Business Rules, and Security Specification

> **Project:** TechCare  
> **Module:** Nurse / Home Nursing Services  
> **Audience:** Product Owner, Scrum Team, Full-Stack .NET Developers, QA, UI/UX  
> **Status:** Prototype / MVP Specification  
> **Core Principle:** Bring Healthcare Closer to the Patient

---

# 1. Module Overview

The **Nurse Module** allows verified nurses to provide approved nursing services to patients through the TechCare platform.

The nurse workflow covers the complete journey:

```text
Registration
    ↓
Account Verification
    ↓
Admin Verification
    ↓
Profile Completion
    ↓
Service Setup
    ↓
Pricing Setup
    ↓
Availability Setup
    ↓
Search Visibility
    ↓
Receive Booking Request
    ↓
Accept / Reject / Expire
    ↓
Confirmed Appointment
    ↓
Visit Preparation
    ↓
Start Visit
    ↓
Perform Nursing Service
    ↓
Complete Visit
    ↓
Payment / Financial Settlement
    ↓
Rating / Review
    ↓
Complaint Handling if needed
    ↓
Earnings / Withdrawal
```

The nurse is a **healthcare service provider**, not a replacement for the doctor. The system must enforce the nurse's role and professional scope.

# 2. Main Objectives

The module should enable the nurse to:

1. Create and verify a professional account.
2. Build a professional profile.
3. Select approved nursing services.
4. Set service prices according to platform rules.
5. Configure working hours and availability.
6. Define service coverage areas.
7. Receive nursing-service requests.
8. Accept or reject requests within the allowed response window.
9. Manage appointments and home visits.
10. Record service/visit notes within the allowed nursing scope.
11. Complete visits and trigger payment settlement.
12. View ratings and reviews.
13. Handle complaints through the platform.
14. Track earnings, transactions, and withdrawals.
15. Receive relevant notifications.
16. Maintain privacy and security of patient data.

# 3. Nurse Registration Workflow

## 3.1 Entry Point

```text
Register
    ↓
Register as Nurse
```

The registration flow should clearly differ from Patient registration in required professional information, while both reuse the same shared authentication system.

## 3.2 Basic Account Information

```text
First Name
Last Name
Phone Number
Email Address
Password
Confirm Password
Profile Photo (optional at initial step)
```

### Validation Rules

- Email must have a valid format.
- Phone number must follow the accepted country format.
- Password must satisfy the platform password policy.
- Password confirmation must match.
- Email/phone uniqueness must be checked server-side.
- Client-side validation is only for UX; server-side validation is mandatory.

## 3.3 Personal Information

```text
Full Name
Date of Birth
Gender (when required by product policy)
National ID / Official Identity Reference
Address
Governorate
City
Area
```

Sensitive identity information must not be exposed publicly.

## 3.4 Professional Information

```text
Nursing Qualification
Nursing Specialty
Years of Experience
Professional / License Number
Graduation Institution
Current Workplace (if applicable)
Professional Bio
Professional Skills
```

Examples of approved services may include:

```text
Home Nursing
Elderly Care
Post-Operative Care
Wound Care
Patient Monitoring
Medication Assistance
Vital Signs Monitoring
Basic Home Care
```

The actual selectable service catalog should be controlled by the platform.

## 3.5 Professional Documents

```text
National ID
Graduation Certificate
Professional License
Professional Registration Documents
Additional Qualification Certificates
Other Required Documents
```

### Document Rules

- Documents are private.
- Documents must be stored securely.
- Public users must never receive unrestricted direct file URLs.
- Access is based on authorization.
- Sensitive downloads should be auditable.
- File type, size, and security checks must be enforced.
- Document status can be:

```text
PENDING
APPROVED
REJECTED
EXPIRED
REQUIRES_UPDATE
```

# 4. OTP / Account Verification

The nurse uses the shared TechCare Authentication module.

```text
Register
    ↓
OTP Sent
    ↓
Enter OTP
    ↓
OTP Valid?
   ├── No → Show Error / Retry Policy
   └── Yes
          ↓
      Account Verified
```

The system should enforce:

- OTP expiration.
- Maximum verification attempts.
- Resend limits.
- Rate limiting.
- Server-side validation.
- No trust in client-submitted verification state.

# 5. Nurse Account States

```text
REGISTERED
    ↓
CONTACT_VERIFIED
    ↓
PENDING_VERIFICATION
    ↓
APPROVED
```

Alternative paths:

```text
PENDING_VERIFICATION
    ↓
REJECTED
```

or:

```text
PENDING_VERIFICATION
    ↓
REQUIRES_UPDATE
    ↓
PENDING_VERIFICATION
```

A rejected/unverified nurse must not receive patient bookings.

# 6. Admin Verification

## 6.1 Why Verification Is Required

The nurse is a professional healthcare provider. The marketplace must not expose an unverified provider as fully bookable.

## 6.2 Admin Reviews

Admin may review:

```text
Identity Information
Professional Qualification
Professional License
Uploaded Documents
Profile Data
Selected Services
Service Area
```

## 6.3 Verification Result

### Approved

```text
Nurse Status = APPROVED
Provider can become searchable
Provider can receive eligible requests
```

### Rejected

```text
Nurse Status = REJECTED
Provider cannot receive new bookings
```

### Requires Update

```text
Nurse Status = REQUIRES_UPDATE
Admin provides reason
Nurse edits information/documents
Nurse resubmits
```

# 7. Nurse Login

The Nurse does not have a separate authentication implementation. The shared authentication system handles:

```text
Login
Logout
Forgot Password
Reset Password
OTP
Refresh Token
Change Password
JWT
Authorization
```

Typical flow:

```http
POST /api/auth/login
```

The authenticated user contains role/claims allowing the system to recognize:

```text
Role = Nurse
```

After successful login:

```text
Login
  ↓
JWT / Session
  ↓
Role Detection
  ↓
Nurse Dashboard
```

# 8. Nurse Dashboard

The dashboard is the nurse's main workspace.

## 8.1 Dashboard Summary Cards

```text
Today's Visits
Pending Requests
Upcoming Appointments
Completed Visits
Average Rating
Current Earnings
```

## 8.2 Dashboard Navigation

```text
Dashboard

Requests
├── New
├── Pending
├── Accepted
├── Rejected
├── Expired
├── Cancelled
└── Completed

Appointments
Calendar

My Services
Pricing
Distance Pricing
Availability
Working Hours
Service Area

Patients
Visit History

Reviews
Complaints

Financials
├── Earnings
├── Transactions
└── Withdrawals

Documents
Verification

Notifications

Profile
Settings
```

# 9. Nurse Profile

## 9.1 Public Profile

The public profile should show only approved/public information, such as:

```text
Name
Profile Photo
Professional Title
Specialty
Experience
Approved Services
Service Area
Starting Price
Rating
Number of Completed Services
Availability
Verification Badge
Professional Bio
```

## 9.2 Private Profile Data

Sensitive/private fields are not public:

```text
National ID
Private Documents
Sensitive Contact Information
Financial Details
Internal Verification Notes
Internal Admin Data
```

# 10. Services Management

The nurse selects services from an approved catalog.

```text
Service Catalog
├── Home Nursing
├── Wound Care
├── Elderly Care
├── Post-Operative Care
├── Patient Monitoring
├── Medication Assistance
└── Other Admin-Approved Nursing Services
```

## 10.1 Add Service

```text
Service Type
Price
Service Duration
Coverage Area
Optional Service Description
```

## 10.2 Service Status

```text
ACTIVE
INACTIVE
PENDING_APPROVAL
SUSPENDED
```

## 10.3 Rule

The nurse must not create arbitrary medical services outside the platform-approved model unless the Admin explicitly allows this.

# 11. Pricing Management

Pricing can be represented as:

```text
Base Service Price
+
Distance Fee
+
Approved Additional Charges
-
Discount
=
Final Price
```

The exact pricing policy is configurable by the product.

## 11.1 Example

```text
Base Price = 500 EGP
Included Distance = 5 KM
Extra Distance = 15 KM
Price per Extra KM = 5 EGP
Distance Fee = 15 × 5 = 75 EGP
Final Before Discount = 575 EGP
```

## 11.2 Price Calculation Rule

The client must not be trusted to define the final total. The server calculates:

```text
base_price
distance_fee
additional_fees
discount
final_amount
```

# 12. Availability Management

The nurse can configure:

```text
Working Days
Working Hours
Breaks
Available Time Slots
Blocked Dates
Vacation / Unavailable Periods
```

Example:

```text
Saturday
10:00 → 18:00

Sunday
12:00 → 20:00
```

# 13. Service Area

The nurse defines where the service can be delivered.

```text
Governorate
City
Areas
Maximum Distance
```

Example:

```text
Governorate: Dakahlia
City: Mansoura
Service Radius: 20 KM
```

# 14. Location and Distance

The system may calculate:

```text
Patient Location
+
Nurse Service Location
=
Estimated Distance
```

Recommended privacy model:

```text
Search Stage
→ Approximate Distance / Area

Booking Decision Stage
→ Minimum necessary location context

Confirmed Appointment
→ Exact service address to authorized nurse

Public Search
→ Never show exact patient address
```

Live patient location must not be publicly visible.

# 15. Nurse Search / Discovery

The patient can search nursing providers using:

```text
Service
Location
Distance
Price
Rating
Availability
Verified Status
Experience
```

Example:

```text
Ahmed Mohamed
Verified Nurse

Home Nursing

Rating: 4.8 ★
Distance: 4.2 KM
Price: 500 EGP
Available Today
```

Only eligible/verified nurses should appear as bookable providers.

# 16. Patient Booking Request

## 16.1 Patient Steps

```text
Search Nursing Service
      ↓
Select Nurse
      ↓
Select Service
      ↓
Select Date
      ↓
Select Time Slot
      ↓
Enter/Confirm Address
      ↓
Add Service Notes
      ↓
View Price
      ↓
Submit Request
```

## 16.2 Request Data

```text
Patient ID
Nurse ID
Service ID
Requested Date
Requested Time Slot
Service Address
Location Coordinates (when applicable)
Service Notes
Price Snapshot
Created At
Expires At
Status
```

# 17. Booking Request Lifecycle

Primary request states:

```text
PENDING
ACCEPTED
REJECTED
EXPIRED
CANCELLED
```

Once accepted and confirmed:

```text
CONFIRMED
IN_PROGRESS
COMPLETED
```

# 18. Five-Minute Response Rule

The platform can enforce a configurable response window. For the current design, the example is five minutes.

```text
Request Created
    ↓
created_at = now
expires_at = created_at + configured response window
```

Example:

```text
created_at = 10:00
expires_at = 10:05
```

At/after expiration:

```text
PENDING → EXPIRED
```

The nurse cannot accept the request after expiration.

**Important:** this must be enforced by the Backend, not only by a countdown in the UI. The UI timer is informational.

# 19. Nurse Receives New Request

The nurse can receive:

```text
In-app Notification
Push Notification (when supported)
Email/SMS depending on product setup
```

The request screen can show:

```text
Service
Requested Date
Requested Time
Approximate Distance
Estimated Price
Necessary Patient Information
Service Notes
```

Do not expose unnecessary sensitive patient data before acceptance.

# 20. Accept Request

Nurse selects:

```text
Accept Request
```

Backend validates:

```text
Request is still PENDING
Request has not expired
Nurse is active
Nurse is verified
Nurse is eligible for service
Requested slot is still available
Pricing is valid
```

Then:

```text
PENDING
   ↓
ACCEPTED
   ↓
CONFIRMED
```

The patient is notified.

# 21. Reject Request

The nurse selects Reject. A reason may be required:

```text
Not Available
Schedule Conflict
Outside Service Area
Unable to Provide Requested Service
Other
```

State:

```text
PENDING → REJECTED
```

The patient is notified.

# 22. Expired Request

When the response window ends:

```text
PENDING → EXPIRED
```

The system should:

- Prevent acceptance.
- Notify the patient if appropriate.
- Release temporary resources.
- Allow the patient to request another provider where applicable.

# 23. Double Booking Prevention

Critical backend rule:

```text
One Nurse
+
One Time Slot
+
One Confirmed Appointment
=
No conflicting second booking
```

Concurrent requests must be handled safely using appropriate transaction/concurrency controls so that two patients cannot successfully confirm the same nurse/time slot.

# 24. Appointment Creation

After acceptance, the system creates/finalizes:

```text
Appointment ID
Nurse
Patient
Service
Date
Start Time
End Time
Address
Price
Status = CONFIRMED
```

# 25. Appointment Management

The nurse can view:

```text
Upcoming
Today
Completed
Cancelled
No-Show
```

Calendar views:

```text
Day
Week
Month
```

# 26. Before Visit

Before the scheduled visit, the nurse can see the information necessary to perform the service:

```text
Patient
Service
Date
Time
Confirmed Address
Relevant Authorized Information
Service Notes
```

The nurse should not automatically receive the patient's entire medical chart.

# 27. Appointment Reminder

The system can send reminders according to configured product rules:

```text
Upcoming Visit
Visit Starting Soon
Schedule Change
Cancellation
```

# 28. Start Visit

When the nurse begins the service:

```text
CONFIRMED
    ↓
IN_PROGRESS
```

The system records:

```text
Appointment ID
Nurse
Start Time
```

# 29. Nursing Visit Record

The visit should have its own service record:

```text
Nursing Visit
├── Appointment
├── Nurse
├── Patient
├── Service
├── Started At
├── Completed At
├── Visit Notes
├── Services Performed
├── Observations
└── Follow-up Notes
```

# 30. Nursing Service Execution

Depending on the approved service, the nurse may record appropriate information such as:

```text
Vital Signs Monitoring
Wound Care
Post-Operative Support
Elderly Care
Medication Assistance
Patient Monitoring
Basic Home Nursing
```

The exact allowed fields depend on the specific service and the nurse's professional scope.

# 31. Clinical Scope Boundary

The Nurse Module must not automatically grant doctor privileges.

The nurse must not:

```text
Create an independent physician diagnosis
Issue physician prescriptions
Change a doctor's medication plan without an authorized workflow
Access unrelated patients
Access unrelated medical records
```

The system should explicitly define role permissions.

# 32. Visit Notes

Possible note fields:

```text
Services Performed
Observed Information
Vital Signs (when relevant)
General Nursing Notes
Patient Response
Follow-up Recommendation
```

Medical-related records should support auditability.

# 33. Complete Visit

When the service is finished:

```text
IN_PROGRESS
     ↓
COMPLETED
```

The system stores:

```text
Start Time
Completion Time
Services Performed
Nursing Notes
Final Price
Payment Status
```

The appointment becomes eligible for post-service processes.

# 34. Cancellation Workflow

Cancellation can happen by:

```text
Patient
Nurse
Admin
System
```

Record:

```text
Cancelled By
Cancellation Reason
Cancelled At
Refund Status
```

Business rules for refunds/penalties should be centrally controlled.

# 35. No-Show Workflow

Possible states:

```text
NO_SHOW_PATIENT
NO_SHOW_NURSE
```

The system should record:

```text
Reported By
Reason
Created At
Evidence/Notes when applicable
Admin Decision when required
```

No-show rules can affect:

```text
Refund
Provider earnings
Patient reliability metrics
Complaint handling
```

# 36. Payment Workflow

Recommended lifecycle:

```text
Booking
   ↓
Payment Initiation
   ↓
Payment Verification
   ↓
Confirmed Appointment
   ↓
Service Completed
   ↓
Settlement
```

Payment information must be verified server-side. The system must not mark a payment successful merely because the frontend says so.

# 37. Price Breakdown

Patient and provider should understand:

```text
Base Service Price
+ Distance Fee
+ Approved Additional Fees
- Discount
= Total Amount
```

Example:

```text
Base Service       500 EGP
Distance Fee        75 EGP
Discount            50 EGP
--------------------------
Total               525 EGP
```

# 38. Financial Transactions

A transaction record may include:

```text
Transaction ID
Appointment ID
Patient ID
Nurse ID
Gross Amount
Platform Fee
Discount
Refund
Net Provider Amount
Payment Status
Created At
```

Financial history should be traceable and historical financial records should not be directly editable like ordinary profile data.

# 39. Nurse Earnings

The nurse dashboard can show:

```text
Current/Period Earnings
Completed Service Earnings
Pending Amounts
Platform Fees
Refunds
Net Earnings
```

# 40. Withdrawals

```text
Financials
   ↓
Withdrawals
   ↓
Request Withdrawal
```

Withdrawal states:

```text
PENDING
APPROVED
PROCESSING
COMPLETED
REJECTED
```

# 41. Ratings and Reviews

A patient can rate the nurse only after an eligible completed service.

```text
Rating = 1 to 5
Review = Optional Text
```

Eligible condition:

```text
Appointment Status = COMPLETED
```

Not eligible:

```text
PENDING
REJECTED
EXPIRED
CANCELLED
```

# 42. Complaints

A patient can create a complaint linked to the service/appointment:

```text
Appointment
    ↓
Create Complaint
    ↓
Admin Review
```

Possible fields:

```text
Complaint Type
Description
Attachments
Created At
Status
```

Statuses:

```text
OPEN
UNDER_REVIEW
RESOLVED
CLOSED
```

# 43. Nurse Complaint Handling

The nurse may:

```text
View Complaint
Read Patient Statement
Submit Response
Upload Allowed Evidence
View Resolution Status
```

The nurse must not be able to:

```text
Delete Complaint
Change Admin Decision
Modify Original Complaint
Delete Audit History
```

# 44. Notifications

Important nurse notifications:

```text
New Booking Request
Request Accepted
Request Rejected
Request Expired
Appointment Confirmed
Appointment Reminder
Patient Cancellation
Appointment Cancellation
No-Show Update
Payment Update
Earnings Update
Withdrawal Update
New Review
Complaint Update
Document Verification Update
Admin Announcement
```

# 45. Security and Authorization

The Nurse Module contains sensitive information and must follow least privilege.

## Nurse can access

```text
Own Profile
Own Services
Own Availability
Own Requests
Own Appointments
Authorized Patient Information
Own Visit Records
Own Reviews
Own Complaints
Own Financial Information
Own Notifications
```

## Nurse cannot access

```text
Another Nurse's private data
Another Provider's financial records
Unrelated Patient Medical Records
Admin Verification Controls
Admin Audit Logs unless explicitly exposed
Doctor-only functions
Pharmacy internal records
Laboratory internal records
Other users' passwords/tokens
```

# 46. Object-Level Authorization

Every sensitive endpoint must verify ownership/authorization.

Bad pattern:

```http
GET /api/appointments/123
```

without checking whether the current nurse is allowed to access appointment 123.

Correct behavior:

```text
Authenticated Nurse
        ↓
Is Nurse owner/authorized participant?
        ↓
Yes → Return allowed data
No  → Forbidden
```

This must be enforced server-side.

# 47. Sensitive Location Privacy

Recommended access stages:

```text
Search Stage
→ Approximate Distance / Area

Booking Decision Stage
→ Minimum necessary location context

Confirmed Appointment
→ Exact service address to authorized nurse

Public Search
→ Never show exact patient address
```

Live patient location must not be publicly exposed.

# 48. File Security

Professional documents should use secure storage.

Controls should include:

```text
Private Storage
Authorization Checks
File Type Validation
Size Limits
Secure Download
Audit Logging
No Public Directory Listing
```

The same concept applies to private patient attachments.

# 49. Auditability

Important events should be auditable:

```text
Account Created
Verification Decision
Document Uploaded
Document Approved/Rejected
Service Added
Price Changed
Availability Changed
Booking Accepted
Booking Rejected
Booking Cancelled
Visit Started
Visit Completed
Complaint Created
Financial Transaction Created
Withdrawal Requested
Withdrawal Processed
```

# 50. Recommended Backend Domain Entities

The exact database design may evolve. Potential entities include:

```text
User
Role
NurseProfile
NurseQualification
NurseDocument
NurseService
ServiceCatalog
NurseAvailability
NurseWorkingHour
ServiceArea
BookingRequest
Appointment
NursingVisit
VisitNote
Patient
PatientAddress
Payment
Transaction
ProviderEarning
Withdrawal
Review
Complaint
Notification
AuditLog
```

# 51. Suggested API Groups

```text
/api/auth/*
/api/nurses/*
/api/nurse/services/*
/api/nurse/availability/*
/api/nurse/requests/*
/api/nurse/appointments/*
/api/nurse/visits/*
/api/nurse/reviews/*
/api/nurse/complaints/*
/api/nurse/financials/*
/api/nurse/withdrawals/*
/api/notifications/*
```

# 52. Example Nurse APIs

## Profile

```http
GET    /api/nurses/me
PUT    /api/nurses/me
```

## Services

```http
GET    /api/nurses/me/services
POST   /api/nurses/me/services
PUT    /api/nurses/me/services/{id}
DELETE /api/nurses/me/services/{id}
```

## Availability

```http
GET    /api/nurses/me/availability
PUT    /api/nurses/me/availability
```

## Requests

```http
GET  /api/nurses/me/requests
POST /api/nurses/me/requests/{id}/accept
POST /api/nurses/me/requests/{id}/reject
```

## Appointments

```http
GET  /api/nurses/me/appointments
GET  /api/nurses/me/appointments/{id}
POST /api/nurses/me/appointments/{id}/cancel
```

## Visits

```http
POST /api/nurse-visits/{id}/start
PUT  /api/nurse-visits/{id}
POST /api/nurse-visits/{id}/complete
```

## Financials

```http
GET  /api/nurses/me/earnings
GET  /api/nurses/me/transactions
GET  /api/nurses/me/withdrawals
POST /api/nurses/me/withdrawals
```

# 53. Frontend Pages

Recommended Nurse frontend pages:

```text
/Nurse/Register
/Nurse/Verification
/Nurse/Dashboard
/Nurse/Profile
/Nurse/Documents
/Nurse/Services
/Nurse/Pricing
/Nurse/DistancePricing
/Nurse/Availability
/Nurse/Requests
/Nurse/Requests/{id}
/Nurse/Appointments
/Nurse/Calendar
/Nurse/Appointments/{id}
/Nurse/Visits/{id}
/Nurse/Patients
/Nurse/Visits/History
/Nurse/Reviews
/Nurse/Complaints
/Nurse/Financials
/Nurse/Transactions
/Nurse/Withdrawals
/Nurse/Notifications
/Nurse/Settings
```

The exact URL convention can be standardized across the project.

# 54. Validation Rules

## Profile

```text
Required personal fields
Valid phone/email
Valid dates
Valid professional data
```

## Services

```text
Service must be allowed
Price > 0
Duration > 0
```

## Availability

```text
Start Time < End Time
No invalid overlapping slots
```

## Booking

```text
Nurse must be verified
Service must be active
Slot must be available
Request must not be expired
Nurse must be eligible for the service
```

## Visit

```text
Only authorized nurse can start/update the visit
Appointment must be valid
Cannot complete before it is started unless an explicit approved workflow allows it
```

# 55. Error Handling

The API should return consistent errors.

Example:

```json
{
  "success": false,
  "message": "This booking request has expired.",
  "code": "BOOKING_REQUEST_EXPIRED"
}
```

Suggested codes:

```text
NURSE_NOT_VERIFIED
SERVICE_NOT_AVAILABLE
BOOKING_EXPIRED
SLOT_ALREADY_BOOKED
UNAUTHORIZED
FORBIDDEN
APPOINTMENT_NOT_FOUND
INVALID_STATUS_TRANSITION
```

# 56. State Machine

## Booking Request

```text
PENDING
 ├── ACCEPTED
 │     └── CONFIRMED
 │            ├── IN_PROGRESS
 │            │      └── COMPLETED
 │            └── CANCELLED
 │
 ├── REJECTED
 │
 ├── EXPIRED
 │
 └── CANCELLED
```

Invalid transitions must be rejected server-side.

Example:

```text
EXPIRED → ACCEPTED
```

must fail.

## Appointment

```text
CONFIRMED
    ├── IN_PROGRESS
    │      └── COMPLETED
    │
    ├── CANCELLED
    │
    ├── NO_SHOW_PATIENT
    │
    └── NO_SHOW_NURSE
```

# 57. End-to-End Example

```text
1. Nurse registers.
2. Nurse verifies account with OTP.
3. Admin reviews professional documents.
4. Admin approves nurse.
5. Nurse completes public profile.
6. Nurse selects Home Nursing.
7. Nurse sets base price = 500 EGP.
8. Nurse defines service area.
9. Nurse defines working hours.
10. Patient searches for Home Nursing.
11. Patient finds verified nurse.
12. Patient chooses date/time.
13. Patient creates booking request.
14. System stores request as PENDING.
15. Backend sets created_at and expires_at.
16. Nurse receives notification.
17. Nurse opens request.
18. Nurse accepts before expiry.
19. Backend checks slot and authorization.
20. Appointment becomes CONFIRMED.
21. Patient is notified.
22. Nurse sees exact confirmed service address.
23. Reminder is sent.
24. Nurse arrives.
25. Nurse starts visit.
26. Status becomes IN_PROGRESS.
27. Nurse performs approved nursing service.
28. Nurse records appropriate visit notes.
29. Nurse completes visit.
30. Status becomes COMPLETED.
31. Payment/settlement is processed.
32. Transaction is recorded.
33. Nurse earnings are updated.
34. Patient becomes eligible to rate the nurse.
35. Patient submits rating/review.
36. If needed, patient creates a complaint.
37. Admin handles the complaint.
38. Nurse sees updated earnings.
39. Nurse requests withdrawal.
40. Withdrawal moves through the financial workflow.
```

# 58. Definition of Done — Nurse Feature

A Nurse feature is not Done merely because the page exists.

```text
UI implemented
Backend implemented
Database changes implemented
API implemented
Validation implemented
Authorization implemented
Error handling implemented
Loading/empty/error states handled
Tests added
Integration tested
Code reviewed
Documentation updated
```

# 59. Suggested Test Scenarios

## Registration

```text
Valid registration
Duplicate email
Duplicate phone
Invalid password
Invalid OTP
Expired OTP
Too many OTP attempts
Missing required document
```

## Verification

```text
Admin approves
Admin rejects
Admin requests update
Unverified nurse tries to receive booking
```

## Booking

```text
Valid request
Expired request
Accept before expiry
Accept after expiry
Reject request
Two patients attempt same slot
Nurse outside service area
Inactive service
Cancelled request
```

## Visit

```text
Start confirmed appointment
Start unauthorized appointment
Complete active visit
Complete before start
Edit another nurse's visit
```

## Security

```text
Access another nurse's appointment
Access unrelated patient record
Change another nurse's service
Read another provider's earnings
Download unauthorized document
Forge price in request
Forge appointment status
```

# 60. Scrum / Team Ownership

For the proposed five-person Full-Stack .NET team:

```text
Member 1
Authentication + Patient

Member 2
Doctor

Member 3
Nurse

Member 4
Pharmacy

Member 5
Laboratory + Blood Donor
```

The Nurse owner is responsible for the Nurse feature end-to-end:

```text
Database
    ↓
ASP.NET Core
    ↓
API
    ↓
Frontend
    ↓
Validation
    ↓
Authorization
    ↓
Testing
    ↓
Integration
```

Ownership means accountability, not isolation. Another team member should review the pull request before merge.

# 61. Recommended Git Structure

```text
main
develop
feature/nurse-profile
feature/nurse-services
feature/nurse-booking
feature/nurse-availability
feature/nurse-appointments
feature/nurse-visits
feature/nurse-financials
feature/nurse-reviews
```

Pull Request flow:

```text
Feature Branch
      ↓
Push
      ↓
Pull Request
      ↓
Code Review
      ↓
Testing
      ↓
Merge
```

# 62. Nurse Workflow Summary

```text
                   NURSE
                     │
             ┌───────┴────────┐
             ↓                ↓
        Registration       Login/Auth
             │
             ↓
        Verification
             │
             ↓
          Approval
             │
             ↓
       Profile & Documents
             │
             ↓
         My Services
             │
             ↓
     Pricing + Distance Pricing
             │
             ↓
     Availability + Working Hours
             │
             ↓
      Search Visibility
             │
             ↓
      Booking Requests
        ┌────┼────┐
        ↓    ↓    ↓
     Accept Reject Expire
        │
        ↓
    Confirmed
        │
        ↓
    Appointment
        │
        ↓
    Start Visit
        │
        ↓
    Nursing Service
        │
        ↓
    Visit Notes
        │
        ↓
   Complete Visit
        │
   ┌────┼────────┐
   ↓    ↓        ↓
Payment Rating Complaint
   │
   ↓
Earnings
   │
   ↓
Withdrawal
```

# 63. Core Business Rules Checklist

```text
[ ] Nurse must be verified before becoming fully bookable.
[ ] Nurse can only provide approved services.
[ ] Exact patient location is protected before confirmation.
[ ] Booking expiry is enforced on the server.
[ ] Expired requests cannot be accepted.
[ ] Double booking is prevented using backend concurrency controls.
[ ] Price is calculated by the server.
[ ] Nurse can only access authorized patient information.
[ ] Nurse cannot access doctor-only functionality.
[ ] Visit status changes follow the state machine.
[ ] Rating is allowed only after completed service.
[ ] Complaints are linked to the relevant transaction/service.
[ ] Financial records are auditable.
[ ] Sensitive documents use protected storage.
[ ] Important security and financial events are logged.
[ ] Client-side checks never replace server-side authorization.
```

# 64. Product Boundary

The Nurse Module is responsible for **professional nursing service delivery and coordination**.

It is not responsible for autonomous medical diagnosis or physician prescribing.

Future integrations may introduce additional clinical workflows, but such capabilities should be explicitly authorized, scoped, audited, and separated by role.

# 65. Final Nurse Journey

```text
Nurse
 ↓
Register
 ↓
Verify Account
 ↓
Submit Professional Documents
 ↓
Wait for Admin Verification
 ↓
Approved
 ↓
Complete Profile
 ↓
Add Approved Nursing Services
 ↓
Set Pricing
 ↓
Set Service Area
 ↓
Set Working Hours
 ↓
Set Availability
 ↓
Appear in Search
 ↓
Receive Patient Request
 ↓
Review Request
 ↓
Accept / Reject
 ↓
Appointment Confirmed
 ↓
Receive Reminder
 ↓
Access Authorized Visit Information
 ↓
Start Visit
 ↓
Perform Nursing Service
 ↓
Record Visit Notes
 ↓
Complete Visit
 ↓
Payment Settlement
 ↓
Receive Rating
 ↓
Handle Complaint if any
 ↓
Track Earnings
 ↓
Request Withdrawal
```

---

## Document Status

This document is the detailed reference workflow for the TechCare Nurse module and can be used as the basis for:

- User Stories
- Use Cases
- Database Design
- ASP.NET Core APIs
- React/Frontend Screens
- Authorization Policies
- QA Test Cases
- Sprint Tasks
- Integration Testing

