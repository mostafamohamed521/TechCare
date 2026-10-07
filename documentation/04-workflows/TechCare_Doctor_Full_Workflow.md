# TechCare — Doctor Full Workflow

> **Complete Doctor Workflow & Field Specification**
>
> This document defines the Doctor experience for the **TechCare Web Prototype** from registration and professional verification through availability, booking requests, home consultation, diagnosis, prescription, patient communication, financials, ratings, complaints, notifications, and account management.

---

## 1. Doctor Role Overview

The Doctor is a verified healthcare provider who can:

- Create a professional profile.
- Submit identity and professional credentials for verification.
- Define medical specialties and services.
- Define home-visit coverage areas.
- Set service prices and distance-pricing policies.
- Configure working hours and available time slots.
- Receive patient booking requests.
- Accept, reject, or allow requests to expire according to the configured response window.
- Access only the patient medical information authorized and relevant to the active consultation.
- Conduct a home consultation.
- Create a structured consultation record.
- Record diagnosis, treatment plan, and prescription where clinically appropriate.
- Complete the service.
- Review financial activity and withdrawals.
- Receive ratings and respond to platform-managed complaints where allowed.
- Manage profile, documents, availability, notifications, and security.

### Core Principle

The Doctor account is **not active in the patient marketplace until the required verification workflow is successfully completed**.

---

# 2. Complete Doctor Journey

```text
Open TechCare
      ↓
Create Doctor Account
      ↓
Verify Contact Information
      ↓
Complete Personal Profile
      ↓
Complete Professional Profile
      ↓
Upload Required Documents
      ↓
Submit Verification Request
      ↓
Admin Review
      ↓
Approved
      ↓
Configure Services & Prices
      ↓
Configure Location / Coverage Area
      ↓
Configure Working Hours
      ↓
Create Available Time Slots
      ↓
Doctor Becomes Discoverable
      ↓
Receive Patient Booking Request
      ↓
Accept / Reject / Request Expires
      ↓
Appointment Confirmed
      ↓
Travel to Patient
      ↓
Start Consultation
      ↓
View Authorized Patient Medical Context
      ↓
Record Consultation
      ↓
Diagnosis / Treatment / Prescription
      ↓
Complete Consultation
      ↓
Transaction / Earnings Updated
      ↓
Patient Rating / Complaint
      ↓
Doctor Reviews Performance & Financials
```

---

# 3. Entry Point

When the user chooses to join as a doctor:

```text
[ Login ]
[ Create Account ]
      ↓
Select Role
      ↓
[ Patient ] [ Doctor ] [ Nurse ] [ Pharmacy ] [ Laboratory ] [ Donor ]
      ↓
Doctor Registration
```

---

# 4. Doctor Registration

The registration process should be divided into logical steps instead of one very long form.

## Step 1 — Basic Account Information

| Field | Type | Required | Notes |
|---|---|---:|---|
| First Name | Text | ✅ | Legal/professional name as applicable. |
| Middle Name | Text | Optional | Useful for formal identity matching. |
| Last Name | Text | ✅ | Legal/professional name as applicable. |
| Email | Email | ✅ | Unique account identifier. |
| Mobile Number | Phone | ✅ | Used for verification and notifications. |
| Password | Password | ✅ | Strong password policy. |
| Confirm Password | Password | ✅ | Must match password. |
| Date of Birth | Date | ✅ | Used where age eligibility is relevant. |
| Gender | Select | Recommended | Configurable based on product requirements. |
| Profile Photo | Image | Recommended | Professional display photo. |

### Account Validation

The system should validate:

- Valid email format.
- Valid phone format.
- Unique email/mobile where required.
- Password strength.
- Matching password confirmation.
- No obvious duplicate account conflict.

---

# 5. Contact Verification

After basic account creation:

```text
Create Account
      ↓
Email / Phone Verification
      ↓
Verified
      ↓
Continue Profile
```

Possible fields/state:

| Field | Required | Description |
|---|---:|---|
| OTP | ✅ | One-time verification code. |
| Verification Status | System | Pending / Verified / Failed / Expired. |
| Verified At | System | Timestamp. |

---

# 6. Personal Information

## Personal Profile

| Field | Required | Notes |
|---|---:|---|
| Full Name | ✅ | Displayed according to approved profile policy. |
| Date of Birth | ✅ | Identity/eligibility information. |
| Gender | Recommended | Product-defined. |
| Profile Photo | Recommended | Public professional profile. |
| Mobile Number | ✅ | Verified contact. |
| Email | ✅ | Verified contact. |
| Nationality | Conditional | Collect only if required by the operating/legal model. |
| Short Bio | ✅ | Public professional introduction. |
| Languages | Recommended | Useful for patient matching. |
| Consultation Description | Recommended | How the doctor provides the service. |

---

# 7. Professional Information

