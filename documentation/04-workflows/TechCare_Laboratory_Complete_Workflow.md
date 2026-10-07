# TechCare — Complete Laboratory Workflow
## End-to-End Product, Functional, Backend, Frontend, Database, Security, Operations, and Testing Specification

> **Project:** TechCare  
> **Module:** Laboratory / Diagnostic Services  
> **Audience:** Product Owner, Scrum Team, Full-Stack .NET Developers, QA, UI/UX, System Design  
> **Status:** Web Prototype / MVP Specification  
> **Core Principle:** Bring Healthcare Closer to the Patient

---

# 1. Module Overview

The **Laboratory Module** connects patients with verified laboratories and enables laboratory diagnostic services from discovery to result delivery.

The module supports the complete journey:

```text
Laboratory Registration
        ↓
Laboratory / Responsible Professional Verification
        ↓
Admin Approval
        ↓
Laboratory Profile Setup
        ↓
Test Catalog
        ↓
Prices & Availability
        ↓
Home Collection Setup
        ↓
Patient Searches Test
        ↓
Nearby Laboratory Discovery
        ↓
Compare Price / Distance / Rating / Availability
        ↓
Book Test
        ↓
Laboratory Confirms Booking
        ↓
Collector / Phlebotomy Staff Assignment
        ↓
Patient Visit / Home Sample Collection
        ↓
Sample Identification & Collection
        ↓
Sample Transport / Receipt
        ↓
Laboratory Processing
        ↓
Result Validation / Authorization
        ↓
Result Published
        ↓
Patient Notification
        ↓
Result Added to Patient Medical Timeline
        ↓
Doctor Can View Relevant Result Under Authorization
        ↓
Payment / Settlement
        ↓
Rating / Complaint
```

The laboratory module is more than a booking page. It models a **diagnostic service lifecycle** where the system must keep the patient, booking, test, sample, result, and responsible laboratory linked from start to finish.

---

# 2. Main Goals

The Laboratory Module should allow TechCare to:

1. Register laboratories as healthcare organizations.
2. Register or link authorized laboratory professionals and staff.
3. Verify laboratory and professional information.
4. Create and manage a laboratory profile.
5. Define available diagnostic tests.
6. Define pricing and service options.
7. Support laboratory location and working hours.
8. Offer home sample collection where enabled.
9. Let patients search for nearby tests and laboratories.
10. Support doctor-recommended laboratory tests.
11. Support patient booking and scheduling.
12. Assign collection staff where required.
13. Track sample collection and laboratory processing states.
14. Publish validated results to the correct patient account.
15. Notify the patient when results are ready.
16. Link relevant laboratory results to the patient medical timeline.
17. Provide financial, rating, complaint, and audit workflows.
18. Protect patient health information and laboratory documents.

---

# 3. Main Actors

## 3.1 Laboratory Owner / Manager

Responsible for:

```text
Laboratory Profile
Staff
Services
Tests
Prices
Availability
Orders / Bookings
Financials
Complaints
```

## 3.2 Laboratory Professional

Depending on the operating model, authorized professionals may review or validate results.

## 3.3 Sample Collector / Phlebotomy Staff

Responsible for collection-related operations when home collection is enabled.

## 3.4 Patient

Can:

```text
Search Test
Choose Laboratory
Book
Track Collection
View Result
Download Report
Rate Laboratory
Create Complaint
```

## 3.5 Doctor

Can create or recommend laboratory requests through the Doctor workflow and later access relevant results when authorized.

## 3.6 Admin

Controls:

```text
Verification
Catalog governance
Laboratory suspension
Complaints
Audit
Financial disputes
```

---

# 4. Important Architecture Principle

Separate these concepts:

```text
Laboratory Organization
        ≠
Laboratory Staff
        ≠
Diagnostic Test
        ≠
Patient Booking
        ≠
Sample
        ≠
Result
        ≠
Payment
```

The most important relationship is:

```text
Patient
   ↓
Lab Order / Booking
   ↓
Test
   ↓
Sample
   ↓
Result
   ↓
Patient Medical Timeline
```

This prevents the system from treating a test as if it were only an appointment.

---

# 5. Laboratory Registration

## 5.1 Entry Point

```text
Register
    ↓
Select Organization / Laboratory
    ↓
Laboratory Registration
```

The registration should be divided into clear steps.

---

# 6. Laboratory Account Information

Use the shared TechCare Authentication module for:

```text
Registration
Login
Logout
Forgot Password
Reset Password
OTP
Refresh Token
Change Password
Security
```

Possible fields:

| Field | Suggested Name | Type | Level |
|---|---|---|---|
| Email | `email` | Email | Required |
| Phone | `phone_number` | Phone | Required |
| Password | `password` | Password | Required |
| Confirm Password | `confirm_password` | Password | Required |
| Contact Verification | `verification_status` | Enum | System |

---

# 7. Laboratory Organization Information

The laboratory profile may include:

| Field | Suggested Name | Level | Notes |
|---|---|---:|---|
| Laboratory Name | `name` | Required | Public name |
| Legal / Business Name | `legal_name` | Conditional | Depends on operating model |
| Phone | `phone_number` | Required | Patient contact |
| Email | `email` | Required | Contact |
| Governorate | `governorate` | Required | Structured location |
| City | `city` | Required | Structured location |
| District / Area | `district` | Required | Search/location |
| Street | `street` | Required | Address |
| Building | `building_number` | Recommended | Physical location |
| Latitude | `latitude` | System / Conditional | Geolocation |
| Longitude | `longitude` | System / Conditional | Geolocation |
| Logo | `logo` | Recommended | Public branding |
| Cover Image | `cover_image` | Optional | Public profile |
| About | `description` | Required | Patient-facing description |
| Working Hours | `working_hours` | Required | Operating schedule |
| Home Collection | `home_collection_enabled` | Required | Service capability |
| Pickup / Walk-in | `walk_in_enabled` | Optional | Physical branch service |

---

# 8. Responsible Professional / Staff

A laboratory may contain multiple authorized staff members.

Recommended model:

```text
Laboratory
    │
    ├── Owner / Manager
    ├── Laboratory Professional
    ├── Collector / Phlebotomy Staff
    └── Operational Staff
```

Relationship:

```text
LaboratoryStaff
├── LaboratoryId
├── UserId
├── Role
├── Status
├── JoinedAt
└── PermissionGroup
```

---

# 9. Staff Roles

Example roles:

```text
LAB_OWNER
LAB_MANAGER
LAB_PROFESSIONAL
SAMPLE_COLLECTOR
LAB_STAFF
```

Do not give all laboratory users full access.

---

