# TechCare — Blood Donor Complete Workflow

## 1. Document Purpose

This document defines the complete Blood Donor workflow for the TechCare healthcare platform.

The goal is to transform the Blood Donation feature from a simple “find a donor” idea into a complete, implementable product workflow covering:

- Donor onboarding
- Authentication and role authorization
- Donor medical/profile information
- Blood type and compatibility matching
- Location and service-area handling
- Availability / temporary unavailability
- Patient blood requests
- Matching and notification
- Donor response flow
- Donation coordination
- Donation appointment / handoff
- Donation status tracking
- Donation history
- Points and rewards
- Notifications
- Abuse prevention
- Privacy and security
- Admin moderation and verification
- APIs
- Database entities and relationships
- State machines
- Frontend pages
- Validation
- Concurrency protection
- Audit logging
- Testing strategy
- MVP scope and future enhancements

> Important product boundary: TechCare coordinates requests between patients and eligible donors. It is not a blood marketplace and should not sell blood, buy blood units, or take a commission from blood donations. Actual donor eligibility, medical screening, blood collection, testing, storage, compatibility confirmation, and transfusion decisions remain under the responsibility of the authorized blood bank / healthcare facility and qualified medical personnel.

---

# 2. Feature Scope

The Blood Donation module contains two main sides:

### Patient / Requester Side

A patient or authorized caregiver can:

1. Create a blood request.
2. Select the required blood type.
3. Enter required units / quantity as requested by the authorized facility.
4. Enter urgency level.
5. Select hospital / blood bank / destination information.
6. Provide the required location where coordination is needed.
7. Track request status.
8. Receive donor response notifications.
9. See how many potential donors were contacted.
10. Cancel a request when the need has been fulfilled or cancelled.
11. See confirmed donor commitments.
12. Rate / report the experience where applicable.

### Donor Side

A donor can:

1. Register as a blood donor.
2. Complete a donor profile.
3. Add blood group information.
4. Add donation preferences.
5. Set service area / preferred donation locations.
6. Set availability.
7. Receive eligible blood requests.
8. See summarized request information.
9. Accept / decline a request.
10. Temporarily pause donor availability.
11. Confirm attendance / donation handoff.
12. View donation history.
13. Earn non-cash points according to platform rules.
14. Receive notifications and reminders.
15. Report suspicious or inappropriate requests.

### Admin Side

Admin can:

1. Monitor donor registrations.
2. Review reports.
3. Verify or suspend donor accounts when required.
4. Monitor blood requests.
5. Cancel fraudulent / abusive requests.
6. Manage reward policies.
7. Review donation records.
8. View audit logs.
9. Manage system configuration.

---

# 3. Core Business Principles

## 3.1 Blood is not sold

The platform must never represent blood units as products for sale.

There is no:

- Blood unit price
- Donor payment
- Blood marketplace commission
- “Buy blood” checkout

The platform coordinates voluntary donation and healthcare-facility workflows.

## 3.2 Donation eligibility is not decided only by the app

The system may store donor-provided health information and configurable eligibility metadata, but the app should not promise that a donor is medically eligible.

Examples of donor status shown by the application:

- Available for matching
- Temporarily unavailable
- Recently donated — restricted by configured policy
- Verification required
- Account suspended

Final eligibility must be confirmed by the authorized blood collection service.

## 3.3 Compatibility is a matching filter, not a transfusion decision

The platform can use blood group data to narrow donor candidates, but final medical compatibility and transfusion decisions must be verified by qualified healthcare professionals / blood-bank systems.

## 3.4 Location is privacy-sensitive

The platform should not expose a donor’s exact home location to unrelated patients.

Use:

- Approximate area
- City / district
- Preferred donation facility
- Distance band

Exact location may be shared only when necessary and authorized for the coordination step.

## 3.5 No direct private medical arrangements outside the platform where avoidable

The platform should encourage coordination through the authorized facility instead of exposing personal information such as:

- Personal phone number
- Exact home address
- National ID number
- Full medical history

unless there is a legitimate and authorized reason.

---

# 4. User Roles

## 4.1 Donor

A registered user who chooses to participate in voluntary blood donation coordination.

## 4.2 Patient / Requester

A patient or authorized person creating a blood request.

## 4.3 Healthcare Facility / Blood Bank

An authorized facility responsible for screening, collection, and medical handling.

The first prototype can represent this through an organization/facility field rather than building a completely separate blood-bank application if scope is limited.

## 4.4 Admin

Platform administrator responsible for moderation, configuration, verification, and monitoring.

---

# 5. Donor Registration Workflow

## Step 1 — User chooses “Become a Blood Donor”

From the authenticated account, the user sees:

`Dashboard → Blood Donation → Become a Donor`

The platform shows an introduction explaining:

- Donation is voluntary.
- Medical eligibility is determined by qualified staff.
- TechCare only coordinates requests.
- The donor can pause availability at any time.

## Step 2 — Donor profile form

Suggested fields:

### Personal

- UserId
- FullName
- DateOfBirth
- Gender (only if actually required by product/business policy)
- City
- District
- Preferred donation area
- Preferred contact method

### Blood Information

- ABO blood group
- Rh factor
- BloodTypeDisplay
- Blood type source
- Blood type verified flag
- Blood type verification date

### Donation Preferences

- AvailableForDonation
- PreferredDonationFacilities
- MaximumTravelDistanceKm
- PreferredDays
- PreferredTimeWindows
- AllowUrgentRequests
- NotificationPreference

### Additional donor information

- LastKnownDonationDate (optional and subject to verified source)
- DonorNotes (limited, privacy-safe)
- EmergencyContact (optional)

## Step 3 — Blood Type Verification

The platform should distinguish between:

`SelfReported`

and

`Verified`

Example:

```text
Blood Type: O+
Source: Self Reported
Verified: No
```

Later:

```text
Blood Type: O+
Source: Authorized Facility
Verified: Yes
VerifiedAt: ...
```

A donor should not become a high-confidence matching candidate based solely on an unverified blood group if product policy requires verification.

## Step 4 — Consent

The donor must acknowledge:

- Voluntary donation
- Data usage for matching
- Notification consent
- Location/privacy rules
- Medical screening is performed by authorized staff

Store consent timestamp and consent version.

## Step 5 — Donor activation

Possible statuses:

```text
DRAFT
PENDING_VERIFICATION
ACTIVE
PAUSED
SUSPENDED
DEACTIVATED
```

For an MVP, the donor may become `ACTIVE` after completing the required fields, while sensitive verification can remain separate.

---

# 6. Donor Dashboard Workflow

Main dashboard sections:

```text
Blood Donor Dashboard
│
├── Availability
├── Blood Profile
├── Nearby Requests
├── My Responses
├── Upcoming Donation Coordination
├── Donation History
├── Points & Rewards
├── Notifications
├── Privacy Settings
└── Help / Report Issue
```

## Dashboard summary cards

Examples:

- Current donor status
- Blood type
- Availability status
- Pending requests
- Accepted requests
- Donation count
- Points balance
- Last recorded donation
- Verification status

---

# 7. Donor Availability Workflow

The donor controls whether they can be contacted.

## Statuses

### AVAILABLE

Donor can appear in matching results.

### PAUSED

Donor temporarily does not want new requests.

### RESTRICTED

The system prevents matching due to a configured rule or verified status.

### SUSPENDED

Admin has blocked donor participation.

### DEACTIVATED

Donor has stopped participation.

## Availability actions

```text
Turn On Availability
Turn Off Availability
Pause Until Date
Pause Indefinitely
Resume Availability
```