This is the most important onboarding section after identity.

## Professional Fields

| Field | Type | Required | Notes |
|---|---|---:|---|
| Professional Title | Select/Text | ✅ | Example: Consultant / Specialist / General Practitioner where applicable. |
| Main Specialty | Select | ✅ | Selected from Admin-managed specialty catalog. |
| Sub-specialties | Multi-select | Recommended | Example: Sports injuries, spine, pediatric orthopedics. |
| Professional Summary | Textarea | ✅ | Public professional description. |
| Years of Experience | Number | Recommended | Validate non-negative value. |
| Professional Registration / License Number | Text | ✅ | Exact requirements depend on role and jurisdiction. |
| Workplace / Affiliation | Text | Optional | Hospital/clinic affiliation if applicable. |
| Education Summary | Textarea | ✅ | Professional education overview. |
| Certifications | Repeater | Recommended | Additional credentials. |
| Professional Languages | Multi-select | Recommended | Languages supported for consultation. |

---

# 8. Professional Documents

The doctor must have a dedicated secure document area.

## Documents

| Document | Required | Purpose |
|---|---:|---|
| National ID / Identity Document | ✅ | Identity verification. |
| Graduation Certificate | ✅ | Academic qualification verification. |
| Professional Practice License / Registration | ✅ | Professional authorization. |
| Specialty Certificate | Conditional | Required where specialty verification is applicable. |
| Membership / Professional Card | Conditional | If required by the selected operating model. |
| Other Credential | Conditional | Additional verification evidence. |

## Document Metadata

Each uploaded document should have system metadata:

```text
Document ID
Document Type
File Name
File Type
File Size
Uploaded By
Uploaded At
Verification Status
Reviewer
Reviewed At
Rejection Reason
Expiration Date (if applicable)
```

### Document States

```text
NOT UPLOADED
    ↓
UPLOADED
    ↓
UNDER REVIEW
    ↓
APPROVED
```

Alternative paths:

```text
UNDER REVIEW → REJECTED → RESUBMISSION
APPROVED → EXPIRED → RENEWAL REQUIRED
```

---

# 9. Verification Workflow

The doctor must understand exactly where the account stands.

## Status

```text
DRAFT
 ↓
SUBMITTED
 ↓
UNDER REVIEW
 ↓
APPROVED
```

Alternative outcomes:

```text
SUBMITTED
   ↓
REJECTED
   ↓
FIX REQUIRED
   ↓
RESUBMITTED
```

And later:

```text
APPROVED
   ↓
SUSPENDED / EXPIRED
```

## Doctor View

The doctor dashboard should clearly display:

```text
Verification Status: Approved ✅
```

or:

```text
Verification Status: Under Review ⏳
```

or:

```text
Verification Status: Action Required ⚠️
Reason: Professional license document could not be verified.
[Upload Replacement]
```

---

# 10. Doctor Marketplace Profile

Once approved, the doctor configures the profile patients will see.

## Public Profile Fields

| Field | Required |
|---|---:|
| Profile Photo | Recommended |
| Full Display Name | ✅ |
| Professional Title | ✅ |
| Specialty | ✅ |
| Sub-specialties | Recommended |
| Bio | ✅ |
| Experience | Recommended |
| Qualifications Summary | Recommended |
| Languages | Recommended |
| Service Types | ✅ |
| Service Prices | ✅ |
| Home Visit Availability | ✅ |
| Coverage Area | ✅ |
| Rating | System |
| Number of Reviews | System |
| Completed Services | System / controlled visibility |
| Verification Badge | System |
| Available Slots | System |

---

# 11. Services Management

The doctor must define what services patients can request.

## Add Service

| Field | Required | Notes |
|---|---:|---|
| Service Name | ✅ | Selected/created according to platform catalog policy. |
| Service Category | ✅ | Admin-controlled category. |
| Description | ✅ | Patient-facing explanation. |
| Base Price | ✅ | Monetary value. |
| Estimated Service Duration | Configurable | Product setting; do not confuse with request response time. |
| Home Visit Enabled | ✅ | Whether the doctor offers home visits. |
| Maximum Coverage Radius | Recommended | Operational limitation. |
| Distance Pricing Enabled | ✅ | Whether extra distance is charged. |
| Included Distance | Conditional | Optional free-distance threshold. |
| Price per Additional KM | Conditional | Additional distance rate. |
| Active | ✅ | Controls marketplace visibility. |

---

# 12. Distance Pricing

A doctor can configure a distance pricing policy.

### Example

```text
Base Service Price = 500 EGP
Included Distance = 5 KM
Additional Rate = 5 EGP / KM

Patient Distance = 20 KM

Chargeable Distance = 20 - 5 = 15 KM
Distance Fee = 15 × 5 = 75 EGP

Final Service Price = 500 + 75 = 575 EGP
```

### Important Rule