# 10. Professional Information

Where professional verification is applicable:

```text
Full Name
Professional Qualification
Professional Registration / License Number
Specialty / Discipline
Years of Experience
Professional Documents
```

Documents should be private.

---

# 11. Laboratory Verification

Before appearing as an approved diagnostic provider:

```text
Registration
    ↓
Profile Completion
    ↓
Documents Uploaded
    ↓
Verification Submitted
    ↓
Admin Review
    ↓
Approved / Rejected / Requires Update
```

---

# 12. Verification States

```text
DRAFT
SUBMITTED
UNDER_REVIEW
APPROVED
REQUIRES_UPDATE
REJECTED
SUSPENDED
DEACTIVATED
```

Only eligible approved laboratories should be bookable.

---

# 13. Verification Documents

Depending on the operating model, document categories may include:

```text
Organization / Business Documents
Responsible Professional Documents
Identity Documents
Professional License / Registration
Relevant Facility Credentials
Additional Required Certificates
```

Metadata:

```text
DocumentId
DocumentType
File
UploadedAt
UploadedBy
Status
ReviewedBy
ReviewedAt
RejectionReason
ExpiryDate
```

Possible states:

```text
UPLOADED
UNDER_REVIEW
APPROVED
REJECTED
EXPIRED
REPLACEMENT_REQUIRED
```

---

# 14. Laboratory Public Profile

Patient can see:

```text
Laboratory Name
Verified Badge
Rating
Reviews
Location / Area
Approximate Distance
Working Hours
Home Collection Availability
Available Tests
Test Prices
Service Information
```

Do not expose:

```text
Private Documents
Internal Staff Data
Financial Data
Admin Notes
Patient Data of Others
Internal System Information
```

---

# 15. Laboratory Dashboard

The dashboard should answer:

> What requires action now?

Example metrics:

```text
New Bookings
Pending Confirmations
Today's Collections
Samples Awaiting Receipt
Samples Processing
Results Awaiting Validation
Results Published
Today's Revenue
Pending Settlement
Average Rating
Open Complaints
```

---

# 16. Laboratory Dashboard Navigation

```text
Dashboard

Bookings
├── New
├── Pending
├── Confirmed
├── Upcoming
├── In Progress
├── Completed
├── Cancelled
├── No-Show
└── History

Tests
├── Test Catalog
├── Prices
├── Availability
└── Packages

Samples
├── Pending Collection
├── Collected
├── In Transit
├── Received
├── Rejected
└── History

Results
├── Draft
├── Awaiting Validation
├── Published
└── History

Staff
Collection Team
Working Hours
Home Collection

Financials
├── Transactions
├── Earnings
├── Refunds
└── Withdrawals

Reviews
Complaints
Notifications
Documents
Verification
Profile
Settings
```

---

# 17. Laboratory Test Catalog

Tests should be managed from a controlled catalog.

Example:

```text
CBC
Glucose
HbA1c
Lipid Profile
Liver Function Tests
Kidney Function Tests
Urine Analysis
Hormonal Tests
Vitamin Tests
Other Approved Tests
```

The final catalog depends on the laboratory model.

---

# 18. Diagnostic Test Master Data

A centralized test definition can contain:

| Field | Suggested Name | Level |
|---|---|---|
| Test Name | `name` | Required |
| Test Code | `code` | Recommended |
| Category | `category_id` | Required |
| Description | `description` | Recommended |
| Sample Type | `sample_type` | Required |
| Preparation Requirements | `preparation_instructions` | Conditional |
| Fasting Required | `fasting_required` | Conditional |
| Default Processing Information | `processing_info` | Optional |
| Active | `active` | Required |

The platform should avoid encoding patient-specific clinical interpretation into the test master unless explicitly designed and clinically governed.

---

# 19. Laboratory-Specific Test Offering

A laboratory may offer one master test with its own:

```text
Price
Availability
Home Collection Availability
Collection Fee
Turnaround Information
Special Instructions
Active Status
```

Entity:

```text
LaboratoryTestOffering
├── LaboratoryId
├── TestId
├── Price
├── HomeCollectionAvailable
├── CollectionFee
├── Active
└── ServiceNotes
```

---

# 20. Test Availability

A test can be:

```text
AVAILABLE
UNAVAILABLE
TEMPORARILY_UNAVAILABLE
SUSPENDED
```

A patient must not be able to book a test that the laboratory has deactivated.

---

# 21. Home Sample Collection

A laboratory can enable:

```text
Home Sample Collection = ON
```

Then the patient can choose:

```text
Laboratory Test
       ↓
Home Collection
       ↓
Address
       ↓
Date / Time Slot
```

---

# 22. Home Collection Configuration

Possible fields:

| Field | Suggested Name | Level |
|---|---|---|
| Home Collection Enabled | `home_collection_enabled` | Required |
| Collection Fee | `collection_fee` | Conditional |
| Service Radius | `collection_radius_km` | Recommended |
| Working Areas | `collection_areas` | Optional |
| Collection Slot Duration | `slot_duration` | Required |
| Collector Capacity | `collector_capacity` | Recommended |
| Active | `active` | Required |

---

# 23. Working Hours

Laboratory can define:

```text
Branch Working Hours
Collection Working Hours
Home Collection Hours
Holiday / Closed Dates
```

Example:

```text
Branch:
08:00 → 22:00

Home Collection:
09:00 → 18:00
```

---

# 24. Collection Time Slots

For home collection:

```text
09:00 → 10:00
10:00 → 11:00
11:00 → 12:00
```

A slot may be:

```text
AVAILABLE
HELD
BOOKED
ASSIGNED
COMPLETED
BLOCKED
```

The backend must prevent conflicting bookings.

---

# 25. Patient Laboratory Discovery

Patient enters:

```text
Laboratory
    ↓
Search Test
```

Possible search inputs:

```text
Test Name
Category
Location
Date
Home Collection
Price
Rating
Distance
```

---

# 26. Laboratory Search Fields

| Field | Suggested Name | Level |
|---|---|---|
| Test | `test_id` | Required |
| Query | `query` | Optional |
| Location | `location_context` | Required for nearby search |
| Radius | `radius_km` | Optional |
| Home Collection | `home_collection` | Optional |
| Maximum Price | `max_price` | Optional |
| Minimum Rating | `min_rating` | Optional |
| Availability | `availability_filter` | Optional |
| Sort | `sort` | Optional |

---

# 27. Laboratory Result Card

Example:

```text
ABC Laboratory
✓ Verified
⭐ 4.8
2.5 KM

CBC
150 EGP

Home Collection ✓
Available Today

[ View Details ]
```

---

# 28. Test Details Page