Example:

```text
Available for blood requests: ON
Preferred area: Mansoura
Maximum distance: 15 km
Urgent requests: ON
```

## Important rule

A donor may receive a notification for a request only when their account is eligible for matching at the time the matching job is executed.

---

# 8. Donor Location Workflow

Location can be configured at multiple levels.

## Level 1 — Broad location

- Governorate
- City
- District

## Level 2 — Preferred donation area

Examples:

- Mansoura
- Talkha
- Nearby hospital area

## Level 3 — Facility preference

The donor can select preferred authorized facilities.

## Level 4 — Exact location

Exact location should not be publicly exposed.

For distance matching the backend can calculate:

```text
DistanceKm = CalculateDistance(DonorLocation, RequestLocation)
```

The UI may show:

```text
About 4.8 km away
```

instead of exposing the donor's exact home coordinates.

---

# 9. Blood Request Creation Workflow

The request can be created by the patient or an authorized caregiver.

Navigation:

`Dashboard → Blood Donation → Request Blood`

## Step 1 — Request type

```text
Emergency / Urgent
Routine / Scheduled
```

Use configurable labels rather than promising a medical emergency response.

## Step 2 — Required blood group

Possible values:

```text
A+
A-
B+
B-
AB+
AB-
O+
O-
Unknown / To be confirmed
```

## Step 3 — Quantity

Suggested field:

```text
RequiredUnits
```

The unit must be defined by the authorized facility so the application does not create medically ambiguous quantities.

## Step 4 — Destination / facility

Fields:

- FacilityId
- FacilityName
- City
- District
- ContactReference
- RequiredByDateTime

## Step 5 — Request location

Prefer the healthcare facility / collection location.

Do not require a patient home address when it is not necessary.

## Step 6 — Additional information

Safe limited fields:

- Request reference number
- Hospital department
- Patient record reference (if applicable)
- Notes for donors

Do not expose unnecessary diagnosis or sensitive medical information.

## Step 7 — Patient confirmation

Before submission show:

```text
Blood Type: O+
Required Units: 2
Facility: Authorized Facility
Needed By: ...
Area: ...
Urgency: Urgent
```

Then:

`Submit Blood Request`

---

# 10. Blood Request Statuses

Recommended lifecycle:

```text
DRAFT
↓
PENDING
↓
MATCHING
↓
DONORS_CONTACTED
↓
PARTIALLY_FULFILLED
↓
FULFILLED
```

Alternative terminal states:

```text
CANCELLED
EXPIRED
REJECTED
SUSPENDED
```

## Meaning

### DRAFT

Patient is still editing.

### PENDING

Request submitted and waiting for validation.

### MATCHING

System is finding suitable donor candidates.

### DONORS_CONTACTED

Notifications have been sent.

### PARTIALLY_FULFILLED

Enough donors have not yet completed the required coordination.

### FULFILLED

The healthcare facility has confirmed the request requirement has been satisfied.

### CANCELLED

Requester cancelled the request.

### EXPIRED

The request passed its valid window.

### SUSPENDED

Admin or safety controls prevented continued activity.

---

# 11. Matching Engine Workflow

The matching service should not simply find everyone with the same blood type.

It should apply layered filtering.

## Candidate filters

### Filter 1 — Account status

Must be:

```text
ACTIVE
```

and not suspended.

### Filter 2 — Donor availability

```text
AvailableForDonation = true
```

### Filter 3 — Blood group

Use configured compatibility rules.

The platform should support a rule table rather than hard-coding all logic directly inside controllers.

Example concept:

```text
BloodCompatibilityRule
----------------------
RequesterBloodType
CompatibleDonorBloodType
IsAllowedForInitialMatch
RequiresClinicalConfirmation
```

### Filter 4 — Location

Check:

```text
DistanceKm <= Donor.MaximumTravelDistanceKm
```

### Filter 5 — Facility preference

Prefer donors whose selected facility matches the request facility.

### Filter 6 — Cooldown / restriction

Respect configured restrictions based on the donor's verified donation record and healthcare policy.

### Filter 7 — Notification availability

Do not send repeated notifications beyond configured limits.

### Filter 8 — Anti-abuse rules

Exclude:

- Blocked users
- Reported accounts under review
- Suspended donors
- Requester/donor blocked relationships
- Already committed donor for the same request

---

# 12. Matching Priority

A practical ranking model:

```text
Score =
    BloodCompatibilityScore
  + DistanceScore
  + FacilityPreferenceScore
  + AvailabilityScore
  + UrgentRequestResponsivenessScore
```

The exact numerical weights should remain configurable.

The system should avoid using sensitive characteristics as ranking factors unless they are medically necessary and authorized.

## Example ranking

```text
Donor A
Blood match: Yes
Distance: 2.4 km
Preferred facility: Yes
Available: Yes

Donor B
Blood match: Yes
Distance: 9.7 km
Preferred facility: No
Available: Yes
```

Donor A should generally appear first.

---

# 13. Donor Request Notification

When a compatible donor is selected, send notification through configured channels.

Possible channels:

- In-app
- Push notification
- Email (optional)
- SMS (optional future integration)

## Example notification

```text
Blood Donation Request

A blood donation request matching your registered blood type is available near your preferred area.

Facility: [Authorized Facility]
Area: [District]
Needed by: [Date/Time]
Urgency: [Urgent/Routine]

Your exact address is not shared with the requester.

View Request
```

## Do not reveal unnecessarily

The notification should not automatically include:

- Patient full medical record
- Diagnosis
- National ID
- Exact home address
- Private contact details

---

# 14. Donor Request Details Screen

When the donor opens the request:

```text
Request #BD-000123

Blood Type: O+
Units Needed: 2
Facility: Authorized Facility
Area: Mansoura
Needed By: Today, 8:00 PM
Status: Open

[Accept to Donate]
[Decline]
[Report]
```

A short safety note can appear:

> Acceptance confirms your willingness to attend screening/coordination. Final donor eligibility is determined by qualified staff.

---

# 15. Donor Accept Workflow

When donor presses `Accept`:

1. Validate request is still open.
2. Validate donor is still active.
3. Re-check availability.
4. Re-check matching eligibility.
5. Prevent duplicate response.
6. Create donor response record.
7. Reserve a coordination slot if applicable.
8. Notify requester.
9. Record audit event.

Possible response status:

```text
ACCEPTED
```

The donor is not automatically marked as having donated.

Acceptance is only a commitment to participate.

---

# 16. Donor Decline Workflow

When donor selects `Decline`:

Optional reason values:

- Not available now
- Too far
- Cannot attend
- Temporarily unavailable
- I do not wish to donate now
- Other

Avoid forcing the donor to provide sensitive health information.

Create:

```text
DonorResponse.Status = DECLINED
```

The system should not penalize normal declines.

---

# 17. Donor Response Statuses

```text
PENDING
ACCEPTED
DECLINED
WITHDRAWN
EXPIRED
CANCELLED
COMPLETED
NO_SHOW
```

## PENDING

Notification sent / response window open.

## ACCEPTED

Donor accepted the coordination.

## DECLINED

Donor rejected the request.

## WITHDRAWN

Donor accepted but later withdrew before the donation event.

## COMPLETED

Authorized facility confirmed the donation event.

## NO_SHOW

Authorized workflow marked the donor as not attending.

---

# 18. Coordination After Acceptance

Once donor accepts, show:

```text
Donation Coordination

Facility: [Facility]
Location: [Area / Facility Address]
Date: [Date]
Time: [Slot]
Reference: [Facility Reference]

Status: Awaiting Facility Confirmation
```