The final amount must be calculated by the **server** and shown to the patient before confirmation.

The booking should store the pricing inputs used at confirmation:

```text
Base Price Snapshot
Distance Snapshot
Included Distance Snapshot
Per-KM Rate Snapshot
Distance Fee Snapshot
Discount Snapshot
Final Total Snapshot
```

This prevents historical transactions from changing after the doctor edits pricing.

---

# 13. Service Coverage and Location

The doctor should define where home services are available.

## Coverage Fields

| Field | Required | Notes |
|---|---:|---|
| Primary Service Location | ✅ | Main starting location. |
| Address | ✅ | Stored privately as appropriate. |
| Latitude | ✅ / System | Geolocation coordinate. |
| Longitude | ✅ / System | Geolocation coordinate. |
| Coverage Radius | Recommended | Maximum service range. |
| Coverage Areas | Optional | Districts/cities/zones. |
| Location Accuracy | System | Optional GPS accuracy metadata. |
| Location Updated At | System | Auditability. |

### Privacy Rule

A doctor's exact private location should not automatically be exposed publicly. The patient can see an appropriate service area/distance before booking, while the exact destination/service details are disclosed only when necessary.

---

# 14. Working Hours

The doctor configures recurring availability.

## Weekly Schedule

```text
Saturday
  Start: 10:00 AM
  End: 06:00 PM

Sunday
  Start: 02:00 PM
  End: 08:00 PM
```

Fields:

| Field | Required |
|---|---:|
| Day | ✅ |
| Start Time | ✅ |
| End Time | ✅ |
| Active | ✅ |
| Breaks | Recommended |
| Service Type | Optional |

---

# 15. Time Slots

The doctor can define bookable slots from the working schedule.

Example:

```text
06:00 PM
06:30 PM
07:00 PM
07:30 PM
08:00 PM
```

Each slot has a system state:

```text
AVAILABLE
HELD
BOOKED
BLOCKED
COMPLETED
```

### Double-Booking Rule

The backend must prevent two patients from confirming the same slot.

This must be enforced transactionally; frontend checks alone are not sufficient.

---

# 16. Doctor Dashboard — Main Navigation

Recommended Doctor Sidebar:

```text
Dashboard

Requests
Appointments
Calendar
Patients
Consultations
Prescriptions
Services
Availability

Reviews
Complaints

Financials
Transactions
Withdrawals

Notifications

Profile
Documents
Verification

Settings
Security
Logout
```

---

# 17. Doctor Dashboard — Overview

The homepage should answer the doctor's immediate operational questions.

## Summary Cards

```text
New Requests
Pending Requests
Today's Appointments
Completed Services

Today's Earnings
Available Balance
Pending Balance

Average Rating
Response Rate
Completion Rate
```

## Quick Actions

```text
[ Manage Availability ]
[ View Requests ]
[ Add Service ]
[ View Calendar ]
[ View Financials ]
```

---

# 18. Requests Section

The Requests page is one of the most important pages in the Doctor dashboard.

## Tabs

```text
New
Pending
Accepted
Rejected
Expired
Cancelled
Completed
```

## Incoming Request Card

```text
Patient Name
Requested Service
Date
Time
Patient Area / Distance
Base Price
Distance Fee
Final Total
Created At
Expires At
Request Status
```

Actions:

```text
[ View Request ]
[ Accept ]
[ Reject ]
```

---

# 19. Five-Minute Request Rule

The proposed workflow uses a **configurable provider response window**, with 5 minutes as the initial product rule.

The important distinction is:

> **The five minutes is the time allowed to respond to the booking request. It is not the duration of the appointment.**

## State Flow

```text
PENDING
   ├── ACCEPTED
   ├── REJECTED
   └── EXPIRED
```

The system should store:

```text
requested_at
expires_at
responded_at
response_type
```

### Expired Request

If no response is received before `expires_at`:

```text
PENDING → EXPIRED
```

The doctor cannot accept an already expired request.

The patient can then search for another available provider.

---

# 20. Accepting a Booking

Before accepting, the doctor sees:

```text
Patient
Service
Date
Time
Distance
Price Breakdown
Relevant Notes
```

Doctor selects:

> **Accept Request**

System validates:

- Doctor is still active and verified.
- Slot is still available.
- Request is not expired.
- Doctor is not blocked/suspended.
- Required service is still active.

If valid:

```text
PENDING
   ↓
ACCEPTED
   ↓
CONFIRMED
```

Patient receives confirmation.

---

# 21. Rejecting a Booking

If the doctor rejects the request:

```text
PENDING → REJECTED
```

Recommended optional rejection categories:

- Schedule conflict
- Outside coverage area
- Service unavailable
- Personal unavailability
- Other

### Rejection Reason

The system may require a reason category and optionally a short note.

The patient should receive a clear notification without exposing inappropriate private information.

---