Patient sees:

```text
Test Name
Description
Preparation Requirements
Sample Type
Price
Home Collection
Collection Fee
Laboratory Information
Available Slots
```

If fasting/preparation is required, it should be clearly displayed before booking.

---

# 29. Doctor → Laboratory Workflow

A doctor can create a laboratory request:

```text
Doctor Consultation
      ↓
Recommended / Requested Test
      ↓
Patient Medical Record / Lab Request
      ↓
Patient Chooses Eligible Laboratory
      ↓
Booking
```

The doctor request should include only the information needed for the laboratory workflow.

---

# 30. Lab Referral / Test Request

Possible fields:

```text
ReferralId
DoctorId
PatientId
TestId
Clinical Note / Reason (when necessary)
CreatedAt
Status
```

Status may include:

```text
CREATED
SENT_TO_PATIENT
BOOKED
COMPLETED
CANCELLED
```

---

# 31. Patient Booking Flow

The patient chooses:

```text
Test
   ↓
Laboratory
   ↓
Home Collection / Lab Visit
   ↓
Address (if home collection)
   ↓
Date
   ↓
Time Slot
   ↓
Review Price
   ↓
Confirm Booking
```

---

# 32. Laboratory Booking Fields

| Field | Suggested Name | Level |
|---|---|---|
| Patient | `patient_id` | System |
| Laboratory | `laboratory_id` | Required |
| Test | `test_id` | Required |
| Referral | `referral_id` | Conditional |
| Fulfillment Mode | `fulfillment_mode` | Required |
| Address | `address_id` | Required for home collection |
| Date | `collection_date` / `appointment_date` | Required |
| Slot | `slot_id` | Required |
| Patient Notes | `patient_note` | Optional |
| Price Snapshot | `price_snapshot` | System |
| Created At | `created_at` | System |
| Status | `status` | System |

---

# 33. Fulfillment Modes

Possible:

```text
HOME_COLLECTION
LAB_VISIT
```

A laboratory may enable one or both.

---

# 34. Price Calculation

For home collection:

```text
Test Price
+
Collection Fee
+
Approved Delivery / Travel Fee if applicable
-
Discount
=
Final Total
```

Example:

```text
CBC = 150 EGP
Home Collection = 50 EGP
Discount = 0
-----------------
Total = 200 EGP
```

The server calculates the final amount.

---

# 35. Price Snapshot

Store at order/booking creation:

```text
test_price_snapshot
collection_fee_snapshot
additional_fee_snapshot
discount_snapshot
final_total_snapshot
pricing_policy_version
```

Historical bookings must not change because the laboratory later changes its prices.

---

# 36. Booking Lifecycle

Recommended booking states:

```text
PENDING
CONFIRMED
COLLECTION_ASSIGNED
IN_PROGRESS
COMPLETED
CANCELLED
NO_SHOW_PATIENT
NO_SHOW_LAB
```

Alternative early outcomes:

```text
REJECTED
EXPIRED
```

---

# 37. Laboratory Accepts Booking

When a booking arrives:

```text
New Booking
      ↓
Lab Reviews
      ↓
Test Available?
      ↓
Slot Available?
      ↓
Home Collection Supported?
      ↓
Accept
```

Then:

```text
PENDING → CONFIRMED
```

Patient receives a confirmation notification.

---

# 38. Booking Rejection

Possible reasons:

```text
Test Unavailable
No Collection Capacity
Outside Service Area
Operational Issue
Selected Slot Unavailable
Other
```

State:

```text
PENDING → REJECTED
```

If payment was already captured, the relevant refund/reversal workflow is triggered.

---

# 39. Home Collection Assignment

After a home-collection booking is confirmed:

```text
CONFIRMED
    ↓
Find Available Collector
    ↓
Assign Collector
    ↓
COLLECTION_ASSIGNED
```

Assignment may consider:

```text
Availability
Coverage Area
Current Workload
Service Capability
Travel Distance
```

---

# 40. Collector Assignment Fields

```text
AssignmentId
BookingId
LaboratoryId
CollectorUserId
AssignedAt
PlannedStart
Status
Notes
```

Possible assignment status:

```text
ASSIGNED
ACCEPTED
ON_THE_WAY
ARRIVED
COLLECTED
CANCELLED
```

---

# 41. Patient Collection Tracking

The patient can see:

```text
Booking Confirmed
Collector Assigned
Collector On The Way (when supported)
Sample Collected
Sample Received
Processing
Result Ready
```

Exact staff location should only be exposed if the product deliberately supports real-time tracking and privacy requirements are satisfied.

---

# 42. Sample Collection

The sample is a first-class entity.

Conceptually:

```text
Booking
   ↓
Sample
```

Sample fields may include:

```text
SampleId
BookingId
PatientId
TestId
SampleType
CollectionLocation
CollectedAt
CollectedBy
ContainerType
SampleStatus
RejectionReason
ReceivedAt
```

---

# 43. Sample Identification

At collection, the system should support a reliable link between:

```text
Patient
+
Booking
+
Test
+
Sample
```

The sample should have a unique identifier/reference.

Example:

```text
Sample ID: S-2026-000123
```

A barcode/QR workflow can be added later if not included in the first prototype.

---

# 44. Sample Statuses

Recommended:

```text
PENDING_COLLECTION
COLLECTED
IN_TRANSIT
RECEIVED
REJECTED
PROCESSING
COMPLETED
DISPOSED / ARCHIVED according to operational policy
```

---

# 45. Sample Rejection

A laboratory may reject a sample for operational reasons such as:

```text
Insufficient Sample
Incorrect Container
Damaged Container
Improper Collection
Delayed / Compromised Sample
Identification Problem
Other Approved Reason
```

The system should store:

```text
RejectedAt
RejectedBy
Reason
Notes
```

Patient and/or doctor notification should follow the defined workflow.

---

# 46. Recollection Workflow

If a sample must be recollected:

```text
Sample Rejected
      ↓
Recollection Required
      ↓
Patient Notification
      ↓
New Collection Booking / Reschedule
      ↓
New Sample
```

Never silently change a rejected sample into a valid one.

---

# 47. Laboratory Receipt

When sample arrives at the laboratory:

```text
COLLECTED / IN_TRANSIT
      ↓
RECEIVED
```

Record:

```text
ReceivedAt
ReceivedBy
SampleCondition
Accepted / Rejected
```

---

# 48. Processing Workflow

After accepted sample:

```text
RECEIVED
   ↓
PROCESSING
   ↓
RESULT DRAFT
```

Depending on the test, the internal workflow may contain multiple technical steps, but the patient should not need to understand them.

---