The preferred flow is:

```text
Patient Request
      ↓
Donor Match
      ↓
Donor Accepts
      ↓
Facility Confirmation
      ↓
Donation Appointment / Visit
      ↓
Medical Screening
      ↓
Donation
      ↓
Facility Confirms Outcome
      ↓
TechCare Updates History
```

---

# 19. Facility Confirmation Workflow

An authorized facility can confirm:

- Donor appointment received
- Donor arrived
- Screening completed
- Donation completed
- Donation could not proceed

Possible facility statuses:

```text
COORDINATION_CONFIRMED
ARRIVED
SCREENING_IN_PROGRESS
ELIGIBLE
NOT_ELIGIBLE
DONATION_COMPLETED
DONATION_NOT_COMPLETED
```

The platform should not expose unnecessary medical screening details.

For example, instead of showing:

> “Your hemoglobin result was X”

the platform can store only:

```text
Donation outcome: Not completed
Reason category: Medical screening result
```

unless the facility has a legitimate reason to send further information through a protected clinical channel.

---

# 20. Donation Completion Workflow

A donation should be marked `COMPLETED` only from an authorized source.

Preferred sources:

1. Authorized facility action.
2. Verified facility API integration.
3. Admin override with reason and audit log.

A patient should not be able to mark a donor as completed by themselves.

A donor should not be able to self-confirm a medical donation as completed when the reward or official history depends on verification.

---

# 21. Donation Record

Create a separate historical entity:

```text
DonationRecord
```

Suggested fields:

```text
Id
DonorId
BloodRequestId
FacilityId
DonationReference
DonationDate
Status
VerifiedBy
VerifiedAt
SourceType
Notes
CreatedAt
UpdatedAt
```

Sensitive medical results should not be duplicated unnecessarily.

---

# 22. Donation History

Donor page:

`Blood Donation → Donation History`

Example:

```text
Donation #1
Date: 2026-09-12
Facility: [Facility]
Status: Verified Completed

Donation #2
Date: 2026-10-02
Facility: [Facility]
Status: Verified Completed
```

For each record:

- Date
- Facility
- Status
- Reference number
- Points earned

Do not expose other patients' information.

---

# 23. Points & Rewards Workflow

The platform can reward donors using non-cash points.

Example:

```text
Donation completed → +100 points
Verified successful contribution → configured bonus
Special campaign → optional bonus
```

Points should be granted only after a verified completion event.

## Points Ledger

Never store only a mutable balance.

Use a ledger:

```text
RewardTransaction
-----------------
Id
UserId
Type
Points
ReferenceType
ReferenceId
Description
CreatedAt
```

Then calculate or maintain:

```text
CurrentBalance = Sum(Credits) - Sum(Debits)
```

A cached balance can be used for performance, but every change must have a ledger entry.

## Rewards examples

- Badges
- Donor levels
- Platform recognition
- Eligible discounts from approved non-medical partners in future

Cash payments for blood itself are outside this module's product model.

---

# 24. Reward State Machine

```text
DONATION_VERIFIED
       ↓
REWARD_PENDING
       ↓
REWARD_GRANTED
       ↓
BALANCE_UPDATED
```

If verification is reversed:

```text
REWARD_GRANTED
       ↓
REWARD_REVERSED
```

All reversals require an audit event.

---

# 25. Patient Blood Request Dashboard

Patient navigation:

```text
Dashboard
  ↓
Blood Donation
  ↓
My Blood Requests
```

Each request card:

```text
Request #BD-000123
Blood Type: O+
Units Needed: 2
Status: 3 donors responding
Facility: [Facility]
Needed By: [Date]

[View]
```

## Request details

Display:

- Request status
- Created date
- Required blood type
- Quantity
- Facility
- Number contacted
- Accepted responses
- Remaining need
- Time/expiry information
- Cancellation option

---

# 26. Request Fulfillment Logic

Suppose:

```text
RequiredUnits = 3
ConfirmedUnits = 0
```

Donor A accepts:

```text
ConfirmedUnits = 1
```

Donor B accepts:

```text
ConfirmedUnits = 2
```

Donor C accepts:

```text
ConfirmedUnits = 3
```

The request may move to:

```text
PARTIALLY_FULFILLED
```

then:

```text
FULFILLED
```

However, the exact meaning of `ConfirmedUnits` must match the operational model. In many implementations, a donor commitment should count as `accepted donor commitments`, while fulfillment should only be completed by the authorized facility.

Recommended separation:

```text
AcceptedDonorCount
VerifiedDonationCount
```

This avoids claiming that an accepted donor has already supplied blood.

---

# 27. Preventing Over-Commitment

A common error is allowing unlimited donors to accept an already fulfilled request.

Example:

```text
Needed = 2
Verified donations = 2
```

The request should stop sending new matching notifications.

Existing accepted donors should see:

```text
This request is already covered by confirmed donors.

You can remain available for future requests.
```

However, do not automatically penalize or mark the donor as unsuccessful solely because the requester no longer needs them.

Possible response status:

```text
CANCELLED_BY_REQUEST_FULFILLMENT
```

---

# 28. Concurrency Protection

The backend must protect the request and donor-response records from race conditions.

Example:

Two donors accept simultaneously while one final slot remains.

Use a transactional operation and appropriate row locking / optimistic concurrency.

Conceptual sequence:

```text
BEGIN TRANSACTION

Load BloodRequest
Check request still accepts donors
Check donor response doesn't already exist
Create response
Update counters/state

COMMIT
```

For ASP.NET Core / EF Core, use an explicit transaction and concurrency strategy where appropriate.

The frontend is never the source of truth for request capacity.

---

# 29. Request Expiration

Each request should have a server-generated expiration timestamp when a time window is required.

Fields:

```text
CreatedAt
ResponseDeadline
NeededBy
ExpiredAt
```

At expiration:

```text
PENDING / MATCHING / DONORS_CONTACTED
            ↓
          EXPIRED
```

The job that expires requests must be idempotent.

Running it twice must not corrupt the state.

---

# 30. Notification Frequency Control

A donor should not receive the same request repeatedly.

Create a notification deduplication concept:

```text
BloodRequestId + DonorId + NotificationType
```

Possible rules:

- One initial notification per request
- One reminder if configured
- No reminder after donor response
- No notifications after request cancellation
- No notifications after donor becomes unavailable

---

# 31. Emergency / Urgent Request Handling

For urgent requests the system may:

1. Increase matching priority.
2. Notify a larger nearby candidate pool.
3. Repeat reminders under configured limits.
4. Surface the request more prominently.

But TechCare should not claim to replace emergency services.

The UI should direct users to emergency medical services when a situation is life-threatening and should follow the local emergency/healthcare process.

---

# 32. Donor Privacy Model

## Data visible to requester

Recommended minimum:

- Donor first name or display name
- Blood type category when needed
- Approximate area
- General availability
- Response status

## Data hidden by default

- Exact home address
- Exact GPS coordinates
- National ID
- Email address
- Personal phone number
- Full health history
- Sensitive medical notes

## Data visible to authorized facility

Only what is required for the authorized donation workflow.

## Data visible to admin

According to administrative permissions and audit requirements.

---

# 33. Blocking & Safety

A patient can report:

- Suspicious donor behavior
- Inappropriate communication
- Fake donation claim
- Harassment
- Fraud

A donor can report:

- Fake blood request
- Repeated spam requests
- Harassment
- Misleading request details

A report creates:

```text
SafetyReport
```