# 22. Appointment Section

After acceptance, the booking becomes an appointment.

## Appointment Details

```text
Appointment ID
Patient
Service
Date
Time
Address / Service Destination
Distance
Price
Status
Notes
Created At
Confirmed At
```

## Appointment States

```text
CONFIRMED
   ↓
STARTED
   ↓
COMPLETED
```

Alternative:

```text
CONFIRMED → CANCELLED
```

---

# 23. Calendar

The doctor should have:

- Day view
- Week view
- Month view
- Available slots
- Booked appointments
- Blocked times
- Breaks
- Cancelled appointments

### Calendar Actions

```text
[ Add Availability ]
[ Block Time ]
[ Edit Slot ]
[ View Appointment ]
```

The doctor must not be able to edit a slot in a way that invalidates an already-confirmed appointment without a defined cancellation/rescheduling workflow.

---

# 24. Before the Visit

Doctor can open the appointment and view:

```text
Patient Name
Service
Appointment Time
Service Address
Distance
Contact Method
Relevant Notes
```

Where the product supports it, the doctor can see navigation information necessary for the visit.

---

# 25. Starting the Consultation

The doctor clicks:

> **Start Consultation**

The appointment changes:

```text
CONFIRMED → STARTED
```

A dedicated Consultation Workspace opens.

---

# 26. Consultation Workspace

The consultation should not be treated as a normal social chat.

Recommended layout:

```text
LEFT PANEL
Patient Summary

CENTER PANEL
Consultation Workspace

RIGHT PANEL
Previous Relevant Records
```

---

# 27. Patient Context Available to Doctor

The doctor should see only the information permitted by the product's authorization and privacy rules.

## Potentially Relevant Data

```text
Basic Patient Information
Relevant Medical Conditions
Current Medications
Known Allergies
Previous Diagnoses
Previous Prescriptions
Relevant Laboratory Results
Relevant Consultation History
Patient Notes
```

### Important

The doctor should **not automatically receive unrestricted access to the patient's entire medical history** merely because the user has a Doctor role.

Access should depend on:

```text
Doctor Role
+
Active Care Relationship
+
Purpose / Consultation Context
+
Authorized Data Scope
```

---

# 28. Consultation Fields

## Symptoms / Patient Complaint

| Field | Required |
|---|---:|
| Chief Complaint | ✅ |
| Symptoms | ✅ |
| Onset | Recommended |
| Duration | Recommended |
| Severity | Recommended |
| Relevant History | Optional |
| Doctor Notes | ✅ |

## Clinical Notes

The doctor can document relevant findings and observations.

## Diagnosis

| Field | Required |
|---|---:|
| Diagnosis | Conditional / Clinical decision |
| Diagnosis Notes | Recommended |
| Diagnosis Date | System |
| Created By | System |

## Treatment Plan

| Field | Required |
|---|---:|
| Treatment Plan | Conditional |
| Instructions | Recommended |
| Follow-up | Optional |
| Follow-up Date | Conditional |

---

# 29. Prescription

The prescription must be a structured record rather than a chat message.

## Prescription Header

```text
Prescription ID
Patient
Doctor
Consultation
Created At
Status
```

## Prescription Item Fields

| Field | Required | Notes |
|---|---:|---|
| Medicine | ✅ | Prefer a controlled medicine/product catalog. |
| Strength | Conditional | Product-dependent. |
| Dosage | ✅ where applicable | Clinical instruction. |
| Frequency | ✅ where applicable | Example: once/twice daily. |
| Route | Conditional | Oral, topical, etc., if applicable. |
| Duration | ✅ where applicable | Treatment duration. |
| Quantity | Recommended | Based on clinical instruction. |
| Instructions | ✅ where applicable | Special directions. |

### Prescription Lifecycle

```text
DRAFT
 ↓
FINALIZED
 ↓
VISIBLE TO PATIENT
```

Corrections after finalization should be auditable/versioned rather than silently replacing the history.

---

# 30. Consultation Completion

Doctor clicks:

> **Complete Consultation**

System validates required fields.

Then:

```text
Appointment
   ↓
COMPLETED

Consultation
   ↓
FINALIZED

Diagnosis
   ↓
Saved

Prescription
   ↓
Saved

Patient Medical Record
   ↓
Updated
```

Patient receives a notification.

---

# 31. Patient Receives Diagnosis

The patient can open:

> Medical History → Consultations

and see:

```text
Consultation Date
Doctor
Diagnosis
Treatment Plan
Prescription
Follow-up
```

---

# 32. Prescription → Pharmacy Connection

One of the strongest TechCare connections is:

```text
Doctor
  ↓
Consultation
  ↓
Prescription
  ↓
Patient
  ↓
Find Medicines
  ↓
Nearby Pharmacies
  ↓
Medicine Order
```

The doctor does not need to manually forward the prescription to every pharmacy.