# 49. Result Creation

A laboratory result is linked to:

```text
Patient
Booking
Test
Sample
Laboratory
```

Recommended structure:

```text
LaboratoryResult
├── ResultId
├── PatientId
├── BookingId
├── TestId
├── SampleId
├── LaboratoryId
├── ResultStatus
├── StructuredValues
├── ReportFile
├── CreatedAt
├── ValidatedAt
├── PublishedAt
└── PublishedBy
```

---

# 50. Structured Result vs Report File

Support both:

```text
Structured Result Values
```

and:

```text
Final Laboratory Report Document
```

This allows:

```text
Patient UI
→ readable values

Doctor UI
→ structured result + report

Download
→ official report file where applicable
```

---

# 51. Result Validation

Before publishing:

```text
Result Draft
    ↓
Authorized Review / Validation
    ↓
VALIDATED
    ↓
PUBLISHED
```

The exact reviewer role depends on the laboratory's operational and clinical model.

---

# 52. Result Statuses

```text
DRAFT
UNDER_REVIEW
VALIDATED
PUBLISHED
AMENDED
CANCELLED
```

An amendment should create an auditable change rather than silently overwriting the original published record.

---

# 53. Result Publishing

After authorized validation:

```text
Result
   ↓
PUBLISHED
   ↓
Patient Notification
   ↓
Patient Medical Timeline
```

---

# 54. Patient Result Page

Patient sees:

```text
Laboratory
Test
Date Collected
Date Resulted
Result Values
Reference Ranges
Comments / Instructions when included by authorized laboratory
Report Document
Status
```

Do not present an automatically generated “diagnosis” from raw lab numbers unless a separate clinically governed workflow explicitly supports that capability.

---

# 55. Reference Ranges

Where the laboratory provides them, a structured result can include:

```text
Value
Unit
Reference Range
Flag
```

Example:

```text
Hemoglobin
Value: 13.8
Unit: g/dL
Reference: Laboratory-defined range
Flag: Normal / High / Low where supplied
```

The reference range should belong to the relevant test/result context and not be hardcoded globally when laboratory-specific ranges are required.

---

# 56. Doctor Access to Results

The doctor may view relevant laboratory results when the doctor has an authorized patient-care relationship.

Concept:

```text
Patient
   ↓
Lab Result
   ↓
Authorization Check
   ↓
Doctor
```

The doctor should not automatically receive the entire laboratory history of every patient in the system.

---

# 57. Patient Medical Timeline Integration

A completed laboratory workflow creates an event:

```text
Laboratory Test
    ↓
Sample Collected
    ↓
Result Published
```

The patient's timeline can display:

```text
CBC
ABC Laboratory
Result Ready
05 Oct
```

---

# 58. Result Download

If the platform provides the official report file:

```text
[ View Result ]
[ Download Report ]
```

The backend must authorize the user before serving the file.

Never expose report files through predictable public URLs.

---

# 59. Laboratory Notifications

Laboratory users receive:

```text
New Booking
Booking Confirmation
Collection Assignment
Collection Cancelled
Sample Received
Sample Rejected
Result Awaiting Validation
Result Published / Workflow Update
Refund
Payment
New Review
Complaint
Document Verification
```

Patient receives:

```text
Booking Confirmed
Collection Reminder
Collector Assigned
Collector On The Way (if supported)
Sample Collected
Sample Received
Sample Recollection Required
Result Ready
Order/Payment Update
```

---

# 60. Appointment Reminders

Reminders should be generated from server-side booking data.

Possible events:

```text
Upcoming Lab Appointment
Home Collection Reminder
Collection Rescheduled
Collection Cancelled
```

---

# 61. Patient Cancellation

The patient may cancel according to policy.

Possible states:

```text
PENDING → CANCELLED_BY_PATIENT
CONFIRMED → CANCELLED_BY_PATIENT
```

The financial effect depends on:

```text
Current State
Time of Cancellation
Payment Status
Collection Assignment
Platform Policy
```

---

# 62. Laboratory Cancellation

The laboratory may cancel when allowed.

Possible reasons:

```text
Test Unavailable
Collection Capacity Problem
Operational Issue
Staff Unavailable
Other
```

Store:

```text
CancelledBy
Reason
CancelledAt
RefundStatus
```

---

# 63. No-Show

Possible states:

```text
NO_SHOW_PATIENT
NO_SHOW_LAB
```

The exact definition depends on whether the service is:

```text
Lab Visit
```

or:

```text
Home Collection
```

---

# 64. Payment Workflow

Recommended:

```text
Booking Created
      ↓
Price Calculated
      ↓
Payment Initiated
      ↓
Payment Verified
      ↓
Booking Confirmed
      ↓
Service Completed
      ↓
Settlement
```

For pay-at-location/pickup models, the payment method/status can be different.

---

# 65. Financial Transaction

Store independently from booking status:

```text
TransactionId
BookingId
PatientId
LaboratoryId
GrossAmount
CollectionFee
Discount
RefundAmount
PlatformFee
NetAmount
PaymentMethod
PaymentStatus
CreatedAt
```

---

# 66. Refund Workflow

```text
Eligible Cancellation / Failed Service
          ↓
Refund Created
          ↓
PROCESSING
          ↓
COMPLETED / FAILED
```

Do not rewrite the original transaction to make it look like the refund never happened.

---

# 67. Laboratory Earnings

Dashboard may show:

```text
Gross Revenue
Platform Fees
Discounts
Refunds
Net Earnings
Pending Settlement
Available Balance
```

---

# 68. Withdrawal

If enabled:

```text
Available Balance
      ↓
Withdrawal Request
      ↓
PENDING
      ↓
PROCESSING
      ↓
COMPLETED
```

Alternative:

```text
PENDING → REJECTED
```

Withdrawal history must remain auditable.

---

# 69. Ratings

After an eligible completed laboratory service:

```text
COMPLETED
   ↓
Rate Laboratory
```

Possible criteria:

```text
Service Quality
Accuracy / Trust Experience
Punctuality
Sample Collection Experience
Communication
Overall
```

The platform should only expose clinically meaningful criteria after product review.

---

# 70. Complaints

Patient can create:

```text
Complaint
   ↓
Related Booking
   ↓
Category
   ↓
Description
   ↓
Attachments
```

Possible categories:

```text
Booking
Delay
Collection
Laboratory Service
Result Delivery
Pricing
Payment
Staff Conduct
Other
```

---

# 71. Complaint Lifecycle

```text
SUBMITTED
   ↓
UNDER_REVIEW
   ↓
IN_PROGRESS
   ↓
RESOLVED
   ↓
CLOSED
```