Suggested statuses:

```text
OPEN
UNDER_REVIEW
RESOLVED
DISMISSED
ESCALATED
```

Admin actions may include:

- Warning
- Temporary restriction
- Suspension
- Permanent deactivation
- Request cancellation

---

# 34. Donor Verification Workflow

Depending on the product policy, donor verification can be lightweight or facility-based.

Possible verification levels:

```text
UNVERIFIED
IDENTITY_VERIFIED
BLOOD_TYPE_VERIFIED
FACILITY_VERIFIED
```

Recommended MVP:

- Basic account verification
- Blood type source captured
- Facility verification for official donation completion

The system should not collect unnecessary identity documents solely because it can.

---

# 35. Donor Profile Screen

Example UI:

```text
----------------------------------------
Blood Donor Profile
----------------------------------------
Name: Mostafa
Blood Type: O+
Verification: Verified
Area: Mansoura
Max Distance: 15 km
Urgent Requests: ON
Availability: ON

Donation History: 3
Points: 320

[Edit Profile]
[Pause Availability]
[Privacy Settings]
----------------------------------------
```

---

# 36. Donor Settings

Settings:

### Availability

- Available for new requests
- Pause until
- Maximum distance

### Notifications

- New blood requests
- Reminders
- Request updates
- Donation confirmations
- Reward notifications

### Privacy

- Display name
- Contact sharing preferences
- Location visibility

### Account

- Change password
- Logout
- Deactivate donor participation

---

# 37. Notification Types

Recommended notification types:

```text
BLOOD_REQUEST_MATCH
BLOOD_REQUEST_REMINDER
DONOR_RESPONSE_ACCEPTED
DONOR_RESPONSE_DECLINED
REQUEST_CANCELLED
REQUEST_FULFILLED
FACILITY_CONFIRMATION
DONATION_REMINDER
DONATION_COMPLETED
DONATION_NOT_COMPLETED
REWARD_GRANTED
REWARD_REVERSED
ACCOUNT_SUSPENDED
VERIFICATION_UPDATED
SAFETY_REPORT_UPDATE
```

---

# 38. Frontend Pages — Donor

Recommended routes:

```text
/blood-donation
/blood-donation/become-donor
/blood-donation/profile
/blood-donation/availability
/blood-donation/requests
/blood-donation/requests/{id}
/blood-donation/responses
/blood-donation/upcoming
/blood-donation/history
/blood-donation/rewards
/blood-donation/notifications
/blood-donation/settings
/blood-donation/report
```

---

# 39. Frontend Pages — Patient Requester

```text
/blood-requests
/blood-requests/create
/blood-requests/{id}
/blood-requests/{id}/donors
/blood-requests/{id}/timeline
```

The donor list should not expose private donor information unnecessarily.

---

# 40. Facility Pages — Optional MVP Extension

If the team later adds a dedicated facility dashboard:

```text
/facility/blood-requests
/facility/blood-requests/{id}
/facility/donors
/facility/appointments
/facility/donations
/facility/reports
```

---

# 41. Database Entities

Suggested entities:

```text
User
Role
DonorProfile
DonorAvailability
DonorBloodType
DonorPreference
DonationFacility
BloodRequest
BloodRequestResponse
DonationAppointment
DonationRecord
RewardTransaction
Notification
SafetyReport
ConsentRecord
AuditLog
```

---

# 42. DonorProfile Entity

Suggested fields:

```text
Id
UserId
DisplayName
DateOfBirth
CityId
DistrictId
MaxTravelDistanceKm
Status
IsAvailable
AllowUrgentRequests
VerificationStatus
CreatedAt
UpdatedAt
```

Avoid duplicating data already owned by the shared User/Profile system.

---

# 43. DonorBloodType Entity

Suggested fields:

```text
Id
DonorProfileId
BloodType
VerificationStatus
SourceType
VerifiedBy
VerifiedAt
CreatedAt
UpdatedAt
```

Example source values:

```text
SELF_REPORTED
AUTHORIZED_FACILITY
IMPORT
ADMIN_VERIFIED
```

---

# 44. BloodRequest Entity

Suggested fields:

```text
Id
RequesterUserId
PatientUserId
FacilityId
BloodTypeRequested
RequiredUnits
AcceptedDonorCount
VerifiedDonationCount
UrgencyLevel
Status
RequestLocationId
CreatedAt
ResponseDeadline
NeededBy
CancelledAt
FulfilledAt
CreatedBy
UpdatedAt
```

The requester and patient should be separate references when a caregiver can submit the request.

---

# 45. BloodRequestResponse Entity

Suggested fields:

```text
Id
BloodRequestId
DonorId
Status
MatchScoreSnapshot
DistanceKmSnapshot
NotificationSentAt
ViewedAt
RespondedAt
AcceptedAt
DeclinedAt
DeclineReason
CreatedAt
UpdatedAt
```

The snapshot fields are useful because the donor's profile may change later.

---

# 46. DonationAppointment Entity

Suggested fields:

```text
Id
BloodRequestId
DonorId
FacilityId
ScheduledDateTime
Status
ReferenceNumber
ConfirmedAt
ArrivedAt
CompletedAt
CancelledAt
CancellationReason
CreatedAt
UpdatedAt
```

---

# 47. DonationRecord Entity

Suggested fields:

```text
Id
DonorId
BloodRequestId
FacilityId
DonationAppointmentId
DonationReference
DonationDate
Status
VerificationSource
VerifiedBy
VerifiedAt
CreatedAt
UpdatedAt
```

Keep clinical test data outside this entity unless there is a clear clinical-data requirement and proper authorization.

---

# 48. RewardTransaction Entity

Suggested fields:

```text
Id
UserId
TransactionType
Points
ReferenceType
ReferenceId
Description
CreatedAt
CreatedBy
```

Examples:

```text
DONATION_REWARD +100
CAMPAIGN_BONUS +50
REWARD_REVERSAL -100
```

---

# 49. SafetyReport Entity

Suggested fields:

```text
Id
ReporterUserId
TargetUserId
BloodRequestId
DonationResponseId
Category
Description
Status
AdminNotes
ResolvedBy
ResolvedAt
CreatedAt
UpdatedAt
```

---

# 50. Entity Relationships

```text
User
 │
 ├── DonorProfile
 │     ├── DonorBloodType
 │     ├── DonorPreference
 │     ├── DonorAvailability
 │     ├── BloodRequestResponse
 │     └── DonationRecord
 │
 └── BloodRequest (as requester/patient)
        ├── BloodRequestResponse
        ├── DonationAppointment
        └── DonationRecord

DonationFacility
 ├── BloodRequest
 ├── DonationAppointment
 └── DonationRecord

User
 └── RewardTransaction

User
 └── Notification

User
 └── SafetyReport
```

---

# 51. API Design

Base route example:

```text
/api/v1/blood-donation
```

## Donor profile APIs

```http
POST   /api/v1/blood-donation/donor-profile
GET    /api/v1/blood-donation/donor-profile/me
PUT    /api/v1/blood-donation/donor-profile/me
POST   /api/v1/blood-donation/donor-profile/me/activate
POST   /api/v1/blood-donation/donor-profile/me/deactivate
```

## Availability APIs

```http
GET    /api/v1/blood-donation/availability/me
PUT    /api/v1/blood-donation/availability/me
POST   /api/v1/blood-donation/availability/pause
POST   /api/v1/blood-donation/availability/resume
```

## Blood request APIs