---

# 33. Doctor Patient History

The Doctor's Patients section should show patients with whom the doctor has valid care relationships.

## Patient List

```text
Patient Name
Last Consultation
Next Appointment
Relevant Condition
Last Prescription
Status
```

## Patient Details

Potential tabs:

```text
Overview
Medical Context
Consultations
Prescriptions
Laboratory Results
Appointments
Notes
```

Access to each tab must be controlled by the authorization rules.

---

# 34. Doctor Messages / Consultation Communication

If messaging is included in the prototype, separate:

### Consultation Chat
Temporary communication tied to an appointment.

### Clinical Record
Structured, durable medical data.

The doctor should never have to search thousands of chat messages to find the final diagnosis or prescription.

---

# 35. Doctor Reviews

After a completed service, the patient may submit a rating.

Doctor sees:

```text
Average Rating
Total Reviews
Rating Distribution
Recent Reviews
```

Optional service-level dimensions:

- Professionalism
- Communication
- Punctuality
- Overall service quality

### Review Rule

Only eligible completed services should become reviewable.

---

# 36. Complaints

A patient can submit a complaint about a completed or eligible service.

Doctor's complaint tab can show:

```text
Complaint ID
Related Appointment
Category
Created At
Current Status
Patient Description
Admin Response
Doctor Response (if enabled)
```

Recommended status:

```text
SUBMITTED
 ↓
UNDER REVIEW
 ↓
IN PROGRESS
 ↓
RESOLVED
 ↓
CLOSED
```

The doctor cannot delete complaints.

---

# 37. Financial Dashboard

Doctor financials should be transparent.

## Summary

```text
Gross Earnings
Platform Fees
Discounts
Refunds
Net Earnings
Pending Balance
Available Balance
```

## Example

```text
Service Price         500 EGP
Distance Fee           75 EGP
Customer Total        575 EGP
Platform Fee           50 EGP
Doctor Net             525 EGP
```

The actual commercial commission rule can be configured independently.

---

# 38. Transactions

The Transactions page should include:

| Field | Purpose |
|---|---|
| Transaction ID | Unique financial reference. |
| Related Appointment | Link to service. |
| Patient | Counterparty as permitted. |
| Gross Amount | Customer-side amount. |
| Commission | Platform share. |
| Discount | Applied discount. |
| Refund | Returned amount. |
| Net Earnings | Provider-side amount. |
| Status | Pending / completed / refunded etc. |
| Created At | Timestamp. |

---

# 39. Withdrawals

Doctor can request withdrawal of eligible funds.

## Withdrawal Fields

| Field | Required |
|---|---:|
| Amount | ✅ |
| Payout Method | ✅ |
| Account / Destination Reference | ✅ where required |
| Notes | Optional |

## Withdrawal States

```text
REQUESTED
 ↓
UNDER REVIEW / PROCESSING
 ↓
PAID
```

Alternative:

```text
REQUESTED → REJECTED
REQUESTED → CANCELLED (where allowed)
PROCESSING → FAILED → RETRY / REVIEW
```

---

# 40. Financial Audit Trail

Every important financial action should be traceable:

```text
Who
What
When
Amount
Reference
Previous State
New State
Reason
```

The Doctor should not be able to directly edit historical financial transactions.

---

# 41. Notifications

## Request Notifications

```text
New Booking Request
Request Expiring Soon
Request Expired
Booking Accepted
Booking Rejected
Booking Cancelled
```

## Appointment Notifications

```text
Upcoming Appointment
Appointment Rescheduled
Appointment Started
Appointment Completed
```

## Clinical Notifications

```text
Patient-related update where appropriate
Prescription saved / finalized
```

## Financial Notifications

```text
Payment received
Refund processed
Withdrawal submitted
Withdrawal approved
Withdrawal rejected
```

## Quality Notifications

```text
New Review
New Complaint
Complaint Update
```

---

# 42. Doctor Notification Center

Recommended filters:

```text
All
Requests
Appointments
Patients
Financial
Reviews
Complaints
System
```

Each notification should have:

```text
Notification ID
Type
Title
Message
Created At
Read At
Related Entity
Priority
```

---

# 43. Profile Management

Doctor can edit profile information, but high-impact changes may trigger re-verification.

Examples:

- Specialty change
- Professional license number change
- Identity information change
- Major credential change
- Pharmacy/laboratory affiliation where applicable

### Profile Change State

```text
Edit
 ↓
Save
 ↓
Review Required? 
 ├── No → Published
 └── Yes → Pending Reverification
```

---

# 44. Verification Documents Page

The Doctor should see:

```text
Document
Status
Uploaded At
Expiry Date
Reviewer
Reason if rejected
```

Actions:

```text
[ View ]
[ Replace ]
[ Upload New ]
```

For highly sensitive documents, access should remain private and protected.

---