Laboratory staff may submit a response, but should not delete the official complaint.

---

# 72. Laboratory Staff Permissions

## Owner / Manager

Can manage:

```text
Organization
Staff
Services
Tests
Inventory/Operational Resources
Orders
Financials
Complaints
```

## Professional

Can access:

```text
Assigned/authorized diagnostic workflows
Results awaiting validation
Relevant test data
```

## Collector

Can access:

```text
Assigned collection tasks
Necessary patient identification/contact
Service Address
Test/collection requirements
Collection Status
```

## Operational Staff

Can access only the workflows necessary for their assigned duties.

---

# 73. Strict Access Rule for Patient Data

The laboratory must not expose all patient health information simply because an employee is part of the laboratory.

Use:

```text
User
 ↓
Role
 ↓
Organization
 ↓
Assignment / Care Context
 ↓
Purpose
 ↓
Allowed Data Scope
```

Example:

```text
Collector
→ Needs address + booking + collection information

Result Reviewer
→ Needs test + sample + result-related information

Finance Staff
→ Needs payment/transaction information
```

---

# 74. Patient Address Privacy

Public:

```text
Approximate Area
Distance
```

Confirmed home-collection workflow:

```text
Exact Address → Authorized Collection Staff
```

The exact address should not appear in public search results.

---

# 75. Sample Privacy

A sample identifier must not expose unnecessary health information.

Prefer:

```text
S-2026-000123
```

rather than putting diagnosis or other medical data directly into the identifier.

---

# 76. Lab Result Privacy

Results are highly sensitive.

Access should be limited to:

```text
Patient
Authorized Doctor
Authorized Laboratory Personnel
Admin when necessary and authorized
```

The exact permission set depends on the business and clinical workflow.

---

# 77. Result Corrections / Amendments

If a published result must be corrected:

```text
Published Result
      ↓
Amendment Requested
      ↓
Authorized Review
      ↓
AMENDED RESULT
      ↓
Audit History Preserved
```

Do not silently replace a published result.

---

# 78. Sample Recollection and Result Traceability

A recollection must create a new sample identity.

Example:

```text
Booking #100
  ├── Sample #S001 → Rejected
  └── Sample #S002 → Accepted
                         ↓
                    Result #R002
```

This preserves traceability.

---

# 79. Laboratory Order / Booking State History

Do not rely only on current status.

Store:

```text
BookingStatusHistory
├── PreviousStatus
├── NewStatus
├── Actor
├── Timestamp
└── Reason
```

Example:

```text
PENDING       09:00
CONFIRMED     09:04
ASSIGNED      09:10
COLLECTED     10:02
PROCESSING    11:00
COMPLETED     15:20
```

---

# 80. Backend Domain Entities

Recommended starting entities:

```text
User
Role
Laboratory
LaboratoryStaff
LaboratoryDocument
LaboratoryWorkingHour

DiagnosticTest
DiagnosticCategory
LaboratoryTestOffering

LabReferral
LabBooking
LabBookingItem

CollectionSlot
CollectionAssignment

Sample
SampleStatusHistory

LaboratoryResult
ResultValue
ResultAttachment
ResultStatusHistory

Payment
Transaction
Refund
ProviderEarning
Withdrawal

Review
Complaint
Notification
AuditLog
```

The prototype can simplify entities while maintaining these boundaries.

---

# 81. Database Relationship Overview

```text
Laboratory
   │
   ├── LaboratoryStaff
   ├── LaboratoryDocument
   ├── LaboratoryWorkingHour
   └── LaboratoryTestOffering
          │
          └── DiagnosticTest

Patient
   │
   ├── LabReferral
   └── LabBooking
          │
          ├── Test
          ├── CollectionAssignment
          └── Sample
                 │
                 └── LaboratoryResult

LabBooking
   ├── Payment / Transaction
   ├── Review
   └── Complaint
```

---

# 82. Suggested API Groups

```text
/api/laboratories/*
/api/laboratory/staff/*
/api/laboratory/tests/*
/api/laboratory/offers/*
/api/laboratory/availability/*
/api/laboratory/bookings/*
/api/laboratory/collections/*
/api/laboratory/samples/*
/api/laboratory/results/*
/api/laboratory/financials/*
/api/laboratory/refunds/*
/api/laboratory/withdrawals/*
/api/laboratory/reviews/*
/api/laboratory/complaints/*
/api/notifications/*
```

---

# 83. Example Laboratory APIs

## Profile

```http
GET  /api/laboratories/me
PUT  /api/laboratories/me
```

## Tests

```http
GET  /api/laboratory/tests
POST /api/laboratory/offers
PUT  /api/laboratory/offers/{id}
POST /api/laboratory/offers/{id}/deactivate
```

## Availability

```http
GET /api/laboratory/availability
PUT /api/laboratory/availability
```

## Bookings

```http
GET  /api/laboratory/bookings
GET  /api/laboratory/bookings/{id}
POST /api/laboratory/bookings/{id}/accept
POST /api/laboratory/bookings/{id}/reject
POST /api/laboratory/bookings/{id}/cancel
```

## Collection

```http
POST /api/laboratory/bookings/{id}/assign-collector
POST /api/laboratory/collections/{id}/start
POST /api/laboratory/collections/{id}/complete
```

## Samples

```http
POST /api/laboratory/samples
PUT  /api/laboratory/samples/{id}
POST /api/laboratory/samples/{id}/receive
POST /api/laboratory/samples/{id}/reject
```

## Results

```http
GET  /api/laboratory/results
GET  /api/laboratory/results/{id}
POST /api/laboratory/results/{id}/draft
POST /api/laboratory/results/{id}/validate
POST /api/laboratory/results/{id}/publish
POST /api/laboratory/results/{id}/amend
```

## Financials

```http
GET  /api/laboratory/financials
GET  /api/laboratory/transactions
GET  /api/laboratory/withdrawals
POST /api/laboratory/withdrawals
```

---

# 84. Frontend Pages — Laboratory Staff

Recommended:

```text
/Laboratory/Register
/Laboratory/Verification
/Laboratory/Dashboard
/Laboratory/Profile
/Laboratory/Staff
/Laboratory/Documents
/Laboratory/Tests
/Laboratory/Tests/{id}
/Laboratory/Offers
/Laboratory/Availability
/Laboratory/Bookings
/Laboratory/Bookings/{id}
/Laboratory/Collections
/Laboratory/Collections/{id}
/Laboratory/Samples
/Laboratory/Samples/{id}
/Laboratory/Results
/Laboratory/Results/{id}
/Laboratory/Reviews
/Laboratory/Complaints
/Laboratory/Financials
/Laboratory/Transactions
/Laboratory/Withdrawals
/Laboratory/Notifications
/Laboratory/Settings
```