```http
POST   /api/v1/blood-requests
GET    /api/v1/blood-requests
GET    /api/v1/blood-requests/{id}
POST   /api/v1/blood-requests/{id}/cancel
GET    /api/v1/blood-requests/{id}/timeline
```

## Matching APIs

Normally matching should be backend-controlled rather than a public “match all donors” endpoint.

Administrative/internal endpoints may be:

```http
POST   /api/v1/internal/blood-requests/{id}/run-matching
```

## Donor response APIs

```http
GET    /api/v1/blood-donation/requests
GET    /api/v1/blood-donation/requests/{id}
POST   /api/v1/blood-donation/requests/{id}/accept
POST   /api/v1/blood-donation/requests/{id}/decline
POST   /api/v1/blood-donation/requests/{id}/withdraw
```

## Donation history APIs

```http
GET    /api/v1/blood-donation/history
GET    /api/v1/blood-donation/history/{id}
```

## Rewards APIs

```http
GET    /api/v1/blood-donation/rewards/balance
GET    /api/v1/blood-donation/rewards/transactions
GET    /api/v1/blood-donation/rewards/badges
```

---

# 52. API Response Example — Blood Request

```json
{
  "id": "bd-000123",
  "bloodType": "O+",
  "requiredUnits": 2,
  "urgencyLevel": "URGENT",
  "status": "DONORS_CONTACTED",
  "facility": {
    "id": "fac-001",
    "name": "Authorized Facility",
    "area": "Mansoura"
  },
  "neededBy": "2026-10-06T20:00:00+03:00",
  "acceptedDonorCount": 1,
  "verifiedDonationCount": 0
}
```

Do not include donor private fields in the patient response by default.

---

# 53. API Response Example — Donor Request

```json
{
  "id": "resp-0001",
  "requestId": "bd-000123",
  "bloodType": "O+",
  "facilityName": "Authorized Facility",
  "area": "Mansoura",
  "distanceKm": 4.8,
  "urgency": "URGENT",
  "status": "PENDING",
  "responseDeadline": "2026-10-06T19:25:00+03:00"
}
```

---

# 54. Authorization Rules

## Donor permissions

A donor can access:

```text
Own donor profile
Own availability
Own matched requests
Own responses
Own donation history
Own points
Own notifications
Own safety reports
```

A donor cannot access:

```text
Other donors' private profile
Other users' medical records
All blood requests in the database
Admin settings
Other donors' reward transactions
```

## Patient permissions

A patient can access:

```text
Own blood requests
Own request timeline
Aggregated donor response information
Own notifications
```

## Facility permissions

A facility user can access only records within their authorized facility scope.

## Admin permissions

Admin access should be permission-based rather than simply checking a generic `IsAdmin` flag everywhere.

Example permissions:

```text
BloodRequests.View
BloodRequests.Manage
Donors.View
Donors.Verify
Donors.Suspend
Donations.Verify
Rewards.Manage
SafetyReports.Manage
AuditLogs.View
```

---

# 55. Object-Level Authorization

Every endpoint handling a resource ID must verify ownership/scope on the server.

Bad:

```text
GET /api/v1/blood-requests/123
```

and assuming the frontend already hid it.

Correct logic:

```text
User can access request 123?
        ↓
Requester owns request? OR
Authorized facility owns request? OR
Admin has permission?
        ↓
YES → continue
NO  → 403 / 404 according to policy
```

Never trust IDs coming from the client.

---

# 56. Security Requirements

## Authentication

Use the shared TechCare authentication system.

Do not create a separate donor login system.

## Token security

Protect JWT access/refresh-token flows according to the global authentication architecture.

## Validation

Validate:

- Blood type enum
- Quantity
- Dates
- Facility ID
- Location fields
- Duplicate responses
- Request status transitions

## Rate limiting

Apply rate limits to:

- Blood request creation
- Donor registration
- Accept/decline endpoints
- Report creation
- Notification-triggering operations

## Audit

Log sensitive operations:

```text
DonorCreated
DonorActivated
DonorPaused
BloodRequestCreated
BloodRequestCancelled
MatchGenerated
DonorAccepted
DonorDeclined
DonationVerified
RewardGranted
RewardReversed
DonorSuspended
```

---

# 57. Idempotency

Important operations should be safely repeatable.

Examples:

```text
POST /blood-requests/{id}/accept
```

If the request is accidentally retried, it should not create two responses.

Similarly:

```text
VerifyDonation
GrantReward
ExpireRequest
```

should be idempotent.

A unique constraint can help:

```text
Unique(BloodRequestId, DonorId)
```

for donor responses.

---

# 58. Background Jobs

Recommended background jobs:

### Match Pending Request

```text
Find candidates
Rank candidates
Create responses / notification records
Send notifications
```

### Expire Requests

```text
Find requests with ResponseDeadline < now
Move valid open requests → EXPIRED
```

### Notification Reminder

Send configured reminder when appropriate.

### Donation Reminder

Notify accepted donors before scheduled coordination.

### Reward Processing

Create reward transaction after verified donation completion.

### Cleanup

Expire stale temporary matching records and notification jobs safely.

---

# 59. Event-Driven Design Option

The module can use domain events internally.

Example:

```text
BloodRequestCreated
        ↓
BloodRequestMatchingRequested
        ↓
DonorsMatched
        ↓
NotificationsQueued
        ↓
DonorAccepted
        ↓
DonationCoordinationConfirmed
        ↓
DonationCompleted
        ↓
RewardGranted
```

This design keeps the controller thin and separates matching/notifications/rewards logic.

---

# 60. Suggested Service Layer

Example ASP.NET Core service structure:

```text
IBloodRequestService
IBloodMatchingService
IDonorService
IDonorAvailabilityService
IDonationCoordinationService
IDonationVerificationService
IRewardService
IBloodNotificationService
IBloodSafetyService
```

Implementation examples:

```text
BloodRequestService
BloodMatchingService
DonorService
DonorAvailabilityService
DonationCoordinationService
DonationVerificationService
RewardService
BloodNotificationService
BloodSafetyService
```

---

# 61. Suggested Controller Structure

```text
BloodRequestsController
DonorsController
DonorAvailabilityController
DonorRequestsController
DonationHistoryController
RewardsController
BloodFacilitiesController
BloodReportsController
```

Internal/admin controllers can be separated where useful.

---

# 62. DTOs

Suggested DTOs:

```text
CreateDonorProfileRequest
UpdateDonorProfileRequest
UpdateDonorAvailabilityRequest
CreateBloodRequestRequest
BloodRequestResponseDto
BloodRequestDetailsDto
DonorMatchDto
AcceptBloodRequestDto
DeclineBloodRequestDto
DonationAppointmentDto
DonationRecordDto
RewardBalanceDto
RewardTransactionDto
CreateSafetyReportRequest
```

Do not expose EF Core entities directly from controllers.

---

# 63. Validation Rules

## Donor profile

```text
Date of birth is valid
Blood type belongs to configured enum
Maximum distance >= 0
Required consent exists
```

## Blood request

```text
RequiredUnits > 0
Facility exists and is active
Blood type is valid
NeededBy is valid
Request location is valid
Requester has permission
```

## Donor response

```text
Request exists
Request is open
Donor is active
Donor is available
Donor hasn't already responded
Matching rules still pass
```

## Donation completion

```text
Authorized actor
Appointment exists when required
Donation not already completed
Verification data present
```

---

# 64. Error Cases

## Patient creates duplicate request

Response:

```text
A similar active request already exists.
```

Offer to open the existing request.

## Donor opens expired request

Show:

```text
This request is no longer accepting responses.
```