# 45. Account Status

The Doctor account should support multiple states:

```text
REGISTERED
PROFILE_INCOMPLETE
VERIFICATION_PENDING
APPROVED
SUSPENDED
EXPIRED_CREDENTIALS
DEACTIVATED
```

Marketplace visibility depends on the combination of:

```text
Account Active
+
Verification Approved
+
Required Documents Valid
+
Service Active
+
Provider Available
```

---

# 46. Availability Controls

The Doctor can:

- Open slots.
- Block slots.
- Close a working day.
- Add breaks.
- Temporarily disable home visits.
- Adjust coverage area.
- Temporarily pause new requests.

## Pause Requests

```text
Accepting New Requests = ON / OFF
```

When OFF, the doctor should not appear as available for new requests while existing appointments remain visible and managed.

---

# 47. Cancellation Rules

Doctor cancellation should be explicit and auditable.

Possible reasons:

- Emergency/unavailability
- Route/coverage problem
- Patient request
- Operational issue
- Other

The cancellation policy should determine whether the patient receives a refund and whether provider metrics are affected.

Never infer financial outcomes solely from a status string.

---

# 48. No-Show / Visit Failure

If the provider arrives but the patient is unavailable, the appointment should have a defined outcome such as:

```text
PATIENT_NO_SHOW
PROVIDER_NO_SHOW
UNABLE_TO_COMPLETE
RESCHEDULED
```

Each outcome can feed:

- Financial rules
- Complaint process
- Provider metrics
- Patient history

---

# 49. Doctor Performance Metrics

Recommended metrics:

```text
Average Rating
Review Count
Response Rate
Average Response Time
Acceptance Rate
Cancellation Rate
Completion Rate
No-Show Rate
Repeat Patient Rate
```

Metrics should be calculated consistently and should not be manipulated by the client.

---

# 50. Doctor Search Visibility

A doctor becomes visible to patients only if the required marketplace conditions are met.

Possible ranking factors:

```text
Verification
Specialty Match
Distance
Availability
Rating
Review Count
Response Rate
Completion Rate
Price
```

New doctors should not be permanently buried simply because they have fewer reviews.

---

# 51. Security for Doctor Account

Recommended controls:

- Strong password policy.
- Secure password reset.
- Session management.
- Optional/required MFA for sensitive operations according to platform policy.
- Rate limiting.
- Login anomaly detection where implemented.
- Audit logging for sensitive actions.

Sensitive actions may require re-authentication:

```text
Change Password
Change Email
Change Phone
Request Withdrawal
Update Professional Credentials
Access sensitive patient data where step-up is required
```

---

# 52. Medical Data Security

The doctor must never receive medical information simply because the frontend requests it.

The backend must verify:

```text
Authenticated User
      ↓
Role = Doctor
      ↓
Valid Care Relationship
      ↓
Patient Resource
      ↓
Allowed Scope
      ↓
Data Returned
      ↓
Audit Event (where required)
```

---

# 53. Error Scenarios

## Request Expired

```text
Doctor opens request
      ↓
Request already expired
      ↓
Accept button disabled
      ↓
Show “Request Expired”
```

## Slot Taken

```text
Doctor attempts acceptance
      ↓
Slot is no longer available
      ↓
Reject with conflict
      ↓
Request cannot be confirmed
```

## Missing Document

```text
Submit Verification
      ↓
Required document missing
      ↓
Submission blocked
      ↓
Show exact missing document
```

## Rejected Document

```text
Document Rejected
      ↓
Show reason
      ↓
Replace document
      ↓
Resubmit
```

## Payment Failure

The appointment state must not falsely show “paid” merely because the browser returned to the platform.

---

# 54. Doctor Settings

Recommended settings:

```text
Profile Settings
Availability Settings
Service Settings
Distance Pricing
Notification Preferences
Privacy
Security
Payout Settings
Language
Account Status
```

---

# 55. Doctor Full Dashboard Map

```text
Doctor Dashboard
│
├── Overview
│   ├── New Requests
│   ├── Upcoming Appointments
│   ├── Earnings
│   └── Rating
│
├── Requests
│   ├── New
│   ├── Pending
│   ├── Accepted
│   ├── Rejected
│   ├── Expired
│   ├── Cancelled
│   └── Completed
│
├── Appointments
│   ├── Calendar
│   ├── Upcoming
│   ├── Completed
│   └── History
│
├── Patients
│   ├── Active
│   └── History
│
├── Consultations
│   ├── Current
│   └── History
│
├── Prescriptions
│   ├── Draft
│   └── Finalized
│
├── Services
│   ├── Active
│   ├── Add Service
│   └── Pricing
│
├── Availability
│   ├── Working Hours
│   ├── Time Slots
│   └── Blocked Time
│
├── Reviews
│
├── Complaints
│
├── Financials
│   ├── Overview
│   ├── Transactions
│   └── Withdrawals
│
├── Notifications
│
├── Profile
├── Documents
├── Verification
└── Settings
```