---

# 85. Frontend Pages — Patient Laboratory

```text
/LaboratoryTests
/LaboratoryTests/Search
/LaboratoryTests/{id}
/Laboratories/Nearby
/Laboratories/{id}
/LabReferrals
/LabBookings
/LabBookings/{id}
/LabResults
/LabResults/{id}
```

---

# 86. Laboratory Booking UI

Patient-facing booking page should show:

```text
Test
Laboratory
Fulfillment Method
Date
Time
Address (if home collection)
Test Price
Collection Fee
Discount
Total
Preparation Instructions
```

The patient confirms only after seeing the final price and important preparation information.

---

# 87. Preparation Instructions

If a test requires preparation, display it clearly before confirmation.

Example:

```text
Preparation
Fasting may be required according to the laboratory's approved instruction.
Please follow the laboratory's provided preparation requirements.
```

The application should display the laboratory-approved instruction instead of inventing medical instructions.

---

# 88. Collector Dashboard

If collector users are included as a separate interface:

```text
Today's Tasks
Upcoming Collections
Assigned Visits
Current Visit
Completed Collections
Notifications
```

Each assigned task:

```text
Booking ID
Patient Display Name
Address
Time
Test
Collection Instructions
Status
```

Sensitive information must remain limited to what is operationally required.

---

# 89. Collector Workflow

```text
Assigned
   ↓
Accept Assignment
   ↓
On The Way
   ↓
Arrived
   ↓
Verify Patient / Booking
   ↓
Collect Sample
   ↓
Label / Identify Sample
   ↓
Complete Collection
   ↓
Transfer Sample
   ↓
Lab Receives Sample
```

The exact identity verification procedure must be defined by the real operational process.

---

# 90. Collection Failure

Possible outcomes:

```text
PATIENT_UNAVAILABLE
ADDRESS_INACCESSIBLE
PATIENT_REFUSED
COLLECTION_FAILED
SAMPLE_NOT_OBTAINED
OTHER
```

The reason must be recorded and communicated according to policy.

---

# 91. Patient Refuses / Changes Mind

The collection flow should not simply mark the booking completed.

Instead:

```text
Collector Arrived
      ↓
Patient Refused / Unable to Collect
      ↓
Collection Outcome Recorded
      ↓
Financial Policy Applied
```

---

# 92. Result Publication Safeguards

Before publishing a result, verify:

```text
Correct Laboratory
Correct Patient
Correct Booking
Correct Test
Correct Sample
Authorized Reviewer
Complete Required Fields
```

Only then:

```text
VALIDATED → PUBLISHED
```

---

# 93. Result Attachment Security

If a PDF/report exists:

```text
Private Storage
     ↓
Authorization Check
     ↓
Secure Download / Streaming
```

Avoid:

```text
public/uploads/lab/result123.pdf
```

where anyone who guesses the URL could access the document.

---

# 94. Result Versioning

A published result can have versions:

```text
Result v1
   ↓
Amendment
   ↓
Result v2
```

History remains accessible to authorized users.

---

# 95. Duplicate Result Protection

The system should prevent accidental duplicate publication for the same result context.

Use a clear relationship:

```text
Booking
+
Sample
+
Test
→ Result
```

Any duplicate submission should be detected and handled explicitly.

---

# 96. Error Scenarios

## Booking

```text
Test unavailable
Slot unavailable
Lab closed
Outside collection radius
Booking duplicate
Payment failure
```

## Collection

```text
Collector unavailable
Patient unavailable
Address problem
Collection failed
Sample rejected
```

## Result

```text
Wrong patient association attempt
Missing required result fields
Validation failed
Publish authorization failure
Report upload failure
```

---

# 97. Security Test Scenarios

```text
Patient accesses another patient's lab result
Patient downloads unauthorized report
Collector accesses unrelated booking
Collector sees unnecessary medical information
Lab staff accesses another laboratory
Staff user modifies result without permission
Unauthorized user publishes result
Unauthorized user changes stock/financial data
Client submits forged price
Client submits forged patient ID
Client submits forged laboratory ID
```

All should be rejected by server-side authorization.

---

# 98. Concurrency Test Scenarios

```text
Two patients book the same collection slot
Two staff members accept the same booking
Two users assign the same collector capacity
Two users update a result simultaneously
Two users cancel the same booking
Duplicate payment callback
Duplicate result publish request
```

The backend must handle concurrency safely.

---

# 99. Financial Test Scenarios

```text
Successful payment
Failed payment
Duplicate payment callback
Cancellation refund
Refund failure
Partial refund if supported
Settlement calculation
Withdrawal request
Withdrawal failure
Historical transaction integrity
```

---

# 100. Result Integrity Test Scenarios

```text
Correct Patient → Correct Result
Wrong Patient → Blocked
Correct Sample → Result Allowed
Rejected Sample → Result blocked until valid workflow
Unvalidated Result → Not published
Published Result → Accessible only to authorized users
Amended Result → Versioned and audited
```

---

# 101. Data Access Matrix — Example

| Resource | Patient | Doctor | Lab Owner | Lab Professional | Collector | Admin |
|---|---|---|---|---|---|---|
| Own Profile | Full | Full | Full | Full | Full | Scoped |
| Own Booking | Full | Related | Full | Related | Assigned | Scoped |
| Patient Full Medical History | Own | Relevant/Authorized | No | No | No | Restricted |
| Prescription | Own | Related | Only if needed | Only if needed | No | Restricted |
| Sample | Own | Relevant | Full operational | Full operational | Assigned | Scoped |
| Result | Own | Relevant/Authorized | Operational | Relevant/Authorized | No | Restricted |
| Financials | Own transactions | Own | Full org | Limited | No | Full controlled |
| Verification Documents | Own where applicable | Own | Org | Professional own | Own if required | Full review |

This table is a design baseline; exact permissions must be finalized with the team.

---

# 102. State Machine — Complete Lab Journey

```text
                    PATIENT REQUEST
                           ↓
                        PENDING
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
         CONFIRMED     REJECTED      CANCELLED
              │
              ↓
       COLLECTION_ASSIGNED
              │
              ↓
          IN_PROGRESS
              │
              ↓
       SAMPLE COLLECTED
              │
              ↓
        SAMPLE RECEIVED
              │
         ┌────┴─────┐
         ↓          ↓
     REJECTED    PROCESSING
         │          │
         │          ↓
         │      RESULT DRAFT
         │          │
         │          ↓
         │     UNDER REVIEW
         │          │
         │          ↓
         │       VALIDATED
         │          │
         │          ↓
         │       PUBLISHED
         │          │
         │          ↓
         │   PATIENT NOTIFIED
         │
         └→ RECOLLECTION REQUIRED
                   │
                   ↓
                 NEW SAMPLE
```