## Donor accepts after request is fulfilled

Show:

```text
This request has already been fulfilled.
```

## Donor became unavailable

Show:

```text
You are currently unavailable for new donation requests.
```

## Facility is suspended

Do not allow new coordination against it.

## Notification failed

Save notification as failed and retry according to the messaging architecture.

---

# 65. Cancellation Rules

## Patient cancellation

Patient may cancel when policy allows.

Example:

```text
OPEN → CANCELLED
```

All contacted donors receive an update.

## Donor withdrawal

Donor may withdraw before a configured cutoff.

The platform should avoid unnecessary punishment for legitimate withdrawals while recording the event for operational analytics.

## Facility cancellation

Authorized facility can cancel the coordination with a reason category.

---

# 66. No-Show Workflow

Possible flow:

```text
ACCEPTED
   ↓
APPOINTMENT_CONFIRMED
   ↓
NO_SHOW
```

Who can mark no-show?

Recommended:

- Authorized facility staff
- Admin with appropriate permission

Not the patient alone.

Repeated no-show behavior may be subject to moderation, but legitimate circumstances should remain reportable.

---

# 67. Donation Cooling / Restriction Rules

The system may store configurable restriction windows based on verified donation records.

Do not hard-code a universal medical rule into the application without the project's authorized clinical policy.

Recommended configuration entity:

```text
DonorEligibilityPolicy
----------------------
Id
PolicyName
PolicyType
Value
Unit
Active
EffectiveFrom
EffectiveTo
```

Then the matching service can use policy data without embedding clinical rules in application code.

---

# 68. Blood Group Compatibility Configuration

Use a configurable table instead of scattering ABO/Rh logic across services.

Example:

```text
BloodCompatibilityRule
----------------------
RequesterType
DonorType
Context
AllowedForInitialMatch
ClinicalConfirmationRequired
Active
```

`Context` could later distinguish:

```text
RED_CELL_DONATION
PLASMA
OTHER
```

The MVP can support only the context actually required by the project.

---

# 69. Important Medical Safety Boundary

The matching result should be phrased as:

```text
Potential donor match
```

not:

```text
Guaranteed compatible transfusion donor
```

The final transfusion compatibility check belongs to the clinical workflow.

---

# 70. Full Donor Journey

```text
User Login
   ↓
Blood Donation
   ↓
Become a Donor
   ↓
Complete Profile
   ↓
Add Blood Type
   ↓
Consent
   ↓
Activate Availability
   ↓
Wait for Match
   ↓
Receive Blood Request
   ↓
View Request
   ↓
Accept
   ↓
Facility Coordination
   ↓
Appointment / Visit
   ↓
Medical Screening by Facility
   ↓
Donation
   ↓
Facility Confirms Donation
   ↓
TechCare Records Donation
   ↓
Reward Points Granted
   ↓
Donation History Updated
   ↓
Donor Available Again When Policy Allows
```

---

# 71. Full Patient Blood Request Journey

```text
Patient Login
   ↓
Blood Donation
   ↓
Request Blood
   ↓
Select Blood Type
   ↓
Enter Units
   ↓
Select Facility
   ↓
Select Needed By
   ↓
Confirm Request
   ↓
Request Created
   ↓
Matching Engine
   ↓
Compatible Candidates Found
   ↓
Notifications Sent
   ↓
Donors Accept
   ↓
Facility Coordination
   ↓
Verified Donations
   ↓
Request Fulfilled
   ↓
All Remaining Notifications Closed
```

---

# 72. Full Facility Journey

```text
Facility Login
   ↓
View Blood Coordination Request
   ↓
Confirm Request
   ↓
Review Donor Appointment
   ↓
Donor Arrives
   ↓
Medical Screening
   ↓
Donation Completed / Not Completed
   ↓
Submit Verification Result
   ↓
TechCare Updates DonationRecord
   ↓
Reward Service Triggered
   ↓
Request Fulfillment Updated
```

---

# 73. End-to-End Example

## Scenario

Patient requires blood at an authorized facility.

### Step 1

Patient creates:

```text
Request = BD-1001
Blood Type = O+
Required Units = 2
Urgency = URGENT
Facility = Facility A
```

### Step 2

Matching finds:

```text
Donor 1 → O+ → 3 km → available
Donor 2 → O+ → 7 km → available
Donor 3 → incompatible / excluded
```

### Step 3

Notifications are sent to Donor 1 and Donor 2.

### Step 4

Donor 1 accepts.

State:

```text
Request = DONORS_CONTACTED
AcceptedDonorCount = 1
VerifiedDonationCount = 0
```

### Step 5

Facility confirms coordination.

### Step 6

Donor 1 attends screening and donation.

### Step 7

Facility verifies completed donation.

```text
VerifiedDonationCount = 1
```

### Step 8

System still needs one verified donation.

The request remains:

```text
PARTIALLY_FULFILLED
```

### Step 9

Donor 2 accepts and later completes donation.

```text
VerifiedDonationCount = 2
```

Request becomes:

```text
FULFILLED
```

### Step 10

Reward service grants points to both verified donors.

---

# 74. Notification Timeline Example

```text
09:00
Request created

09:01
Matching completed

09:01
Donor A notified

09:02
Donor B notified

09:05
Donor A accepted

09:05
Patient notified

09:20
Facility confirmed

13:30
Donor A appointment reminder

14:00
Donor A arrived

14:30
Donation verified

14:31
Reward granted
```

---

# 75. Admin Workflow

Admin dashboard section:

```text
Admin
 ↓
Blood Donation
 ├── Donors
 ├── Blood Requests
 ├── Donation Records
 ├── Facilities
 ├── Safety Reports
 ├── Rewards
 ├── Eligibility Policies
 ├── Compatibility Rules
 └── Audit Logs
```

## Donor moderation

Admin can:

- View donor status
- Review verification state
- Suspend donor
- Restore donor
- View reports
- View activity history

Every sensitive admin action requires:

- Admin identity
- Timestamp
- Reason
- Audit record

---

# 76. Admin Blood Request Controls

Admin can:

- Review suspicious requests
- Suspend request
- Cancel request
- Restore when appropriate
- View request activity
- Review donor notification activity

Admin should not casually modify verified medical donation data.

Clinical information should remain under authorized clinical ownership.

---

# 77. Admin Reward Controls

Admin can configure:

```text
PointsPerVerifiedDonation
CampaignBonus
BadgeThresholds
RewardExpiryPolicy (if used)
```

Changes should be versioned.

Historical transactions should not be silently recalculated because a policy changed.

---

# 78. Audit Log Example

```json
{
  "actorUserId": "admin-1",
  "action": "DONOR_SUSPENDED",
  "entityType": "DonorProfile",
  "entityId": "donor-123",
  "reason": "Repeated fraudulent activity reports",
  "timestamp": "2026-10-06T18:00:00+03:00"
}
```

---

# 79. Performance / Database Indexing

Recommended indexes:

```text
DonorProfile(Status, IsAvailable)
DonorBloodType(BloodType, VerificationStatus)
BloodRequest(Status, NeededBy)
BloodRequest(FacilityId, Status)
BloodRequestResponse(BloodRequestId, DonorId)
BloodRequestResponse(DonorId, Status)
DonationAppointment(DonorId, ScheduledDateTime)
DonationAppointment(FacilityId, ScheduledDateTime)
DonationRecord(DonorId, DonationDate)
RewardTransaction(UserId, CreatedAt)
SafetyReport(Status, CreatedAt)
```

For geospatial search, use the appropriate database/location strategy selected by the team rather than scanning every donor record.