---

# 56. Complete Doctor Booking Scenario

## Scenario

A patient with limited mobility wants an orthopedic doctor at home.

### Patient Side

```text
Search Orthopedics
      ↓
Choose Dr. Ahmed
      ↓
See:
Rating = 4.8
Distance = 20 KM
Base Price = 500 EGP
Distance Fee = 75 EGP
Total = 575 EGP
      ↓
Choose 07:00 PM Slot
      ↓
Send Request
```

### Doctor Side

```text
Notification
      ↓
Open Request
      ↓
Review Patient + Service + Time + Location + Price
      ↓
Accept
```

### System

```text
PENDING → ACCEPTED → CONFIRMED
```

### Visit

```text
Doctor Arrives
      ↓
Start Consultation
      ↓
Review Relevant Medical Context
      ↓
Clinical Assessment
      ↓
Diagnosis
      ↓
Treatment Plan
      ↓
Prescription
      ↓
Complete Consultation
```

### After Visit

```text
Appointment → COMPLETED
Consultation → FINALIZED
Prescription → Patient Record
Transaction → Settled / Pending Settlement
Patient → Can Review Service
```

---

# 57. Doctor ↔ Patient Data Relationship

```text
Doctor
   │
   │ provides
   ▼
Appointment / Service Request
   │
   ▼
Consultation
   │
   ├── Diagnosis
   ├── Treatment Plan
   └── Prescription
              │
              ▼
           Patient
              │
              ├── Medical History
              ├── Prescriptions
              └── Pharmacy Search
```

---

# 58. Doctor ↔ Pharmacy Connection

```text
Doctor
   ↓
Prescription
   ↓
Patient
   ↓
Find Medicines
   ↓
Pharmacy Inventory
   ↓
Pharmacy Order
```

The Doctor's prescription should contain enough structured information to support safe medicine discovery without allowing the pharmacy marketplace to silently rewrite the clinical instruction.

---

# 59. Doctor ↔ Laboratory Connection

A future/extended flow can be:

```text
Doctor
   ↓
Request Laboratory Test
   ↓
Patient
   ↓
Choose / Confirm Laboratory
   ↓
Home Sample Collection
   ↓
Result
   ↓
Patient Medical Record
   ↓
Doctor Can View Relevant Result
```

The prototype should keep this relationship structured even if the first version does not implement every advanced integration.

---

# 60. Doctor ↔ Blood Donation Boundary

The doctor may become relevant to a patient care journey involving blood, but the blood-donation workflow remains a dedicated module.

```text
Patient / Authorized Requester
       ↓
Blood Request
       ↓
Compatible Donor Matching
       ↓
Authorized Blood Service
```

The doctor role should not independently determine final donor eligibility or transfusion suitability through the marketplace application.

---

# 61. Doctor Acceptance Criteria

The Doctor module is considered functionally complete when:

```text
✅ Doctor can register.
✅ Contact information can be verified.
✅ Professional profile can be completed.
✅ Required documents can be uploaded securely.
✅ Verification state is visible.
✅ Admin can approve/reject/suspend.
✅ Approved doctor can configure services.
✅ Doctor can set prices.
✅ Doctor can configure distance pricing.
✅ Doctor can configure working hours.
✅ Doctor can create available slots.
✅ Patient can discover an eligible doctor.
✅ Patient can send a booking request.
✅ Doctor receives notification.
✅ Request expires when response window ends.
✅ Doctor cannot accept expired request.
✅ Double booking is prevented.
✅ Doctor can accept/reject valid requests.
✅ Confirmed appointment appears in calendar.
✅ Doctor can start consultation.
✅ Doctor can view authorized patient context.
✅ Doctor can create structured diagnosis/treatment/prescription.
✅ Patient can see finalized records.
✅ Service can be completed.
✅ Financial transaction is recorded.
✅ Doctor can see earnings/transactions.
✅ Ratings are enabled after eligible completion.
✅ Complaints are trackable.
✅ Sensitive actions are authorized and auditable.
```

---

# 62. Doctor Data Model — High Level

```text
User
  │
  └── DoctorProfile
        ├── ProfessionalCredentials
        ├── VerificationDocuments
        ├── Specialties
        ├── Services
        ├── AvailabilityRules
        ├── TimeSlots
        ├── Location / Coverage
        ├── Appointments
        ├── Consultations
        │      ├── Diagnoses
        │      ├── TreatmentPlans
        │      └── Prescriptions
        ├── Reviews
        ├── Complaints
        ├── FinancialTransactions
        ├── Withdrawals
        └── Notifications
```

---

# 63. Minimum Necessary Data Principle

The Doctor workflow should not collect every possible field simply because the system can store it.

For each field ask:

```text
Why is this data needed?
      ↓
Which workflow needs it?
      ↓
Who can access it?
      ↓
How long is it retained?
      ↓
What happens if the doctor does not provide it?
```

Fields required only for a specific service should be **conditional**, not globally required.

---

# 64. Security Checklist

Before considering the Doctor module production-ready:

- [ ] Provider role cannot be self-escalated from another role.
- [ ] Verification status is server-controlled.
- [ ] Professional documents are private.
- [ ] Patient medical records cannot be fetched by arbitrary patient ID.
- [ ] Object-level authorization is tested.
- [ ] Expired booking requests cannot be accepted.
- [ ] Double-booking tests pass.
- [ ] Price is server-calculated.
- [ ] Historical prices are snapshotted.
- [ ] Financial records cannot be edited by the client.
- [ ] Withdrawal actions are authenticated and authorized.
- [ ] Sensitive actions are auditable.
- [ ] Notifications do not expose unnecessary medical information.

---

# 65. Recommended UX Rules

## Keep the Doctor Dashboard Action-Oriented

The first screen should prioritize:

```text
What needs my attention now?
```

not:

```text
Every piece of data the system has.
```

## Make Status Obvious

Use consistent status labels:

```text
Pending
Accepted
Rejected
Expired
Confirmed
In Progress
Completed
Cancelled
```

## Never Hide the Financial Breakdown

The Doctor and Patient should be able to understand:

```text
Base Price
+
Distance Fee
-
Discount
=
Final Total
```

---

# 66. Recommended Doctor Experience Summary

The entire Doctor experience can be summarized as:

```text
JOIN
 ↓
VERIFY
 ↓
CONFIGURE
 ↓
DISCOVERABLE
 ↓
RECEIVE REQUESTS
 ↓
RESPOND
 ↓
VISIT PATIENT
 ↓
CONSULT
 ↓
DOCUMENT
 ↓
COMPLETE
 ↓
GET PAID
 ↓
GET RATED
 ↓
IMPROVE
```

---

# 67. Prototype Boundary

This workflow belongs to the **first TechCare Web Prototype**.

### Included

- Doctor registration
- Doctor verification
- Professional profile
- Services
- Pricing
- Distance pricing
- Availability
- Time slots
- Booking requests
- Request expiry
- Patient context during valid consultation
- Consultation
- Diagnosis
- Treatment plan
- Prescription
- Medical history connection
- Ratings
- Complaints
- Notifications
- Financial transactions
- Withdrawals
- Doctor dashboard

### Future Features

The following remain outside the first prototype unless explicitly added to the next scope:

- AI medical assistant
- AI symptom analysis
- AI triage
- Emergency dispatch integration
- Telemedicine/video consultation
- Hospital-system integration
- Advanced predictive analytics
- Native mobile application

---

# 68. Final Doctor Workflow

```text
                    ┌────────────────────┐
                    │   DOCTOR ACCOUNT   │
                    └─────────┬──────────┘
                              │
                         Registration
                              │
                              ▼
                    Personal Information
                              │
                              ▼
                   Professional Information
                              │
                              ▼
                    Documents & Credentials
                              │
                              ▼
                        ADMIN REVIEW
                              │
                ┌─────────────┴─────────────┐
                │                           │
             REJECT                      APPROVE
                │                           │
          Resubmission                      ▼
                                  Services + Prices
                                          │
                                  Location / Coverage
                                          │
                                  Availability / Slots
                                          │
                                          ▼
                                   PATIENT DISCOVERY
                                          │
                                          ▼
                                    BOOKING REQUEST
                                          │
                             ┌────────────┼────────────┐
                             │            │            │
                           ACCEPT       REJECT       EXPIRE
                             │
                             ▼
                          CONFIRMED
                             │
                             ▼
                       HOME CONSULTATION
                             │
                             ▼
                    AUTHORIZED PATIENT DATA
                             │
                             ▼
                       CONSULTATION RECORD
                             │
                  ┌──────────┼──────────┐
                  │          │          │
              DIAGNOSIS   TREATMENT  PRESCRIPTION
                  │          │          │
                  └──────────┼──────────┘
                             │
                             ▼
                         COMPLETED
                             │
                  ┌──────────┼───────────┐
                  │          │           │
              FINANCIALS   RATING    COMPLAINT
                  │
                  ▼
              WITHDRAWAL
```

---

# 69. Final Product Principle

The Doctor module should not be treated as a collection of pages.

It is a complete provider lifecycle:

> **Verify the professional → publish trusted services → receive a controlled request → deliver care → document the clinical outcome → complete the transaction → maintain trust.**

The quality bar for TechCare is not that the Doctor can “use the dashboard”. The quality bar is that the **entire doctor journey is consistent, permission-safe, traceable, and connected to the Patient, Pharmacy, Laboratory, and platform workflows.**