---

# 103. Complete Patient Journey

```text
Patient Login
      ↓
Search Laboratory Test
      ↓
Choose Laboratory
      ↓
Compare
├── Price
├── Distance
├── Rating
├── Availability
└── Home Collection
      ↓
Choose Fulfillment Method
      ↓
Choose Date / Slot
      ↓
Confirm Address if Home Collection
      ↓
Read Preparation Instructions
      ↓
Review Price
      ↓
Payment / Confirmation
      ↓
Booking Created
      ↓
Laboratory Confirms
      ↓
Collector Assigned (if home collection)
      ↓
Collection
      ↓
Sample Received
      ↓
Processing
      ↓
Result Validation
      ↓
Result Published
      ↓
Patient Notification
      ↓
View / Download Result
      ↓
Medical Timeline Updated
      ↓
Rate Laboratory / Complaint if needed
```

---

# 104. Complete Doctor → Lab → Patient Journey

```text
Doctor Consultation
      ↓
Doctor Recommends / Requests Test
      ↓
Lab Request Linked to Patient
      ↓
Patient Opens Request
      ↓
Choose Laboratory
      ↓
Choose Home Collection / Lab Visit
      ↓
Book
      ↓
Sample Collection
      ↓
Processing
      ↓
Result Published
      ↓
Patient Receives Result
      ↓
Doctor Can View Relevant Result
      ↓
Doctor Uses Result During Authorized Follow-up
```

---

# 105. Complete Home Collection Journey

```text
Patient Chooses Home Collection
          ↓
Address Validated
          ↓
Area Within Laboratory Coverage
          ↓
Slot Available
          ↓
Booking Confirmed
          ↓
Collector Assigned
          ↓
Collector Accepts
          ↓
Collector On The Way
          ↓
Collector Arrives
          ↓
Patient / Booking Verified
          ↓
Sample Collected
          ↓
Sample Identified
          ↓
Collection Completed
          ↓
Sample Transferred
          ↓
Laboratory Receives Sample
          ↓
Processing
          ↓
Result
```

---

# 106. Laboratory Operational Dashboard — Recommended Widgets

```text
Today's Bookings
Pending Confirmations
Home Collections
Collections In Progress
Samples Awaiting Receipt
Rejected Samples
Tests Processing
Results Awaiting Validation
Results Published
Open Complaints
Today's Revenue
Pending Settlement
```

The dashboard should prioritize exceptions and work requiring action.

---

# 107. Laboratory Search Ranking

Possible ranking signals:

```text
Verification
Test Availability
Distance
Price
Rating
Review Count
Home Collection
Availability
Operational Completion Rate
```

Ranking should be transparent enough for product owners to explain why a laboratory appears higher.

---

# 108. Service Area Rules

A laboratory may define:

```text
Cities
Districts
Collection Radius
```

When a patient location is outside coverage:

```text
Outside Service Area
```

and the patient should be offered alternative laboratories where available.

---

# 109. Admin Controls

Admin may manage:

```text
Laboratory Verification
Laboratory Suspension
Professional Verification
Test Master Catalog
Test Categories
Result Dispute Workflows
Complaints
Reviews
Financial Disputes
Audit Logs
```

Admin actions should be controlled and auditable.

---

# 110. Laboratory Suspension

If suspended:

```text
Laboratory Status = SUSPENDED
```

New patient bookings should be blocked or restricted according to policy.

Existing bookings should enter an explicit resolution workflow.

---

# 111. Staff Removal

When a staff member is removed:

```text
ACTIVE
  ↓
REMOVED
```

Access must be revoked immediately.

Historical actions remain in the audit record.

---

# 112. Background Jobs / System Processing

The system may use background workers for:

```text
Expired Reservation Cleanup
Booking Reminders
Collection Reminders
Unassigned Collection Alerts
Result Ready Notifications
Document Expiry Alerts
Refund Processing
Financial Reconciliation
```

These are system jobs, not browser-only timers.

---

# 113. Performance Considerations

Important optimized queries include:

```text
Search tests
Nearby labs
Available collection slots
Pending bookings
Samples by status
Results by patient
Results awaiting validation
Financial transactions
```

Use pagination and indexing where appropriate.

---

# 114. Database Indexing Candidates

Depending on database/provider:

```text
DiagnosticTest.Name
DiagnosticTest.Code
Laboratory.City
Laboratory.Status
LaboratoryTestOffering.TestId
LaboratoryTestOffering.LaboratoryId
LabBooking.PatientId
LabBooking.LaboratoryId
LabBooking.Status
LabBooking.CollectionDate
Sample.BookingId
Sample.Status
LaboratoryResult.PatientId
LaboratoryResult.Status
```

Geospatial querying should use an appropriate strategy for the chosen database.

---

# 115. Observability

Log important operational failures without putting sensitive medical values into normal application logs.

Useful events:

```text
Booking creation failure
Slot conflict
Collection assignment failure
Sample rejection
Result validation failure
Result publication failure
Payment failure
Refund failure
Unauthorized access attempt
```

Never log passwords, authentication tokens, or unnecessary sensitive medical content.

---

# 116. API Validation

Never trust client-submitted:

```text
PatientId
LaboratoryId
Price
Total
Status
SampleStatus
ResultStatus
PublishedBy
PaymentStatus
```

The backend derives or validates them from authenticated context and server-side state.

---

# 117. Idempotency / Duplicate Protection

Critical operations should be protected against duplicate execution:

```text
Create Booking
Create Payment
Receive Payment Callback
Publish Result
Complete Collection
Create Refund
Request Withdrawal
```

A repeated request should not create duplicate records or duplicate money movement.

---

# 118. MVP Scope

## Include in the Web Prototype

```text
Laboratory Registration
Verification
Profile
Staff / Basic Roles
Test Catalog
Laboratory Test Offers
Prices
Working Hours
Home Collection
Patient Search
Nearby Labs
Test Booking
Collection Scheduling
Sample Tracking
Basic Result Creation
Result Validation / Publication
Patient Result Page
Medical Timeline Connection
Notifications
Payments / Transactions
Ratings
Complaints
Admin Controls
```

## Future Enhancements

```text
Advanced LIS Integration
Barcode / QR Sample Tracking
Automated Analyzer Integration
Real-Time Collector GPS
Advanced Route Optimization
Insurance Integration
Hospital Integration
External Referral Networks
Advanced Analytics
AI-assisted Result Summaries
```