---

# 80. Pagination

Never return all donors or all requests at once.

Use:

```text
page
pageSize
sort
filters
```

or cursor-based pagination for high-volume endpoints.

Example:

```http
GET /api/v1/blood-donation/requests?page=1&pageSize=20
```

---

# 81. Search / Filter Options

Patient blood requests:

- Status
- Blood type
- Facility
- Urgency
- Date range

Donor request inbox:

- Distance
- Urgency
- Needed by
- Facility area
- Status

Admin:

- Donor status
- Request status
- Reports
- Date range

---

# 82. Testing Strategy

The Blood Donor module requires unit, integration, authorization, concurrency, and end-to-end tests.

## Unit tests

### Donor service

- Create donor profile
- Update profile
- Activate
- Pause
- Resume

### Matching service

- Blood group filter
- Availability filter
- Distance filter
- Facility preference
- Configured restrictions
- Ranking

### Request service

- Create request
- Cancel request
- Expire request
- Fulfillment transition

### Reward service

- Reward verified donation
- Prevent duplicate reward
- Reverse reward

---

# 83. Integration Tests

## Scenario 1 — Create request

```text
POST /blood-requests
```

Expected:

```text
201 Created
Status = PENDING
```

## Scenario 2 — Donor accepts

Expected:

```text
200 / 201
Response = ACCEPTED
```

and no duplicate response.

## Scenario 3 — Unauthorized access

Donor tries to access another donor's private profile.

Expected:

```text
403 or 404
```

according to project policy.

---

# 84. Concurrency Tests

### Test A — Two requests accepted simultaneously

Only valid state transitions are committed.

### Test B — Donation verified twice

Only one reward is granted.

### Test C — Request cancelled while donor accepts

One final consistent state is produced.

### Test D — Donor paused while matching job runs

The service re-checks availability before finalizing notification/response state.

---

# 85. Security Tests

Test:

- JWT required
- Role claims enforced
- Object-level authorization
- Broken access control attempts
- ID enumeration
- Rate limiting
- Duplicate requests
- Duplicate rewards
- Input validation
- SQL/ORM injection safety
- XSS-safe text rendering
- Audit log creation
- Admin authorization

---

# 86. Abuse Prevention

Potential abuse:

### Fake blood requests

Controls:

- Request frequency limits
- Facility reference where available
- Report mechanism
- Admin review
- Duplicate-request detection

### Fake donation completion

Controls:

- Facility verification
- Admin override with audit
- No donor self-reward

### Spam notifications

Controls:

- Candidate caps
- Deduplication
- Notification rate limits
- Reminder limits

### Harassment

Controls:

- Limited contact exposure
- In-app messaging only if necessary
- Block/report feature
- Admin moderation

---

# 87. Observability

Log business events and failures.

Useful metrics:

```text
BloodRequestsCreated
MatchingJobsCompleted
AverageCandidatesPerRequest
NotificationSuccessRate
DonorAcceptanceRate
DonationCompletionRate
FulfillmentRate
ExpiredRequestRate
FakeRequestReports
DonorNoShowRate
RewardTransactions
```

Track errors by correlation ID.

Example:

```text
CorrelationId: TC-BD-9A81
RequestId: BD-000123
Operation: MatchBloodRequest
```

---

# 88. Recommended State Transition Rules

```text
Blood Request

DRAFT → PENDING
PENDING → MATCHING
MATCHING → DONORS_CONTACTED
DONORS_CONTACTED → PARTIALLY_FULFILLED
PARTIALLY_FULFILLED → FULFILLED
DONORS_CONTACTED → FULFILLED

PENDING → CANCELLED
MATCHING → CANCELLED
DONORS_CONTACTED → CANCELLED
PARTIALLY_FULFILLED → CANCELLED

PENDING → EXPIRED
MATCHING → EXPIRED
DONORS_CONTACTED → EXPIRED
```

Invalid examples:

```text
FULFILLED → MATCHING
CANCELLED → DONORS_CONTACTED
EXPIRED → ACCEPTED
```

unless an explicit admin recovery workflow exists.

---

# 89. Donor State Machine

```text
PENDING_VERIFICATION
        ↓
      ACTIVE
      ↙   ↘
  PAUSED  RESTRICTED
      ↓       ↓
    ACTIVE   ACTIVE

ACTIVE → SUSPENDED
ACTIVE → DEACTIVATED
PAUSED → DEACTIVATED
```

---

# 90. Donor Response State Machine

```text
PENDING
  ├──→ ACCEPTED
  ├──→ DECLINED
  └──→ EXPIRED

ACCEPTED
  ├──→ WITHDRAWN
  ├──→ CANCELLED
  ├──→ NO_SHOW
  └──→ COMPLETED
```

---

# 91. Definition of Done — Backend

The Blood Donor backend is complete when:

- Entities and migrations are implemented.
- Donor onboarding works.
- Blood request creation works.
- Matching service works.
- Compatibility rules are configurable.
- Location filtering works.
- Donor availability works.
- Donor responses work.
- Request state transitions are enforced.
- Facility verification works.
- Donation records are stored.
- Reward ledger works.
- Notifications are integrated.
- Authorization is enforced.
- Object-level access is tested.
- Concurrency is handled.
- Idempotency is handled.
- Audit logs exist.
- Unit and integration tests exist.
- API documentation is available.

---

# 92. Definition of Done — Frontend

- Donor onboarding screen implemented.
- Donor dashboard implemented.
- Availability controls implemented.
- Blood request list implemented.
- Request details implemented.
- Accept / decline actions implemented.
- Patient request creation implemented.
- Request tracking implemented.
- Donation history implemented.
- Points dashboard implemented.
- Notifications implemented.
- Error / loading / empty states implemented.
- Responsive design implemented.
- Authorization-aware navigation implemented.
- Accessibility basics handled.

---

# 93. Definition of Done — Integration

Complete integration means:

```text
Patient
  ↓
Blood Request
  ↓
Matching
  ↓
Donor Notification
  ↓
Donor Acceptance
  ↓
Facility Coordination
  ↓
Donation Verification
  ↓
Request Fulfillment
  ↓
Reward
  ↓
Donation History
```

All major steps must have persisted states, authorization, and auditability.

---

# 94. MVP Scope

For the TechCare prototype, the recommended MVP can include:

### Donor

- Donor profile
- Blood type
- Availability
- Preferred location
- Donor request inbox
- Accept / decline
- Donation history
- Points

### Patient

- Create blood request
- Blood type
- Required quantity
- Facility/location
- Request status
- Donor response count

### Matching

- Blood group matching
- Distance filtering
- Availability filtering
- Basic ranking
- Notifications

### Verification

- Authorized facility confirmation
- Donation completion

### Admin

- Donor management
- Blood request moderation
- Donation verification
- Reports
- Rewards management

---

# 95. Future Enhancements

These can remain outside the initial prototype:

- Real blood-bank API integration
- Hospital/EHR integration
- Automated facility capacity checks
- Advanced geospatial optimization
- Multi-channel emergency notification
- Mobile push optimization
- QR-based donation verification
- Verified digital donor card
- Campaign management
- Geographic demand heatmaps
- Advanced analytics
- Fraud scoring
- Recommendation engine
- Blood inventory integration with authorized blood banks
- Multi-facility regional coordination

---

# 96. Recommended TechCare Module Structure

For ASP.NET Core:

```text
TechCare
│
├── Domain
│   └── BloodDonation
│       ├── Entities
│       ├── Enums
│       ├── ValueObjects
│       └── Rules
│
├── Application
│   └── BloodDonation
│       ├── Donors
│       ├── Requests
│       ├── Matching
│       ├── Donations
│       ├── Rewards
│       └── Reports
│
├── Infrastructure
│   └── BloodDonation
│       ├── Persistence
│       ├── Repositories
│       ├── Jobs
│       └── Notifications
│
└── API
    └── Controllers
        ├── DonorsController
        ├── BloodRequestsController
        ├── DonorRequestsController
        ├── DonationsController
        └── RewardsController
```

The exact architecture may follow the team's agreed project structure, but the domain responsibilities should remain separated.

---

# 97. Recommended Frontend Module Structure

```text
src/
└── features/
    └── blood-donation/
        ├── donor/
        │   ├── pages/
        │   ├── components/
        │   ├── services/
        │   └── models/
        ├── requests/
        │   ├── pages/
        │   ├── components/
        │   ├── services/
        │   └── models/
        ├── rewards/
        └── notifications/
```

This keeps the module independently maintainable by the Blood Donor owner while sharing global auth, UI, and infrastructure.

---

# 98. Suggested API Authorization Matrix

| Endpoint Group | Patient | Donor | Facility | Admin |
|---|---:|---:|---:|---:|
| Create blood request | ✅ | ❌/optional | ✅ | ✅ |
| View own requests | ✅ | ❌ | ✅ scoped | ✅ |
| Donor profile | own | ✅ | limited | ✅ |
| Donor matched requests | ❌ | ✅ | ❌ | ✅ |
| Accept donor request | ❌ | ✅ | ❌ | controlled |
| Donation verification | ❌ | ❌ | ✅ | ✅ |
| Donation history | own | own | scoped | ✅ |
| Reward balance | own | own | ❌ | ✅ |
| Safety reports | ✅ | ✅ | ✅ | ✅ |
| Compatibility rules | ❌ | ❌ | ❌ | ✅ |
| Eligibility policy | ❌ | read applicable | ❌ | ✅ |
```

The exact matrix must be reflected in backend policy configuration and tested.

---

# 99. Complete Blood Donation Module Diagram

```text
                         TECHCARE
                            │
             ┌──────────────┴──────────────┐
             │                             │
         PATIENT                         DONOR
             │                             │
      Create Blood Request        Become Donor
             │                             │
             └──────────────┬──────────────┘
                            │
                       MATCHING ENGINE
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      Blood Type         Location        Availability
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                      Notifications
                            │
                        Donor Accept
                            │
                   Donation Coordination
                            │
                    Authorized Facility
                            │
                      Medical Screening
                            │
                  ┌─────────┴─────────┐
                  │                   │
              Completed           Not Completed
                  │
            Verified Donation
                  │
        ┌─────────┴──────────┐
        │                    │
  Request Fulfillment      Rewards
        │                    │
     Patient              Donor
```

---

# 100. Complete Scenario Checklist

## Donor onboarding

- [ ] Existing TechCare account authenticated
- [ ] Donor profile created
- [ ] Blood type captured
- [ ] Blood type source captured
- [ ] Consent saved
- [ ] Availability configured
- [ ] Location preference configured
- [ ] Notification preferences configured

## Requester

- [ ] Blood request created
- [ ] Blood type validated
- [ ] Quantity validated
- [ ] Facility validated
- [ ] Needed-by time validated
- [ ] Request status persisted

## Matching

- [ ] Active donor filter
- [ ] Availability filter
- [ ] Blood compatibility configuration
- [ ] Distance filter
- [ ] Preference ranking
- [ ] Duplicate prevention
- [ ] Notification deduplication

## Donor response

- [ ] Request viewed
- [ ] Accept
- [ ] Decline
- [ ] Withdraw
- [ ] Response status persisted

## Coordination

- [ ] Facility confirmation
- [ ] Appointment/coordination record
- [ ] Reminder
- [ ] Arrival
- [ ] Screening outcome category
- [ ] Donation verification

## Completion

- [ ] Donation record created
- [ ] Request counters updated
- [ ] Request fulfilled if appropriate
- [ ] Reward granted once
- [ ] History updated
- [ ] Notifications sent

## Safety

- [ ] Report
- [ ] Block
- [ ] Rate limits
- [ ] Admin review
- [ ] Audit log

---

# 101. Final End-to-End Workflow

```text
                    ┌───────────────────────────┐
                    │      TechCare User        │
                    └─────────────┬─────────────┘
                                  │
                           Shared Authentication
                                  │
                ┌─────────────────┴─────────────────┐
                │                                   │
             PATIENT                              DONOR
                │                                   │
        Create Blood Request                Complete Donor Profile
                │                                   │
        Choose Blood Type                    Verify/record blood type
                │                                   │
        Choose Facility                      Set availability
                │                                   │
        Set Required Units                   Set service area
                │                                   │
                └─────────────────┬─────────────────┘
                                  │
                           Matching Engine
                                  │
                     ┌────────────┼────────────┐
                     │            │            │
                 Compatibility  Distance   Availability
                     │            │            │
                     └────────────┼────────────┘
                                  │
                            Candidate List
                                  │
                             Notifications
                                  │
                           Donor Response
                         ┌────────┴────────┐
                         │                 │
                      Decline           Accept
                         │                 │
                    End response     Coordination
                                           │
                                     Authorized Facility
                                           │
                                    Medical Screening
                                           │
                                    Donation Event
                                      ┌────┴────┐
                                      │         │
                                  Completed   Not Completed
                                      │
                                Verification
                                      │
                       ┌──────────────┴─────────────┐
                       │                            │
                 Donation History              Reward Ledger
                       │                            │
                       └──────────────┬─────────────┘
                                      │
                               Request Fulfillment
                                      │
                           ┌──────────┴──────────┐
                           │                     │
                       Partial               Fulfilled
                           │                     │
                           └──── Continue        End
                                Matching if needed
```

---

# 102. Final Implementation Principle

The Blood Donor module should be treated as a coordination and workflow system, not a blood-commerce feature.

The clean architecture is:

```text
Shared Auth
    ↓
User / Patient / Donor
    ↓
Blood Request
    ↓
Matching Engine
    ↓
Donor Response
    ↓
Authorized Facility
    ↓
Donation Verification
    ↓
Donation History + Rewards
```

The most important backend guarantees are:

1. No unauthorized access to donor or patient data.
2. No duplicate donor response.
3. No duplicate donation verification.
4. No duplicate reward.
5. No matching to unavailable/suspended donors.
6. No new notifications after request cancellation/fulfillment.
7. No claim of medical compatibility without clinical confirmation.
8. No blood sale or donor payment workflow.
9. Every sensitive state change is auditable.
10. Final medical eligibility and donation decisions remain with authorized healthcare professionals.

---

# 103. Blood Donor Module — Handoff Summary

A developer taking ownership of this module should be able to answer all of the following before implementation:

```text
Who can become a donor?
What data is required?
How is blood type stored and verified?
How does donor availability work?
How does a patient create a request?
How are donors matched?
How is distance calculated?
How is compatibility represented?
What information is visible to each role?
What happens when a donor accepts?
What happens when a donor declines?
What happens when the request expires?
How is donation completion verified?
How is history updated?
How are reward points granted?
How are duplicate rewards prevented?
How are complaints handled?
How is object-level authorization enforced?
How are concurrent accepts handled?
Which operations are audited?
Which parts belong to the authorized facility instead of TechCare?
```

This document defines those responsibilities and provides the implementation baseline for the Blood Donor module of TechCare.