Any AI feature must be separately governed and must not silently replace professional clinical interpretation.

---

# 119. Definition of Done — Laboratory Module

The module is complete when:

```text
[ ] Laboratory registration works.
[ ] Staff accounts can be linked with correct permissions.
[ ] Verification works.
[ ] Public profile works.
[ ] Test catalog works.
[ ] Laboratory test offerings work.
[ ] Prices are server-controlled.
[ ] Home collection configuration works.
[ ] Working hours work.
[ ] Collection slots work.
[ ] Patient can discover laboratories.
[ ] Patient can compare test providers.
[ ] Patient can book a test.
[ ] Doctor-linked test request can reach the patient workflow.
[ ] Laboratory can accept/reject bookings.
[ ] Double booking is prevented.
[ ] Collector assignment works when enabled.
[ ] Collection status works.
[ ] Sample entity is created and traceable.
[ ] Sample rejection/recollection works.
[ ] Laboratory receives sample.
[ ] Result can be created.
[ ] Result can be validated by authorized staff.
[ ] Result can be published.
[ ] Result is linked to correct patient/test/sample.
[ ] Patient receives notification.
[ ] Patient can view/download result if enabled.
[ ] Result reaches medical timeline.
[ ] Authorized doctor can view relevant result.
[ ] Payments and transactions are traceable.
[ ] Refunds work according to policy.
[ ] Ratings only work after eligible completion.
[ ] Complaints are trackable.
[ ] Sensitive files are protected.
[ ] Object-level authorization tests pass.
[ ] Concurrency tests pass.
[ ] Critical workflows have automated tests.
```

---

# 120. Core Business Rules Checklist

```text
[ ] Only verified active laboratories are publicly bookable.
[ ] Laboratory staff are organization-scoped.
[ ] Only approved test offerings can be booked.
[ ] Patient location is protected.
[ ] Exact home address is disclosed only to authorized collection staff.
[ ] Booking slots cannot be double-booked.
[ ] Prices are calculated server-side.
[ ] Historical booking prices are snapshotted.
[ ] Sample identifiers are unique.
[ ] Rejected samples cannot silently become valid samples.
[ ] Recollection creates a traceable new sample.
[ ] Results cannot be published without required validation.
[ ] Published results are not silently overwritten.
[ ] Result amendments are versioned/audited.
[ ] Unauthorized users cannot access patient results.
[ ] Staff cannot access unrelated patients.
[ ] Financial records are independent and auditable.
[ ] Refunds are separate financial events.
[ ] Notifications do not expose unnecessary health information.
[ ] Critical state transitions are server-validated.
[ ] Duplicate operations are safely handled.
```

---

# 121. Full Laboratory Workflow — Final Diagram

```text
                         LABORATORY
                              │
                  ┌───────────┴───────────┐
                  │                       │
             Registration              Login/Auth
                  │
                  ↓
             Verification
                  │
                  ↓
               APPROVED
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
   Lab Profile         Staff Management
        │
        ↓
    Test Catalog
        │
        ↓
 Test Offerings + Prices
        │
        ↓
 Working Hours + Collection Slots
        │
        ↓
    Public Discovery
        │
        ↓
 Patient Search / Doctor Referral
        │
        ↓
       BOOKING
        │
    ┌───┼────────┐
    ↓   ↓        ↓
 ACCEPT REJECT CANCEL
    │
    ↓
 CONFIRMED
    │
    ↓
COLLECTION ASSIGNMENT
    │
    ↓
 SAMPLE COLLECTION
    │
    ↓
 SAMPLE IDENTIFICATION
    │
    ↓
 SAMPLE RECEIVED
    │
    ↓
 PROCESSING
    │
    ↓
 RESULT DRAFT
    │
    ↓
 RESULT VALIDATION
    │
    ↓
 RESULT PUBLISHED
    │
    ├──────────────→ Patient Notification
    │
    ├──────────────→ Patient Medical Timeline
    │
    └──────────────→ Authorized Doctor View

    Then:
       Payment / Settlement
              ↓
          Rating / Complaint
              ↓
          Financial History
```

---

# 122. Final Product Definition

The TechCare Laboratory module is a complete **diagnostic service lifecycle**, not simply a laboratory directory.

Its core value comes from connecting:

```text
Verified Laboratory
        +
Diagnostic Test Catalog
        +
Patient Discovery
        +
Doctor Referral
        +
Booking
        +
Home Collection
        +
Sample Tracking
        +
Result Validation
        +
Patient Medical Record
        +
Financial / Quality Workflows
        +
Privacy / Authorization
        =
Connected Diagnostic Care
```

The defining journey is:

> **Find the right test → choose a trusted laboratory → book the service → collect the sample at the laboratory or the patient's home → track the sample → publish the validated result → connect it to the patient's ongoing healthcare journey.**

---

# 123. Implementation Principle

The Laboratory owner in the five-person Full-Stack .NET team owns the module end-to-end:

```text
Database
    ↓
ASP.NET Core Domain / Application Logic
    ↓
API
    ↓
Frontend
    ↓
Validation
    ↓
Authorization
    ↓
Tests
    ↓
Integration
```

Ownership does not mean isolated work. The module must integrate with:

```text
Patient
Doctor
Payments
Notifications
Reviews / Complaints
Admin
Medical Timeline
Authentication
```

Every pull request should receive code review from another team member before merge.

---

# 124. Final Laboratory Journey in One View

```text
REGISTER
  ↓
VERIFY
  ↓
CONFIGURE LAB
  ↓
ADD TESTS
  ↓
SET PRICES
  ↓
SET HOURS / COLLECTION
  ↓
BECOME DISCOVERABLE
  ↓
RECEIVE BOOKING
  ↓
CONFIRM
  ↓
ASSIGN COLLECTION
  ↓
COLLECT SAMPLE
  ↓
RECEIVE SAMPLE
  ↓
PROCESS
  ↓
VALIDATE RESULT
  ↓
PUBLISH RESULT
  ↓
NOTIFY PATIENT
  ↓
UPDATE MEDICAL TIMELINE
  ↓
SETTLE FINANCIALS
  ↓
RATING / COMPLAINT
  ↓
REPORTING / HISTORY
```

---

## Document Status

This document is the complete functional reference for the TechCare Laboratory module and can be used as the basis for:

- Product Backlog
- User Stories
- Use Cases
- UI/UX Screens
- Entity Relationship Design
- ASP.NET Core APIs
- EF Core Models and Migrations
- Authorization Policies
- QA Test Cases
- Integration Testing
- Sprint Tasks
- End-to-End Demo Scenarios
