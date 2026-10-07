# TechCare — Admin Complete Workflow

> **Document Type:** Full Admin Workflow Specification
>
> **Project:** TechCare Healthcare Platform
>
> **Audience:** Full-Stack .NET Team — Backend, Frontend, Database, QA, Product
>
> **Scope:** Prototype / MVP Admin Portal
>
> **Goal:** Give the TechCare Admin team complete operational control over the platform while protecting patient/provider data, enforcing authorization, handling verification, complaints, financial operations, service catalogs, and system monitoring.
>
---

# 1. Admin Role Overview

The Admin is not another normal platform provider.

The Admin is the operational and governance layer that supervises the entire TechCare ecosystem:

- Patients
- Doctors
- Nurses
- Pharmacies and Pharmacists
- Laboratories and Laboratory Staff
- Blood Donors
- Appointments
- Orders
- Laboratory bookings and samples
- Blood requests
- Complaints
- Ratings
- Payments
- Refunds
- Withdrawals
- Platform catalogs
- Notifications
- Security events
- Audit logs
- System configuration

The Admin should control the platform without bypassing application security.

The Admin dashboard must expose only the minimum data required for a specific administrative task.

---

# 2. Core Admin Principles

## 2.1 Least Privilege

Every admin account must receive only the permissions required for its job.

Example:

```text
Support Admin
    -> complaints
    -> user support
    -> appointment/order visibility
    -> cannot approve doctor license documents
    -> cannot execute financial withdrawals
```

```text
Verification Admin
    -> doctor verification
    -> nurse verification
    -> pharmacy verification
    -> laboratory verification
    -> cannot change financial ledger
```

```text
Finance Admin
    -> transactions
    -> refunds
    -> withdrawal review
    -> financial reports
    -> cannot modify clinical records
```

```text
Super Admin
    -> all administrative permissions
```

## 2.2 Object-Level Authorization

Being an Admin does not automatically mean every Admin can access every object.

Authorization must validate:

```text
Admin Identity
    + Admin Role
    + Permission
    + Requested Resource
    + Requested Action
    = Authorized Operation
```

## 2.3 Audit Everything Important

Sensitive administrative actions must produce an audit record.

Examples:

- Approve provider
- Reject provider
- Suspend account
- Restore account
- Edit a catalog item
- Change service price
- Approve withdrawal
- Reject withdrawal
- Issue refund
- Resolve complaint
- Hide abusive content
- Change system setting
- Assign another Admin
- Change admin permission

## 2.4 Never Store or Display Passwords

Admin must never see:

- User passwords
- Password hashes
- Refresh token values
- OTP secrets
- Other authentication secrets

The Admin can trigger account recovery workflows, but must not know the user's password.

---

# 3. Admin Actors

## 3.1 Super Admin

Full system administrator.

Responsibilities:

- Manage Admin users
- Manage permissions
- Manage system settings
- Access all operational dashboards
- Handle escalated security issues
- Handle final verification escalations
- Review sensitive audit events

## 3.2 Verification Admin

Responsible for professional and organization verification.

Can review:

- Doctor identity/professional documents
- Nurse identity/professional documents
- Pharmacy documents
- Laboratory documents

## 3.3 Support Admin

Responsible for platform support and complaints.

Can manage:

- Patient support tickets
- Provider complaints
- Order issues
- Booking issues
- Cancellation disputes
- Rating reports

## 3.4 Finance Admin

Responsible for financial operations.

Can manage:

- Payment status review
- Refunds
- Provider earnings review
- Withdrawal requests
- Financial reconciliation
- Financial reports

## 3.5 Operations Admin

Responsible for day-to-day platform operations.

Can monitor:

- Appointments
- Orders
- Lab bookings
- Blood requests
- Failed workflows
- Provider availability
- System activity

---

# 4. Admin Authentication Workflow

## 4.1 Login

Admin logs in through a dedicated admin portal endpoint.

Example:

```http
POST /api/v1/admin/auth/login
```

Request:

```json
{
  "email": "admin@techcare.local",
  "password": "********"
}
```

Response concept:

```json
{
  "accessToken": "...",
  "refreshToken": "...",
  "expiresIn": 900,
  "admin": {
    "id": "...",
    "name": "System Admin",
    "role": "SuperAdmin",
    "permissions": ["users.read", "users.suspend"]
  }
}
```

## 4.2 Admin MFA / OTP

Sensitive Admin accounts should support multi-factor authentication.

Recommended flow:

```text
Email + Password
      ↓
Credential Validation
      ↓
MFA / OTP Challenge
      ↓
OTP Validation
      ↓
Issue Access Token
      ↓
Open Admin Dashboard
```

## 4.3 Admin Session Security

Use short-lived access tokens.

Use refresh tokens with rotation/blacklisting strategy appropriate to the application's authentication design.

Admin sessions should support:

- Logout
- Session revocation
- Suspicious-login detection
- Device/session listing
- Forced logout by Super Admin

## 4.4 Failed Login

After repeated failed attempts:

```text
Failed Login
   ↓
Track Attempt
   ↓
Rate Limit
   ↓
Temporary Lock if Threshold Reached
   ↓
Security Log
```

Do not permanently lock an account without a controlled recovery path.

---

# 5. Admin Dashboard Workflow

After authentication:

```text
Admin Login
   ↓
Load Admin Profile
   ↓
Load Role + Permissions
   ↓
Load Dashboard KPIs
   ↓
Load Pending Actions
   ↓
Load Alerts
   ↓
Open Dashboard
```

## 5.1 Dashboard Sections

### KPI Cards

Examples:

- Total Users
- Active Patients
- Active Doctors
- Active Nurses
- Verified Pharmacies
- Verified Laboratories
- Registered Donors
- Pending Verifications
- Active Appointments
- Active Pharmacy Orders
- Active Lab Bookings
- Active Blood Requests
- Pending Complaints
- Pending Withdrawals
- Completed Transactions

### Operational Alerts

Examples:

- Large number of expired doctor requests
- Unprocessed complaints
- Pending provider verifications
- Failed payments
- Failed delivery attempts
- Failed lab sample collection
- Suspicious account activity
- Expiring provider documents
- Very low inventory reported by pharmacies
- Repeated blood-request cancellations

### Quick Actions

Examples:

- Review Doctor Verification
- Review Nurse Verification
- Review Pharmacy Verification
- Review Laboratory Verification
- Review Complaints
- Review Withdrawals
- Add Medicine
- Add Diagnostic Test
- View Audit Log

---

# 6. Dashboard Data Strategy

Dashboard values should come from aggregated queries rather than loading every user or transaction record.

Examples:

```sql
COUNT(active_users)
COUNT(pending_verifications)
COUNT(open_complaints)
SUM(completed_provider_earnings)
```

Do not load entire tables into memory just to calculate dashboard totals.

Recommended database indexes should support common dashboard filters.

---

# 7. User Management Workflow

## 7.1 User Search

Admin can search users by controlled fields.

Search examples:

- User ID
- Name
- Email
- Phone
- Role
- Account Status
- Registration Date

Do not expose unnecessary medical information in general user search results.

## 7.2 User List

Example columns:

| Field | Purpose |
|---|---|
| User ID | Unique identifier |
| Name | Display |
| Email | Account identification |
| Phone | Support verification |
| Role | Patient / Doctor / Nurse / etc. |
| Status | Active / Suspended / Pending |
| Created At | Registration history |
| Last Login | Security/operations |
| Actions | View / Suspend / Restore |

## 7.3 User Profile View

Admin profile page should have sections:

```text
Account
Profile
Roles
Verification
Activity
Bookings / Orders Summary
Complaints
Financial Summary if applicable
Security Events
Audit History
```

## 7.4 Suspend User

Workflow:

```text
Open User
   ↓
Click Suspend
   ↓
Select Reason
   ↓
Optional Internal Note
   ↓
Confirm
   ↓
Update Account Status
   ↓
Revoke Active Sessions if required
   ↓
Notify User
   ↓
Write Audit Log
```

Example reasons:

- Policy violation
- Fraud suspicion
- Repeated abuse
- Fake information
- Security concern
- Harassment
- Administrative review

## 7.5 Restore User

```text
Suspended User
   ↓
Review Suspension
   ↓
Confirm Restore
   ↓
Set Status = Active
   ↓
Audit Log
   ↓
Notify User
```

---

# 8. Patient Administration Workflow

The Admin's Patient module is operational, not a substitute for a clinician.

## 8.1 Patient Search

Admin can locate a patient by:

- User ID
- Name
- Email
- Phone
- Status

## 8.2 Patient Information

Operational view may include:

- Basic profile
- Contact information
- Address summary
- Account status
- Booking history summary
- Order history summary
- Laboratory booking summary
- Blood request summary
- Complaint history

Clinical data should be protected and only displayed to authorized Admins when the administrative task genuinely requires it.

## 8.3 Patient Account Actions

Admin can:

- Suspend
- Restore
- Verify contact information where applicable
- Investigate complaints
- Review booking issues
- Review order issues
- Trigger account recovery process

Admin should not arbitrarily edit clinical records.

---

# 9. Doctor Verification Workflow

This is one of the most important Admin workflows.

## 9.1 Doctor Registration State

```text
Doctor Registered
      ↓
Profile Created
      ↓
Documents Submitted
      ↓
Pending Verification
```

The doctor must not receive `Verified` status before the required verification process completes.

## 9.2 Verification Queue

Dashboard:

```text
Verification Center
    ├── Doctors
    ├── Nurses
    ├── Pharmacies
    └── Laboratories
```

Each row shows:

- Applicant
- Submission date
- Verification status
- Document count
- Missing documents indicator
- Reviewer
- Action

## 9.3 Doctor Verification Details

Show:

- Full legal name
- Profile information
- Specialty
- Qualifications
- Professional information
- Submitted documents
- Submission timestamp
- Previous verification attempts
- Rejection history

Avoid displaying unrelated patient information.

## 9.4 Approve Doctor

```text
Pending Verification
   ↓
Review Data
   ↓
Review Documents
   ↓
Check Completeness
   ↓
Decision = Approve
   ↓
Provider Status = VERIFIED
   ↓
Publish Public Profile if Ready
   ↓
Notify Doctor
   ↓
Audit Log
```

## 9.5 Reject Doctor

Admin must select a clear rejection reason.

Example:

```json
{
  "decision": "Rejected",
  "reasonCode": "DOCUMENT_INVALID",
  "note": "Required professional document is unclear."
}
```

Doctor receives the actionable reason, not an internal-only note.

## 9.6 Request Changes

Useful state:

```text
PENDING
NEEDS_CHANGES
RESUBMITTED
APPROVED
REJECTED
```

`NEEDS_CHANGES` allows the doctor to update missing or incorrect data without registering a new account.

---

# 10. Nurse Verification Workflow

The same administrative pattern applies to nurses, but the document set is nursing-specific.

```text
Nurse Registered
   ↓
Professional Data Submitted
   ↓
Documents Uploaded
   ↓
Pending Review
   ↓
Approve / Needs Changes / Reject
```

Admin should not grant doctor-specific privileges to a nurse account.

---

# 11. Pharmacy Verification Workflow

## 11.1 Pharmacy Registration

```text
Create Organization
      ↓
Submit Pharmacy Profile
      ↓
Submit Required Documents
      ↓
Create Owner Account
      ↓
Pending Verification
```

## 11.2 Admin Review

Review:

- Pharmacy information
- Address
- Contact details
- Responsible person
- Submitted documents
- Staff linkage data

## 11.3 Decision

```text
Pending
 ├── Approve
 ├── Needs Changes
 └── Reject
```

## 11.4 Pharmacy Suspension

Admin may suspend a pharmacy from receiving new orders while preserving historical records.

Example:

```text
Verified Pharmacy
      ↓
Compliance / Operational Issue
      ↓
Suspend New Orders
      ↓
Keep Existing Records
      ↓
Notify Pharmacy
      ↓
Audit
```

The exact handling of active orders must be explicit in business rules.

---

# 12. Laboratory Verification Workflow

Workflow:

```text
Lab Registration
    ↓
Organization Data
    ↓
Professional / Operational Data
    ↓
Documents
    ↓
Pending Verification
    ↓
Admin Review
    ↓
Approve / Needs Changes / Reject
    ↓
Public Availability
```

Admin can suspend an approved laboratory from accepting new bookings while retaining historical test and result records.

---

# 13. Blood Donor Administration Workflow

The blood module requires extra care because it coordinates voluntary donation rather than functioning as a marketplace for blood.

## 13.1 Donor Account

Admin can review:

- Donor status
- Blood type declared by donor
- Eligibility information required by the workflow
- Approximate service location if necessary
- Donation history summary
- Request response activity
- Reward points ledger
- Complaints

## 13.2 Donor Status

Example:

```text
ACTIVE
TEMPORARILY_INELIGIBLE
SUSPENDED
DEACTIVATED
```

## 13.3 Blood Request Monitoring

Admin can monitor:

- Request status
- Requested blood type
- Number of matched donors
- Notifications sent
- Donor responses
- Fulfillment progress
- Cancellation
- Expiration

## 13.4 Fraud / Abuse Controls

Admin can flag:

- Repeated fake requests
- Excessive duplicate requests
- Suspicious donor accounts
- Reward abuse
- Automated/spam behavior

Admin should not manually claim that a donor is medically eligible unless the process explicitly delegates that verification to authorized medical personnel.

---

# 14. Medicine Master Catalog Workflow

The platform uses a shared medicine master catalog.

Pharmacies maintain their own inventory against the shared catalog.

## 14.1 Add Medicine

Fields may include:

- Medicine ID
- Generic Name
- Brand Name
- Active Ingredient
- Strength
- Dosage Form
- Package Description
- Search Keywords
- Status

Example:

```text
Medicine Master
      ↓
Pharmacy Inventory Item
      ↓
Available Quantity
      ↓
Price
      ↓
Expiry / Batch Data
```

## 14.2 Catalog States

```text
ACTIVE
INACTIVE
ARCHIVED
```

Do not hard-delete medicines that have historical order references.

## 14.3 Edit Medicine

Changes affecting historical transactions should not rewrite historical snapshots.

Example:

```text
Current Medicine Name = X
Historical Order Item Name = Y
```

The historical order should remain accurate as of the purchase time.

---

# 15. Diagnostic Test Master Catalog Workflow

The Laboratory module also needs a shared diagnostic test catalog.

Fields:

- Test ID
- Test Name
- Category
- Description
- Sample Type
- Preparation Instructions
- Status

Lab-specific pricing and availability remain separate.

---

# 16. Service Catalog Administration

Admin can manage platform-approved service definitions.

Examples:

### Nurse Services

- Home Injection
- Wound Dressing
- Vital Sign Measurement
- Elderly Care Visit
- Post-Procedure Nursing Support

### Doctor Services

- Home Consultation
- General Consultation
- Specialty Consultation

The exact service list should be configurable rather than hard-coded throughout the application.

## 16.1 Service Entity

Example:

```text
Service
- Id
- Name
- Category
- Description
- BasePricePolicy
- Active
- CreatedAt
- UpdatedAt
```

## 16.2 Deactivate Service

Do not delete a service already used by historical appointments.

Set:

```text
IsActive = false
```

Historical appointments continue showing the historical service snapshot.

---

# 17. Appointment Administration Workflow

Admin needs a global operational view over doctor and nurse appointments.

## 17.1 Appointment Search

Filters:

- Appointment ID
- Patient
- Provider
- Provider type
- Status
- Date range
- Service
- Payment status
- Location type

## 17.2 Appointment Lifecycle

```text
REQUESTED
    ↓
ACCEPTED
    ↓
CONFIRMED
    ↓
IN_PROGRESS
    ↓
COMPLETED
```

Alternative terminal states:

```text
REJECTED
EXPIRED
CANCELLED
NO_SHOW
```

## 17.3 Admin Intervention

Admin may intervene when:

- Support case requires review
- Payment dispute exists
- Provider is suspended
- Platform bug blocks progression
- Safety/security issue exists

Every manual status change must be logged.

Admin must avoid using direct database edits as an operational workflow.

---

# 18. Pharmacy Order Administration Workflow

Admin can inspect order lifecycle across the entire pharmacy module.

## 18.1 Order Search

Filters:

- Order ID
- Patient
- Pharmacy
- Status
- Payment status
- Fulfillment mode
- Date range

## 18.2 Order States

```text
PENDING
   ↓
ACCEPTED
   ↓
PREPARING
   ↓
READY_FOR_PICKUP / OUT_FOR_DELIVERY
   ↓
COMPLETED
```

Alternative states:

```text
REJECTED
CANCELLED
PAYMENT_FAILED
DELIVERY_FAILED
REFUND_PENDING
REFUNDED
```

## 18.3 Admin Order Investigation

Admin can see:

- Order metadata
- Item snapshots
- Price snapshots
- Pharmacy
- Patient identity required for support
- Fulfillment state
- Payment references
- Delivery data required for support
- Complaint links
- Audit history

Sensitive prescription data should only be visible to authorized personnel.

---

# 19. Laboratory Booking Administration

Admin can monitor laboratory bookings.

Filters:

- Booking ID
- Patient
- Laboratory
- Test
- Booking status
- Collection mode
- Date
- Result status

Admin can investigate:

- Booking acceptance issues
- Collection failures
- Sample rejection
- Recollection requests
- Result publication problems
- Payment/refund problems

Clinical result content should have stricter access control than simple booking metadata.

---

# 20. Laboratory Sample Operations Monitoring

Admin operational dashboard:

```text
Booked
  ↓
Collection Scheduled
  ↓
Collector Assigned
  ↓
Collected
  ↓
Received
  ↓
Processing
  ↓
Result Draft
  ↓
Validated
  ↓
Published
```

Admin can monitor exceptions:

- Collection failed
- Sample rejected
- Recollection required
- Result delayed
- Result publication blocked

Admin should not modify laboratory results as an ordinary support action.

---

# 21. Prescription Monitoring Workflow

Admin may monitor prescription metadata for platform operations.

Examples:

- Prescription ID
- Doctor
- Patient
- Creation time
- Linked consultation
- Fulfillment order if any

The Admin should not act as the doctor or alter clinical prescribing decisions.

If prescription correction is required, use a controlled workflow such as:

```text
Issue Reported
   ↓
Create Support / Clinical Review Case
   ↓
Authorized Professional Reviews
   ↓
Correction / Clarification
   ↓
Audit Event
```

---

# 22. Complaint Management Workflow

Complaints are a major Admin responsibility.

## 22.1 Complaint Creation

A complaint can reference:

- Doctor
- Nurse
- Pharmacy
- Laboratory
- Order
- Appointment
- Blood request
- Delivery issue
- Payment issue
- Platform issue

## 22.2 Complaint States

```text
OPEN
   ↓
UNDER_REVIEW
   ↓
WAITING_FOR_USER
   ↓
WAITING_FOR_PROVIDER
   ↓
ESCALATED
   ↓
RESOLVED
```

Alternative:

```text
CLOSED
REJECTED
DUPLICATE
```

## 22.3 Complaint Review

Admin opens complaint.

System shows:

```text
Complaint
   ↓
Related Entity
   ↓
Relevant Timeline
   ↓
Messages / Evidence
   ↓
Previous Cases
   ↓
Administrative Actions
```

## 22.4 Admin Response

Admin adds:

- Public response
- Internal note
- Status update
- Attachment if supported

Public notes and internal notes must be stored separately.

## 22.5 Escalation

Example:

```text
Support Admin
      ↓
Cannot Resolve
      ↓
Escalate
      ↓
Senior / Super Admin
      ↓
Final Decision
```

---

# 23. Ratings and Review Moderation

The Admin can moderate abusive or invalid reviews.

## 23.1 Review Flags

Reasons:

- Spam
- Harassment
- Sensitive personal information
- Fraudulent activity
- Irrelevant content
- Duplicated review

## 23.2 Moderation States

```text
VISIBLE
FLAGGED
UNDER_REVIEW
HIDDEN
RESTORED
```

## 23.3 Moderation Rule

Admin should not hide a review merely because it is negative.

The review moderation reason must be recorded.

---

# 24. Notification Administration

Admin may manage system-generated notifications.

Notification categories:

- Account
- Verification
- Appointment
- Order
- Laboratory
- Blood request
- Payment
- Complaint
- Security
- System

## 24.1 Template Management

Template fields may include:

- Template key
- Title
- Body
- Channel
- Language
- Active state

Example:

```text
APPOINTMENT_ACCEPTED
```

## 24.2 Notification Audit

Admin should know:

- Notification created
- Target user
- Channel
- Sent / failed
- Retry count
- Timestamp

Sensitive values should not be unnecessarily logged.

---

# 25. Financial Administration Workflow

Financial operations must be separated from ordinary CRUD permissions.

## 25.1 Financial Dashboard

Show aggregated data such as:

- Gross transaction volume
- Completed payments
- Pending payments
- Refund amount
- Provider payable amount
- Withdrawal requests
- Completed withdrawals
- Failed withdrawals

Use a ledger-based model rather than reconstructing balances only from mutable order rows.

## 25.2 Transaction States

```text
INITIATED
PENDING
SUCCEEDED
FAILED
REFUND_PENDING
REFUNDED
```

## 25.3 Withdrawal Workflow

```text
Provider Requests Withdrawal
        ↓
Withdrawal = PENDING
        ↓
Finance Admin Review
        ↓
Approve / Reject
        ↓
Payment Processing
        ↓
COMPLETED / FAILED
        ↓
Ledger Updated
        ↓
Notification
        ↓
Audit Log
```

## 25.4 Withdrawal Rejection

A rejection should include:

- Reason code
- Internal note
- User-visible explanation where appropriate

## 25.5 Refund Workflow

```text
Refund Request
   ↓
Validate Original Payment
   ↓
Check Refund Eligibility
   ↓
Approve Refund
   ↓
Create Refund Transaction
   ↓
Update Order / Booking Financial State
   ↓
Notify User
   ↓
Audit
```

Avoid manually editing balances without creating corresponding ledger entries.

---

# 26. Payment Reconciliation

Admin should have a reconciliation screen.

Concept:

```text
Platform Order
      ↕
Payment Provider Reference
      ↕
Internal Transaction
      ↕
Ledger Entry
```

Potential mismatch examples:

- Payment provider says paid but TechCare says pending
- Refund succeeded externally but internal state remains pending
- Duplicate webhook
- Missing webhook

Admin should be able to open a reconciliation case rather than manually inventing a payment state.

---

# 27. Provider Earnings Administration

For Doctors, Nurses, Pharmacies, and Laboratories where financial earnings apply:

Admin can review:

- Gross amount
- Platform fee if applicable
- Refunds
- Adjustments
- Net payable
- Paid amount
- Outstanding amount

Historical financial snapshots must not change because current prices or service configuration change.

---

# 28. Location and Service Area Administration

Admin may need to validate provider operational location data.

Admin view may contain:

- City / area
- Service coverage
- Approximate coordinates when necessary
- Availability status

Do not expose precise patient location to unrelated admins.

For sensitive location information:

```text
Need-to-Know Access
      ↓
Permission Check
      ↓
Purpose Validation
      ↓
Show Minimum Required Data
```

---

# 29. Account Verification Management

The platform can have different verification concepts:

```text
Identity Verification
Professional Verification
Organization Verification
Contact Verification
```

These should be modeled independently when possible.

Example:

```text
Doctor
 ├── Identity = VERIFIED
 ├── Professional = VERIFIED
 └── PublicProfile = PUBLISHED
```

Do not infer one verification state from another unless the business rule explicitly says so.

---

# 30. Document Management for Verification

Admin can:

- Open document metadata
- Preview authorized document
- Mark as accepted
- Mark as invalid
- Request re-upload
- Record reason

Document states:

```text
UPLOADED
UNDER_REVIEW
ACCEPTED
REJECTED
EXPIRED
REPLACED
```

## Security Rules

Documents should:

- Not be public URLs
- Use authorization checks
- Prefer short-lived secure access URLs if external object storage is used
- Be access-logged when sensitive
- Support revocation/expiration when appropriate

---

# 31. Provider Expired Document Monitoring

If professional documents have expiry dates:

```text
Document Expiry Date Approaching
       ↓
Create Alert
       ↓
Notify Provider
       ↓
Provider Uploads New Document
       ↓
Admin Review
       ↓
Update Verification State
```

Admin dashboard may have a queue:

```text
Expiring Soon
Expired
Awaiting Renewal
```

---

# 32. Provider Availability Monitoring

Admin can inspect high-level availability states.

Example:

```text
AVAILABLE
BUSY
OFFLINE
SUSPENDED
```

The Admin should not arbitrarily rewrite provider working hours unless the product explicitly provides an administrative override.

---

# 33. System Status and Operational Monitoring

Admin dashboard should show system health at a high level.

Examples:

- API health
- Background jobs health
- Notification queue health
- Payment webhook failures
- Database connectivity status
- Failed integration events

The Admin portal is not a replacement for infrastructure observability tools, but it should surface business-critical failures.

---

# 34. Background Job Monitoring

TechCare may depend on background workers for:

- Request expiration
- Notifications
- Reminder messages
- Verification reminders
- Expiring document alerts
- Payment reconciliation
- Blood request expiration
- Reward processing
- Report generation

Admin can see job states:

```text
QUEUED
RUNNING
COMPLETED
FAILED
RETRYING
CANCELLED
```

## 34.1 Failed Job View

Show:

- Job type
- Created time
- Started time
- Last retry time
- Retry count
- High-level failure reason

Avoid dumping secrets into the admin screen.

---

# 35. Blood Request Monitoring

Admin blood dashboard:

```text
Open Requests
Matched Requests
Accepted Donors
Fulfilled Requests
Expired Requests
Cancelled Requests
Flagged Requests
```

Admin may investigate suspicious requests.

A normal Admin operation should not manually mark a blood donation as medically completed without the appropriate confirmation event from the authorized donation workflow.

---

# 36. Blood Donor Rewards Administration

The reward system should use a ledger.

Example:

```text
RewardTransaction
- Id
- DonorId
- Type
- Points
- ReferenceType
- ReferenceId
- Description
- CreatedAt
```

Possible reward events:

- Verified donation completed
- Approved campaign reward
- Administrative adjustment

## 36.1 Manual Adjustment

Only authorized Admins can adjust points.

Workflow:

```text
Open Donor
   ↓
Open Reward Ledger
   ↓
Create Adjustment
   ↓
Enter Reason
   ↓
Confirm
   ↓
Ledger Entry
   ↓
Audit Log
```

Never overwrite the donor's total points directly.

---

# 37. Content and System Configuration

Super Admin may manage non-clinical settings.

Examples:

- Platform name
- Support contact information
- Notification settings
- Service availability flags
- Reward configuration
- Operational thresholds
- Search ranking toggles
- Maintenance banners

Each configuration change should record:

- Who changed it
- Previous value
- New value
- When
- Reason when sensitive

---

# 38. Search and Ranking Controls

The platform may surface:

- Top-rated doctors
- Nearby doctors
- Available nurses
- Nearby pharmacies
- Nearby laboratories

Admin may configure ranking signals but should preserve explainable business rules.

Possible ranking inputs:

```text
Availability
Distance
Rating
Number of completed services
Verification status
```

Do not let an unverified provider rank above verified providers if verification is a required platform rule.

---

# 39. User Search and Provider Search Moderation

Admin must be able to remove from public discovery:

- Suspended provider
- Unverified provider
- Deactivated pharmacy
- Suspended laboratory
- Deactivated donor

Example:

```text
Provider Status != VERIFIED
       ↓
Not visible in public provider discovery
```

---

# 40. Data Privacy Administration

Admin access to sensitive data should be intentionally designed.

## 40.1 Data Classification Example

### Low Sensitivity

- Public provider name
- Specialty
- Public service description
- Public rating

### Medium Sensitivity

- Phone
- Email
- Operational address
- Transaction metadata

### High Sensitivity

- Patient medical history
- Allergies
- Medications
- Clinical notes
- Laboratory results
- Prescriptions
- Sensitive identity documents
- Precise location

High-sensitivity data requires stricter permissions and audit trails.

---

# 41. Sensitive Data Access Workflow

When an Admin needs high-sensitivity data:

```text
Admin Opens Protected Resource
       ↓
Permission Check
       ↓
Purpose / Case Context Check if implemented
       ↓
Audit Access Event
       ↓
Show Minimum Required Data
```

Do not log the entire sensitive document or clinical payload in plain text inside the audit log.

---

# 42. Audit Log Workflow

Audit log is one of the most important Admin capabilities.

## 42.1 Audit Event Fields

```text
AuditLog
- Id
- ActorType
- ActorId
- Action
- ResourceType
- ResourceId
- Timestamp
- IPAddress if collected
- UserAgent if collected
- CorrelationId
- Result
- Reason
- Metadata summary
```

## 42.2 Example

```json
{
  "action": "PROVIDER_VERIFICATION_APPROVED",
  "resourceType": "Doctor",
  "resourceId": "doctor-123",
  "actorId": "admin-17",
  "result": "SUCCESS",
  "timestamp": "..."
}
```

## 42.3 Audit Log Rules

Audit logs should be append-oriented.

Admin users should not be able to silently erase their own administrative history.

---

# 43. Security Event Monitoring

Security-related admin events may include:

- Failed admin login
- Repeated failed user logins
- Suspicious account creation
- Permission changes
- Session revocation
- Multiple unusual password resets
- Repeated abusive requests
- Access to sensitive documents
- Repeated authorization failures

Security dashboard can aggregate events by severity:

```text
INFO
WARNING
HIGH
CRITICAL
```

---

# 44. Role and Permission Management

## 44.1 Permission Naming

Recommended pattern:

```text
users.read
users.suspend
users.restore
providers.verify
providers.reject
complaints.read
complaints.update
finance.read
finance.refund
withdrawals.approve
catalog.medicine.manage
catalog.test.manage
settings.manage
admins.manage
```

## 44.2 Permission Assignment

```text
Admin User
    ↓
Admin Role(s)
    ↓
Permissions
```

Avoid hard-coding every permission directly into UI buttons.

The backend remains the final authority.

---

# 45. Admin User Management

Super Admin can manage other Admin users.

## Create Admin

```text
Super Admin
   ↓
Create Admin Account
   ↓
Assign Role
   ↓
Assign Permissions where supported
   ↓
Enable / Disable MFA requirement
   ↓
Send Activation Flow
```

## Disable Admin

```text
Admin Active
   ↓
Disable Account
   ↓
Revoke Sessions
   ↓
Audit
```

Historical audit records must remain intact.

---

# 46. Admin Permission Change Safety

Changing permissions is a privileged operation.

Recommended flow:

```text
Open Admin
   ↓
Select Role / Permissions
   ↓
Review Current Permissions
   ↓
Change Permissions
   ↓
Confirmation Screen
   ↓
Save
   ↓
Audit
   ↓
Revoke Existing Tokens/Sessions if required
```

For highly sensitive systems, a two-person approval flow may be added later.

---

# 47. Global User Status Model

Recommended account statuses:

```text
ACTIVE
PENDING_VERIFICATION
NEEDS_ACTION
SUSPENDED
DEACTIVATED
```

Do not overload one status field to represent every possible business state.

Example:

```text
AccountStatus
VerificationStatus
ProviderStatus
```

These are separate concerns.

---

# 48. Admin API Design

Base path:

```text
/api/v1/admin
```

## Dashboard

```http
GET /api/v1/admin/dashboard/summary
GET /api/v1/admin/dashboard/alerts
```

## Users

```http
GET /api/v1/admin/users
GET /api/v1/admin/users/{id}
POST /api/v1/admin/users/{id}/suspend
POST /api/v1/admin/users/{id}/restore
```

## Verification

```http
GET /api/v1/admin/verifications
GET /api/v1/admin/verifications/{id}
POST /api/v1/admin/verifications/{id}/approve
POST /api/v1/admin/verifications/{id}/reject
POST /api/v1/admin/verifications/{id}/request-changes
```

## Complaints

```http
GET /api/v1/admin/complaints
GET /api/v1/admin/complaints/{id}
POST /api/v1/admin/complaints/{id}/assign
POST /api/v1/admin/complaints/{id}/status
POST /api/v1/admin/complaints/{id}/notes
POST /api/v1/admin/complaints/{id}/resolve
```

## Finance

```http
GET /api/v1/admin/finance/transactions
GET /api/v1/admin/finance/withdrawals
POST /api/v1/admin/finance/withdrawals/{id}/approve
POST /api/v1/admin/finance/withdrawals/{id}/reject
POST /api/v1/admin/finance/refunds
```

## Catalog

```http
GET /api/v1/admin/catalog/medicines
POST /api/v1/admin/catalog/medicines
PUT /api/v1/admin/catalog/medicines/{id}
POST /api/v1/admin/catalog/medicines/{id}/deactivate
```

```http
GET /api/v1/admin/catalog/tests
POST /api/v1/admin/catalog/tests
PUT /api/v1/admin/catalog/tests/{id}
POST /api/v1/admin/catalog/tests/{id}/deactivate
```

## Audit

```http
GET /api/v1/admin/audit-logs
GET /api/v1/admin/security-events
```

---

# 49. Standard API Response Pattern

Success:

```json
{
  "success": true,
  "data": {},
  "message": "Operation completed successfully"
}
```

Validation error:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Some fields are invalid",
    "details": {}
  }
}
```

Authorization error:

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to perform this action"
  }
}
```

---

# 50. Pagination and Filtering

Admin lists should use server-side pagination.

Example:

```http
GET /api/v1/admin/users?page=1&pageSize=20&status=ACTIVE&search=mostafa
```

Response:

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalCount": 1250,
  "totalPages": 63
}
```

Do not load thousands of records into the browser unnecessarily.

---

# 51. Admin Frontend Information Architecture

Recommended React / frontend admin layout:

```text
Admin Portal
│
├── Dashboard
├── Users
│   ├── Patients
│   ├── Doctors
│   ├── Nurses
│   ├── Pharmacists
│   └── Donors
│
├── Organizations
│   ├── Pharmacies
│   └── Laboratories
│
├── Verification
│   ├── Doctors
│   ├── Nurses
│   ├── Pharmacies
│   └── Laboratories
│
├── Healthcare Operations
│   ├── Appointments
│   ├── Pharmacy Orders
│   ├── Laboratory Bookings
│   ├── Samples
│   └── Blood Requests
│
├── Complaints & Moderation
├── Finance
│   ├── Transactions
│   ├── Refunds
│   ├── Withdrawals
│   └── Reconciliation
│
├── Catalogs
│   ├── Medicines
│   ├── Diagnostic Tests
│   └── Services
│
├── Notifications
├── Reports
├── Security
│   ├── Audit Logs
│   └── Security Events
│
└── Settings
```

---

# 52. Admin Navigation Rules

Menus should be generated according to permissions.

Example:

```text
Finance Admin
   ↓
Finance menu visible
Users management menu partially visible
Clinical data menu hidden
Admin management hidden
```

Hidden UI is not security by itself.

The backend must still enforce authorization.

---

# 53. Verification UI Workflow

Suggested page structure:

```text
Verification List
       ↓
Verification Details
       ↓
Applicant Summary
       ↓
Document Viewer
       ↓
Verification History
       ↓
Decision Panel
```

Decision buttons:

```text
Approve
Request Changes
Reject
```

Buttons appear only when the current Admin has the required permission.

---

# 54. Complaint UI Workflow

```text
Complaint Queue
      ↓
Complaint Details
      ↓
Related Entity
      ↓
Conversation / Evidence
      ↓
Assign
      ↓
Change Status
      ↓
Add Note
      ↓
Resolve
```

Use confirmation before destructive or final actions.

---

# 55. Financial UI Workflow

### Withdrawal List

Columns:

- Withdrawal ID
- Provider
- Amount
- Status
- Requested At
- Review At
- Reviewer

### Withdrawal Details

Show:

- Earnings summary
- Withdrawal amount
- Previous withdrawals
- Transaction references
- Validation checks
- Audit history

---

# 56. Database Design — Core Admin Entities

The Admin module should integrate with the existing domain rather than duplicate entire domain tables.

Recommended entities:

```text
AdminUser
AdminRole
AdminPermission
AdminRolePermission
AdminUserRole
AuditLog
SecurityEvent
Complaint
ComplaintMessage
VerificationRequest
VerificationDocument
AdminAction
SystemSetting
NotificationTemplate
```

Operational integrations read domain entities such as:

```text
User
Doctor
Nurse
Pharmacy
Laboratory
Donor
Appointment
Prescription
PharmacyOrder
LabBooking
Sample
BloodRequest
PaymentTransaction
WithdrawalRequest
RewardTransaction
```

---

# 57. AdminUser Entity

Example fields:

```text
Id
UserId
EmployeeCode
Status
LastLoginAt
MfaEnabled
CreatedAt
UpdatedAt
```

The Admin should still map to the common authentication identity where possible.

---

# 58. AdminRole Entity

```text
Id
Name
Description
IsActive
CreatedAt
UpdatedAt
```

Examples:

```text
SuperAdmin
VerificationAdmin
SupportAdmin
FinanceAdmin
OperationsAdmin
```

---

# 59. Complaint Entity

Possible fields:

```text
Id
ReporterUserId
TargetType
TargetId
OrderId nullable
AppointmentId nullable
Category
Priority
Status
Subject
Description
AssignedAdminId nullable
CreatedAt
UpdatedAt
ResolvedAt nullable
```

---

# 60. VerificationRequest Entity

```text
Id
ApplicantUserId
ApplicantType
Status
SubmittedAt
ReviewedAt nullable
ReviewedByAdminId nullable
DecisionReasonCode nullable
DecisionNote nullable
CreatedAt
UpdatedAt
```

---

# 61. AuditLog Entity

```text
Id
ActorUserId
ActorType
Action
ResourceType
ResourceId
Result
Reason
CorrelationId
IpAddress nullable
UserAgent nullable
CreatedAt
```

Avoid storing huge before/after payloads by default.

Use structured metadata and carefully redact sensitive fields.

---

# 62. Admin State Machines

## 62.1 Verification

```text
DRAFT
  ↓
SUBMITTED
  ↓
UNDER_REVIEW
  ├── NEEDS_CHANGES → RESUBMITTED → UNDER_REVIEW
  ├── APPROVED
  └── REJECTED
```

## 62.2 Complaint

```text
OPEN
  ↓
ASSIGNED
  ↓
UNDER_REVIEW
  ├── WAITING_FOR_USER
  ├── WAITING_FOR_PROVIDER
  └── ESCALATED
          ↓
      RESOLVED
          ↓
        CLOSED
```

## 62.3 Withdrawal

```text
REQUESTED
   ↓
UNDER_REVIEW
   ├── REJECTED
   └── APPROVED
          ↓
      PROCESSING
          ├── FAILED
          └── COMPLETED
```

## 62.4 User Suspension

```text
ACTIVE
  ↓
SUSPENDED
  ↓
RESTORED
```

---

# 63. Double-Action / Concurrency Protection

Two Admins may attempt the same action simultaneously.

Example:

```text
Admin A opens Withdrawal #100
Admin B opens Withdrawal #100
Admin A approves
Admin B tries approve
```

The second request must fail safely because the withdrawal is already no longer reviewable.

Use transactional checks / optimistic concurrency where appropriate.

Example concept:

```text
WHERE Id = @id AND Status = 'UNDER_REVIEW'
```

Then verify affected rows count.

---

# 64. Idempotency

Administrative endpoints that trigger external or financial side effects should be protected against duplicate submissions.

Examples:

- Refund
- Notification campaign
- Withdrawal approval
- Reward adjustment
- Account restoration action where appropriate

Example:

```http
Idempotency-Key: 8a7f...
```

---

# 65. Manual Override Rules

Manual admin overrides should be rare.

Allowed examples:

- Suspend fraudulent user
- Resolve support issue
- Correct a verified operational state through an explicit workflow
- Reconcile payment
- Approve/reject verification

Avoid:

```text
Admin changes database row directly
```

Prefer:

```text
Admin Action
   ↓
Application Service
   ↓
Validation
   ↓
Transaction
   ↓
Domain Update
   ↓
Audit
   ↓
Notification if required
```

---

# 66. Transactional Safety

Actions that change multiple related records should use database transactions.

Example withdrawal approval:

```text
Begin Transaction
   ↓
Validate status
   ↓
Create financial entry
   ↓
Update withdrawal status
   ↓
Update provider payable state if applicable
   ↓
Create audit event
   ↓
Commit
```

If any critical step fails, rollback the transaction.

---

# 67. Soft Delete Strategy

Do not hard-delete historical healthcare or financial records merely to hide them from UI.

Use states such as:

```text
ACTIVE
INACTIVE
ARCHIVED
SUSPENDED
DEACTIVATED
```

Hard deletion should be exceptional and governed by explicit data lifecycle rules.

---

# 68. Reporting Workflow

Admin should have reports that help operations without requiring database access.

Reports may include:

## User Reports

- Registrations over time
- Active users
- Suspended accounts
- Role distribution

## Provider Reports

- Verified doctors
- Verified nurses
- Active pharmacies
- Active laboratories
- Verification turnaround metrics

## Operational Reports

- Appointment completion
- Appointment cancellation
- Expired requests
- Pharmacy orders
- Lab bookings
- Failed collections
- Blood requests

## Support Reports

- Open complaints
- Resolution times
- Complaint categories
- Escalations

## Financial Reports

- Completed payments
- Refunds
- Provider earnings
- Withdrawals
- Failed financial operations

---

# 69. Export Workflow

Reports may support CSV export.

Example:

```text
Apply Filters
    ↓
Generate Export Job
    ↓
Background Processing
    ↓
Generate File
    ↓
Secure Download
```

Large exports should not block the HTTP request.

---

# 70. Report Access Control

Finance reports should not automatically be visible to Support Admin.

Clinical result-related reports require additional authorization.

Every sensitive export should be auditable.

---

# 71. Admin Notification Center

Admin notification center can contain:

```text
New Verification
New Complaint
Critical Security Event
Payment Failure
Withdrawal Request
System Alert
Expired Document
Failed Background Job
```

Notification levels:

```text
INFO
WARNING
HIGH
CRITICAL
```

---

# 72. Admin Action Center

A useful dashboard section:

```text
MY PENDING ACTIONS
------------------
12 Provider Verifications
5 Complaints
3 Withdrawals
2 Security Alerts
4 Expiring Documents
```

Clicking each item opens the corresponding queue.

---

# 73. Assignment System

Some administrative queues should support assignment.

Example:

```text
Complaint
   ↓
Assigned to Support Admin #12
```

Fields:

```text
AssignedTo
AssignedAt
AssignedBy
```

Assignment actions should be audited.

---

# 74. Priority Management

Use priority for support/operational queues.

Example:

```text
LOW
NORMAL
HIGH
CRITICAL
```

Critical operational events should appear first.

Do not make every issue Critical.

---

# 75. SLA-Oriented Queue Design

Even without fixed project timelines, the Admin portal can track business response expectations.

Example:

```text
CreatedAt
DueAt
CurrentAge
Overdue = true/false
```

This can be used for:

- Verification queues
- Complaints
- Withdrawal review
- Operational incidents

---

# 76. Global Search

Admin may have a unified search.

Example:

```text
Search: TC-APPT-10027
```

Possible result:

```text
Appointment
Patient
Doctor
Payment
Complaint
Audit Events
```

Search should return only resources the Admin is authorized to access.

---

# 77. Timeline View

A unified timeline is useful for investigations.

Example:

```text
10:02 Patient created booking
10:04 Doctor accepted
10:05 Payment succeeded
10:16 Appointment started
10:45 Consultation completed
10:47 Prescription created
10:55 Patient submitted pharmacy order
```

The timeline should distinguish system events from human administrative actions.

---

# 78. Investigation Workflow

When an Admin receives a complex case:

```text
Case Created
   ↓
Identify Primary Entity
   ↓
Open Timeline
   ↓
Review Related Entities
   ↓
Review Audit Events
   ↓
Review Payment State if relevant
   ↓
Review Complaint Messages
   ↓
Take Action
   ↓
Document Reason
   ↓
Resolve / Escalate
```

---

# 79. Support Case Example — Appointment Problem

Scenario:

Patient says:

> "The doctor accepted but the appointment disappeared."

Admin process:

```text
Open Complaint
   ↓
Open Appointment
   ↓
Check Appointment Status
   ↓
Check Payment
   ↓
Check Provider Account
   ↓
Check Audit Log
   ↓
Check Notification Log
   ↓
Identify Failure
   ↓
Apply Approved Resolution
   ↓
Record Internal Note
   ↓
Respond to Patient
   ↓
Resolve Complaint
```

---

# 80. Support Case Example — Pharmacy Order Problem

Scenario:

Patient claims the order was charged but rejected by pharmacy.

Admin checks:

```text
Order
Payment
Pharmacy
Inventory reservation
Order state history
Refund state
Complaint
```

Possible resolution:

```text
Order Rejected
    ↓
Payment Confirmed
    ↓
Refund Required
    ↓
Create Refund
    ↓
Notify Patient
    ↓
Audit
```

---

# 81. Support Case Example — Lab Collection Failure

```text
Open Complaint
   ↓
Open Lab Booking
   ↓
Check Collection Mode
   ↓
Check Collector Assignment
   ↓
Check Collection Events
   ↓
Check Sample Status
   ↓
Determine: Retry / Reschedule / Refund
   ↓
Resolve Case
```

---

# 82. Support Case Example — Blood Request Abuse

```text
Open Report
   ↓
Review Blood Request
   ↓
Check Request History
   ↓
Check Notification Activity
   ↓
Check Duplicate Requests
   ↓
Flag / Suspend if justified
   ↓
Protect affected donors
   ↓
Audit
```

Do not expose donor private data to the requester unnecessarily.

---

# 83. Admin Security — Backend Authorization

Every controller/action must be protected.

Example concept in ASP.NET Core:

```csharp
[Authorize(Roles = "SuperAdmin,VerificationAdmin")]
[HttpPost("{id}/approve")]
public async Task<IActionResult> Approve(...)
{
    ...
}
```

For more granular systems:

```csharp
[Authorize(Policy = "Providers.Verify")]
```

Policies are preferred where permission-based authorization is required.

---

# 84. Resource-Level Security

Example:

```text
Finance.Admin
    ✅ Can read withdrawals
    ✅ Can approve withdrawal
    ❌ Cannot edit doctor diagnosis
    ❌ Cannot read unrelated patient clinical details
```

The authorization layer should reject unauthorized operations even if a malicious admin client manually calls the API.

---

# 85. File Upload Security for Admin

Admin uploads such as catalog images, evidence, or exported reports must be validated.

Validate:

- Extension
- MIME type where possible
- File size
- Storage path
- Authorization
- Malware scanning strategy if available

Never trust the filename supplied by the browser.

---

# 86. Input Validation

Validate every Admin write request.

Examples:

```text
Required fields
Valid IDs
Valid status transition
Allowed enum value
Numeric boundaries
String length
File limits
Reason requirement for sensitive action
```

---

# 87. Dangerous Admin Actions Confirmation

Actions such as:

- Suspend
- Delete/deactivate
- Approve withdrawal
- Refund
- Change permissions
- Modify system settings

should have confirmation dialogs.

For very sensitive operations, require:

```text
Type confirmation text
```

or re-authentication / MFA.

---

# 88. Status Transition Guard

Admin should not be able to move any record from any state to any state.

Bad:

```text
PUT /orders/100/status = COMPLETED
```

Better:

```text
POST /orders/100/approve
POST /orders/100/refund
POST /appointments/100/cancel
```

The application service validates the transition.

---

# 89. Domain Service Layer

Recommended .NET architecture:

```text
Controller
   ↓
Application Service / Command Handler
   ↓
Validation
   ↓
Authorization
   ↓
Domain Logic
   ↓
Repository / DbContext
   ↓
Database
```

Do not put all Admin business logic directly inside controllers.

---

# 90. Suggested ASP.NET Core Admin Module Structure

Example modular structure:

```text
Admin/
├── Controllers/
│   ├── AdminDashboardController.cs
│   ├── AdminUsersController.cs
│   ├── AdminVerificationController.cs
│   ├── AdminComplaintsController.cs
│   ├── AdminFinanceController.cs
│   ├── AdminCatalogController.cs
│   ├── AdminSecurityController.cs
│   └── AdminSettingsController.cs
│
├── Services/
│   ├── AdminDashboardService.cs
│   ├── VerificationService.cs
│   ├── ComplaintService.cs
│   ├── FinanceAdminService.cs
│   ├── CatalogService.cs
│   └── AuditService.cs
│
├── Policies/
│   └── AdminAuthorizationPolicies.cs
│
├── DTOs/
│   ├── DashboardDto.cs
│   ├── VerificationDto.cs
│   ├── ComplaintDto.cs
│   └── WithdrawalDto.cs
│
└── Validators/
    └── AdminValidators.cs
```

---

# 91. EF Core Database Considerations

Important indexes:

```text
Users.Email
Users.Phone
Users.Role
Users.Status
VerificationRequests.Status
VerificationRequests.SubmittedAt
Complaints.Status
Complaints.AssignedAdminId
Complaints.Priority
AuditLogs.ResourceType + ResourceId
AuditLogs.CreatedAt
WithdrawalRequests.Status
WithdrawalRequests.CreatedAt
SecurityEvents.CreatedAt
```

Use indexes based on actual query patterns.

---

# 92. Concurrency Tokens

For frequently reviewed resources, use an optimistic concurrency strategy.

Example:

```csharp
public byte[] RowVersion { get; set; }
```

This helps detect stale Admin pages when another Admin changes the same record.

---

# 93. Logging Strategy

Separate:

```text
Application Logs
Audit Logs
Security Logs
Financial Logs
```

Do not treat ordinary application logs as a replacement for formal audit records.

Never log:

- Passwords
- OTPs
- Access tokens
- Refresh token secrets
- Full sensitive medical records

---

# 94. Error Handling

Example error codes:

```text
ADMIN_NOT_FOUND
FORBIDDEN
RESOURCE_NOT_FOUND
INVALID_STATE_TRANSITION
VERIFICATION_ALREADY_REVIEWED
WITHDRAWAL_ALREADY_PROCESSED
REFUND_NOT_ALLOWED
CATALOG_ITEM_IN_USE
USER_ALREADY_SUSPENDED
USER_ALREADY_ACTIVE
CONCURRENCY_CONFLICT
```

Frontend should map error codes to user-friendly messages.

---

# 95. Admin Rate Limiting

Sensitive Admin endpoints may need stricter rate limits.

Examples:

- Login
- Password reset
- User bulk actions
- Notification campaigns
- Exports
- Security searches

---

# 96. Bulk Operations

Bulk operations can improve productivity but increase risk.

Examples:

- Suspend multiple flagged users
- Deactivate several catalog items
- Assign complaints to an Admin
- Export filtered records

Bulk operations should:

```text
Validate each eligible record
   ↓
Report successes / failures
   ↓
Create audit event
```

Do not silently skip failures.

---

# 97. Bulk Action Result

Example:

```json
{
  "total": 5,
  "succeeded": 4,
  "failed": 1,
  "results": [
    {
      "id": "u1",
      "status": "SUCCESS"
    },
    {
      "id": "u2",
      "status": "FAILED",
      "errorCode": "ALREADY_SUSPENDED"
    }
  ]
}
```

---

# 98. Public Profile Moderation

Admin can remove a provider from public discovery when the provider becomes:

```text
SUSPENDED
UNVERIFIED
DEACTIVATED
```

The public profile should reflect the current approved state without destroying historical relationships.

---

# 99. Provider Rating Integrity

Admin must not manually change rating averages as a shortcut.

Ratings should be derived from rating records.

If a rating is moderated:

```text
Rating = HIDDEN
   ↓
Exclude from public aggregate according to defined rule
```

Record the moderation action.

---

# 100. Notification Retry Monitoring

Admin can see failed notifications.

State example:

```text
QUEUED
SENT
FAILED
RETRYING
DELIVERED if channel provides delivery confirmation
```

Retry policy belongs to background infrastructure/application logic, not manual Admin clicking for every failure.

---

# 101. Admin Maintenance Mode

Super Admin may activate a controlled maintenance message.

Example configuration:

```text
MaintenanceEnabled = true
MaintenanceMessage = "System maintenance is in progress."
```

The system should preserve an emergency admin access path where appropriate.

---

# 102. System Setting Change Audit

Every sensitive setting change:

```text
Old Value
New Value
Admin
Timestamp
Reason
```

Example:

```text
Blood request expiry policy
Old = X
New = Y
Actor = Super Admin
```

The exact business value should live in configuration and policy documents, not be silently hard-coded in controllers.

---

# 103. Admin Reporting Filters

Common filters:

```text
Date From
Date To
Role
Status
Provider Type
Service
Area
Payment Status
Complaint Category
Verification State
```

All filters must be validated and parameterized.

---

# 104. Date/Time Handling

Store timestamps consistently, preferably in UTC at the database level.

Convert to the UI/business timezone when displaying.

Avoid building business logic from the browser's local clock.

---

# 105. Admin Localization

The admin UI may support:

- Arabic
- English

Dates, currency, status labels, error messages, and notifications should be localizable.

Enum values in the backend should remain stable machine-readable identifiers.

Example:

```text
UNDER_REVIEW
```

UI Arabic label:

```text
قيد المراجعة
```

---

# 106. Admin Accessibility

Admin portal should support:

- Keyboard navigation
- Visible focus states
- Clear labels
- Accessible tables
- Accessible dialogs
- Meaningful validation messages
- Responsive layout for laptop/tablet where practical

---

# 107. Admin Page Details

## Dashboard

- KPI cards
- Alerts
- Pending queues
- Trend summaries

## Users

- Search
- Filters
- List
- Details
- Suspend/restore

## Verification

- Queue
- Details
- Documents
- Decision
- History

## Complaints

- Queue
- Details
- Assignment
- Conversation
- Resolution

## Finance

- Transactions
- Withdrawals
- Refunds
- Reconciliation

## Catalog

- Medicines
- Tests
- Services

## Security

- Audit
- Security events
- Admin sessions

---

# 108. Admin Home Dashboard Example

```text
┌──────────────────────────────────────────────┐
│ TECHCARE ADMIN                               │
├──────────────────────────────────────────────┤
│ Users     Providers    Orders    Revenue     │
│ 52,340      1,284       8,420     ...        │
├──────────────────────────────────────────────┤
│ Pending Actions                              │
│ • 18 Doctor Verifications                    │
│ • 7 Nurse Verifications                      │
│ • 4 Pharmacy Verifications                   │
│ • 3 Lab Verifications                        │
│ • 9 Complaints                                │
│ • 5 Withdrawals                               │
├──────────────────────────────────────────────┤
│ Alerts                                        │
│ • Payment webhook failures                    │
│ • Expiring professional documents             │
└──────────────────────────────────────────────┘
```

---

# 109. Full End-to-End Scenario — Doctor Approval

```text
Doctor Registers
      ↓
Doctor Completes Profile
      ↓
Uploads Documents
      ↓
Verification Request Created
      ↓
Admin Dashboard Counter +1
      ↓
Verification Admin Opens Queue
      ↓
Opens Doctor Application
      ↓
Reviews Profile
      ↓
Reviews Documents
      ↓
All Valid
      ↓
Approve
      ↓
Doctor = VERIFIED
      ↓
Doctor Public Profile = Eligible for Discovery
      ↓
Doctor Notification Sent
      ↓
Audit Event Created
```

---

# 110. Full End-to-End Scenario — Rejected Doctor

```text
Doctor Submits
   ↓
Admin Reviews
   ↓
Missing Document
   ↓
Request Changes
   ↓
Doctor Receives Notification
   ↓
Doctor Uploads New Document
   ↓
Status = RESUBMITTED
   ↓
Admin Reviews Again
   ↓
Approve
```

---

# 111. Full End-to-End Scenario — User Suspension

```text
Complaint / Security Alert
        ↓
Case Review
        ↓
Evidence Review
        ↓
Admin Decision
        ↓
Suspend User
        ↓
Revoke Sessions
        ↓
Prevent New Actions
        ↓
Keep Historical Records
        ↓
Notify User
        ↓
Audit Log
```

---

# 112. Full End-to-End Scenario — Withdrawal

```text
Provider Completes Services
       ↓
Earnings Recorded
       ↓
Provider Requests Withdrawal
       ↓
Finance Queue
       ↓
Finance Admin Opens Request
       ↓
Checks Balance + Existing Transactions
       ↓
Approve
       ↓
Create Processing Record
       ↓
External / Internal Processing
       ↓
Completed
       ↓
Ledger Updated
       ↓
Provider Notified
       ↓
Audit Logged
```

---

# 113. Full End-to-End Scenario — Complaint Resolution

```text
Patient Reports Problem
      ↓
Complaint Created
      ↓
Support Admin Queue
      ↓
Assign Admin
      ↓
Open Related Appointment / Order
      ↓
Read Timeline
      ↓
Contact Relevant Party
      ↓
Investigate
      ↓
Resolution Decision
      ↓
Optional Refund / Cancellation / Escalation
      ↓
Respond to Patient
      ↓
Resolve Complaint
      ↓
Close Case
      ↓
Audit
```

---

# 114. Full End-to-End Scenario — Pharmacy Verification + First Order

```text
Pharmacy Registers
      ↓
Documents Submitted
      ↓
Verification Admin Approves
      ↓
Pharmacy Becomes Visible
      ↓
Pharmacy Adds Inventory
      ↓
Patient Places Order
      ↓
Order Appears in Pharmacy Dashboard
      ↓
Operational Monitoring Available to Admin
      ↓
Order Completed
      ↓
Financial Record Created
```

Admin does not need to interfere with a healthy order.

---

# 115. Full End-to-End Scenario — Lab Result Issue

```text
Patient Books Test
      ↓
Lab Accepts
      ↓
Sample Collected
      ↓
Sample Received
      ↓
Result Prepared
      ↓
Validation
      ↓
Result Published
      ↓
Patient Receives Result
```

If complaint arises:

```text
Complaint
   ↓
Admin Opens Booking
   ↓
Checks Sample Timeline
   ↓
Checks Result Publication Events
   ↓
Escalates to Authorized Lab Professional if Clinical Review Needed
```

Admin should not independently make a medical interpretation of laboratory results.

---

# 116. Full End-to-End Scenario — Blood Request

```text
Patient Creates Blood Request
      ↓
Request Becomes OPEN
      ↓
Matching Logic Finds Potential Donors
      ↓
Notifications Sent
      ↓
Donors Respond
      ↓
Donation Coordination
      ↓
Authorized Donation Event
      ↓
Request Fulfillment
      ↓
Donor Reward Event
      ↓
History Updated
```

Admin can monitor and intervene in platform operations, abuse, or account issues without turning the system into a blood marketplace.

---

# 117. Admin Notifications by Module

| Event | Target |
|---|---|
| New provider verification | Verification Admin |
| New complaint | Support Admin |
| Withdrawal request | Finance Admin |
| Failed payment reconciliation | Finance/Ops |
| Critical security event | Super Admin |
| Expiring provider document | Verification Admin |
| Failed notification batch | Operations Admin |
| Suspicious blood activity | Support/Ops |
| Failed lab workflow | Operations Admin |

---

# 118. Security and Privacy Test Matrix

| Scenario | Expected Result |
|---|---|
| Support Admin accesses finance approval endpoint | 403 |
| Finance Admin edits diagnosis | 403 |
| Unauthenticated user calls admin API | 401 |
| Suspended Admin calls API with old token | Access revoked / rejected according to session strategy |
| Admin opens unauthorized sensitive document | 403 |
| User attempts Admin endpoint | 403 |
| Invalid status transition | 400 / domain error |
| Duplicate approval request | Safe rejection / idempotent outcome |
| Concurrent verification decision | One successful state transition |

---

# 119. Verification Test Cases

### TC-ADM-VER-001

Doctor submits all required documents.

Expected:

```text
Status = SUBMITTED
```

### TC-ADM-VER-002

Admin approves application.

Expected:

```text
Status = APPROVED
Provider = VERIFIED
Audit = Created
Notification = Sent
```

### TC-ADM-VER-003

Admin requests changes.

Expected:

```text
Status = NEEDS_CHANGES
Reason = Required
Notification = Sent
```

### TC-ADM-VER-004

Admin tries to approve already approved provider.

Expected:

```text
INVALID_STATE_TRANSITION
```

---

# 120. User Management Test Cases

### TC-ADM-USR-001

Suspend active user.

Expected:

```text
User = SUSPENDED
Sessions revoked where configured
Audit created
Notification generated
```

### TC-ADM-USR-002

Suspend already suspended user.

Expected:

```text
Safe rejection
No duplicate audit transition
```

### TC-ADM-USR-003

Restore suspended user.

Expected:

```text
User = ACTIVE
Audit created
```

---

# 121. Complaint Test Cases

### TC-ADM-CMP-001

New complaint appears in queue.

Expected:

```text
OPEN
```

### TC-ADM-CMP-002

Support Admin assigns complaint.

Expected:

```text
AssignedAdminId updated
Audit created
```

### TC-ADM-CMP-003

Complaint resolved without required permission.

Expected:

```text
403
```

---

# 122. Finance Test Cases

### TC-ADM-FIN-001

Valid withdrawal approval.

Expected:

```text
Transaction consistent
Withdrawal status updated
Audit created
```

### TC-ADM-FIN-002

Two Admins approve same withdrawal simultaneously.

Expected:

```text
One succeeds
Other receives state/concurrency error
No duplicate payout
```

### TC-ADM-FIN-003

Duplicate refund request.

Expected:

```text
No duplicate refund
```

---

# 123. Catalog Test Cases

### TC-ADM-CAT-001

Create active medicine.

Expected:

```text
Medicine searchable by pharmacies
```

### TC-ADM-CAT-002

Deactivate medicine used historically.

Expected:

```text
New inventory configuration blocked or controlled
Historical orders unchanged
```

### TC-ADM-CAT-003

Edit medicine display information.

Expected:

```text
Current catalog changes
Historical transaction snapshots remain unchanged
```

---

# 124. Audit Test Cases

Every critical Admin action should create exactly the expected audit event.

Examples:

```text
Approve provider
Reject provider
Suspend user
Restore user
Approve withdrawal
Reject withdrawal
Issue refund
Edit setting
Change permissions
```

Test that audit data is:

- Correct
- Timestamped
- Linked to actor
- Linked to resource
- Not leaking secrets

---

# 125. Security Test Cases

### Authentication

- Invalid password
- Repeated failed login
- Expired access token
- Revoked session
- Invalid refresh token

### Authorization

- Wrong role
- Missing permission
- Direct API access with hidden button
- IDOR attempts against user/resource IDs

### Input

- SQL injection payloads
- XSS payloads
- Oversized request
- Invalid file upload
- Malformed IDs

---

# 126. IDOR Protection Example

Bad:

```http
GET /api/v1/admin/users/12345
```

with no backend permission/resource validation.

The server must still verify that the caller has permission to retrieve this resource.

Never rely on UI hiding as the security layer.

---

# 127. Bulk Operation Security Test

When bulk suspending users:

```text
User A -> allowed
User B -> not allowed
User C -> already suspended
```

The response should clearly identify:

```text
A = success
B = forbidden
C = invalid state
```

---

# 128. Performance Rules

Admin tables should use:

- Pagination
- Server-side filtering
- Server-side sorting
- Projection/DTO queries
- Appropriate indexes
- Async database queries

Avoid loading full entities with all navigation properties when only list metadata is needed.

---

# 129. N+1 Query Prevention

For dashboard/list pages, avoid:

```text
1 query for users
+ 1 query per user for role
+ 1 query per user for status
+ 1 query per user for complaints
```

Prefer projection or batch queries.

---

# 130. Caching Strategy

Some Admin data can be cached carefully:

- Static catalog metadata
- Service definitions
- Permission definitions
- Dashboard aggregates for short intervals

Do not blindly cache:

- Sensitive user-specific data
- Live financial status
- Current security decisions

---

# 131. Transaction Integrity Rules

Any action affecting money must maintain:

```text
Business Entity
    +
Payment Transaction
    +
Ledger / Financial Record
    +
Audit
```

A partially completed money operation is a critical defect.

---

# 132. Clinical Data Safety Rules for Admin

Admin is not the clinician.

Therefore:

```text
Admin
   ├── operational access
   ├── support access
   ├── compliance/verification access
   └── controlled sensitive-data access when required
```

But:

```text
Admin ≠ Doctor
Admin ≠ Laboratory Professional
Admin ≠ Nurse
```

Do not grant clinical privileges simply because a user has Admin role.

---

# 133. Emergency Future Integration Boundary

Emergency response is future scope unless explicitly included in the prototype.

If added later:

```text
Emergency Event
      ↓
Admin Monitoring
      ↓
Authorized Emergency Workflow
      ↓
Hospital / EMS Integration
```

Admin should not invent clinical dispatch behavior directly inside the ordinary user-management module.

---

# 134. Suggested Admin DTOs

```text
AdminDashboardSummaryDto
AdminAlertDto
AdminUserListItemDto
AdminUserDetailsDto
VerificationQueueItemDto
VerificationDetailsDto
ComplaintListItemDto
ComplaintDetailsDto
WithdrawalListItemDto
WithdrawalDetailsDto
FinancialTransactionDto
AuditLogDto
SecurityEventDto
CatalogItemDto
SystemSettingDto
```

---

# 135. Suggested Commands / Use Cases

```text
LoginAdmin
LogoutAdmin
SuspendUser
RestoreUser
ApproveVerification
RejectVerification
RequestVerificationChanges
AssignComplaint
ResolveComplaint
ApproveWithdrawal
RejectWithdrawal
CreateRefund
AdjustRewardPoints
CreateMedicine
UpdateMedicine
DeactivateMedicine
CreateDiagnosticTest
UpdateDiagnosticTest
DeactivateDiagnosticTest
CreateService
UpdateService
DeactivateService
CreateAdminUser
DisableAdminUser
AssignAdminRole
RevokeAdminRole
UpdateSystemSetting
```

---

# 136. Suggested Queries

```text
GetDashboardSummary
GetDashboardAlerts
SearchUsers
GetUserDetails
GetVerificationQueue
GetVerificationDetails
GetComplaintQueue
GetComplaintDetails
GetAppointmentOperations
GetPharmacyOrders
GetLabBookings
GetBloodRequests
GetTransactions
GetWithdrawals
GetAuditLogs
GetSecurityEvents
GetMedicineCatalog
GetDiagnosticTestCatalog
GetServices
GetSystemSettings
```

---

# 137. Complete Admin Workflow — Daily Operations

```text
Admin Login
   ↓
Dashboard
   ↓
Review Alerts
   ↓
Review Pending Queue
   ├── Verification
   ├── Complaints
   ├── Finance
   └── Operations
   ↓
Handle Tasks
   ↓
Review Critical Events
   ↓
Check Failed Background Jobs
   ↓
Review Escalations
   ↓
Logout
```

---

# 138. Complete Admin Workflow — New Doctor

```text
Doctor Registration
      ↓
Verification Request
      ↓
Queue
      ↓
Review
      ↓
Decision
      ├── Needs Changes → Doctor Updates
      │                     ↓
      │                  Resubmission
      │                     ↓
      │                   Review
      │
      ├── Reject → Applicant Notified
      │
      └── Approve
             ↓
         VERIFIED
             ↓
        Public Profile
             ↓
       Searchable/Bookable
```

---

# 139. Complete Admin Workflow — Provider Suspension

```text
Complaint / Security Event
       ↓
Investigation
       ↓
Evidence Review
       ↓
Admin Decision
       ↓
Suspend Provider
       ↓
Remove from New Discovery
       ↓
Prevent New Bookings / Orders as defined
       ↓
Handle Existing Commitments
       ↓
Notify Provider
       ↓
Audit
```

Existing appointments/orders must follow explicit cancellation/refund policy.

---

# 140. Complete Admin Workflow — Payment Failure

```text
Payment Failure Alert
      ↓
Open Reconciliation
      ↓
Check Internal Transaction
      ↓
Check Provider Payment Reference
      ↓
Check Order / Appointment
      ↓
Determine Actual State
      ↓
Trigger Approved Reconciliation Action
      ↓
Audit
      ↓
Notify Affected User
```

---

# 141. Admin Operational State Matrix

| Module | Admin Can Monitor | Admin Can Act | Sensitive Data |
|---|---|---|---|
| Patient | Yes | Account/support actions | High |
| Doctor | Yes | Verification/suspension | High |
| Nurse | Yes | Verification/suspension | Medium/High |
| Pharmacy | Yes | Verification/suspension | Medium |
| Laboratory | Yes | Verification/operations | High |
| Blood Donor | Yes | Account/moderation | High |
| Appointment | Yes | Controlled intervention | Medium/High |
| Pharmacy Order | Yes | Support/refund workflows | Medium/High |
| Lab Booking | Yes | Support/operations | High |
| Complaints | Yes | Full case management | Medium/High |
| Finance | Yes | Finance actions by permission | High |
| Catalog | Yes | CRUD/deactivate | Low |
| Audit | Yes | Usually read-only | High |

---

# 142. What Admin Must NOT Do Directly

Admin should not:

- Read or store passwords
- Manually change password hashes
- Create fake patient clinical records
- Write a diagnosis as an operational shortcut
- Modify laboratory result values casually
- Alter historical financial totals directly
- Bypass payment controls
- Circumvent authorization checks
- Delete audit history silently
- Expose patient private data to unrelated staff
- Treat blood as a commercial marketplace item

---

# 143. Admin Definition of Done

The Admin module is considered functionally complete when:

## Authentication

- Admin login works
- Admin logout works
- Token/session revocation works
- Role/permission authorization works
- Sensitive Admin access is protected

## Users

- Search users
- View appropriate user information
- Suspend users
- Restore users
- Audit actions

## Verification

- Doctor verification works
- Nurse verification works
- Pharmacy verification works
- Laboratory verification works
- Request-changes flow works
- Verification history exists

## Operations

- Appointments monitored
- Pharmacy orders monitored
- Laboratory bookings monitored
- Samples monitored
- Blood requests monitored

## Support

- Complaints searchable
- Assignment works
- Status workflow works
- Resolution works
- Escalation works

## Finance

- Transactions visible
- Withdrawals reviewable
- Refund workflow protected
- Reconciliation supported
- Ledger integrity tested

## Catalogs

- Medicines manageable
- Diagnostic tests manageable
- Services manageable
- Historical references preserved

## Security

- Audit logs implemented
- Security events visible
- Sensitive access protected
- IDOR tests pass
- Admin permission tests pass

## Quality

- Unit tests
- Integration tests
- Authorization tests
- Concurrency tests
- Financial tests
- UI validation tests
- Error handling tests

---

# 144. Recommended MVP Boundary

## Include in Prototype

```text
Admin Authentication
RBAC / Permissions
Dashboard
User Management
Doctor Verification
Nurse Verification
Pharmacy Verification
Laboratory Verification
Complaint Management
Appointment Monitoring
Pharmacy Order Monitoring
Laboratory Booking Monitoring
Blood Request Monitoring
Medicine Catalog
Diagnostic Test Catalog
Service Catalog
Financial Monitoring
Withdrawal Review
Refund Workflow
Audit Logs
Basic Security Events
Basic Reports
Notifications
```

## Future Enhancements

```text
Advanced BI dashboards
Automated anomaly detection
Sophisticated fraud scoring
Multi-level approval chains
Advanced case management
Advanced financial reconciliation engine
Automated document verification
AI-assisted support triage
Advanced operational forecasting
External emergency integrations
Multi-tenant enterprise administration
```

---

# 145. Admin Workflow Master Diagram

```text
                              ┌───────────────────────┐
                              │      ADMIN LOGIN      │
                              └──────────┬────────────┘
                                         ↓
                              ┌───────────────────────┐
                              │ AUTH + MFA + RBAC     │
                              └──────────┬────────────┘
                                         ↓
                              ┌───────────────────────┐
                              │      DASHBOARD        │
                              └──────────┬────────────┘
                                         ↓
        ┌──────────────────────────────────────────────────────────┐
        │                      ADMIN OPERATIONS                     │
        └──────────────────────────────────────────────────────────┘
             ↓               ↓                ↓            ↓
        ┌─────────┐     ┌────────────┐   ┌─────────┐  ┌───────────┐
        │ USERS   │     │ VERIFICATION│   │ SUPPORT │  │ FINANCE   │
        └────┬────┘     └─────┬──────┘   └────┬────┘  └─────┬─────┘
             ↓                ↓               ↓             ↓
        Patients/        Doctors/Nurses/   Complaints/   Payments/
        Providers         Pharmacy/Lab     Moderation    Refunds/
                                                           Withdrawals
             │                │               │             │
             └────────────────┴───────┬───────┴─────────────┘
                                      ↓
                              ┌───────────────┐
                              │  OPERATIONS   │
                              └───────┬───────┘
                                      ↓
                     Appointments / Orders / Labs /
                           Blood Requests
                                      ↓
                              ┌───────────────┐
                              │ CATALOGS      │
                              └───────┬───────┘
                                      ↓
                           Medicines / Tests /
                                Services
                                      ↓
                         ┌────────────────────────┐
                         │ SECURITY + AUDIT       │
                         └───────────┬────────────┘
                                     ↓
                              EVERY SENSITIVE
                              ADMIN ACTION LOGGED
```

---

# 146. Final Admin Philosophy

The Admin module should be treated as a **control plane** for TechCare rather than a giant CRUD page.

The correct mental model is:

```text
                    TECHCARE PLATFORM
                           │
                           ↓
                    ┌─────────────┐
                    │    ADMIN    │
                    │ CONTROL     │
                    │    PLANE    │
                    └──────┬──────┘
                           │
      ┌────────────────────┼────────────────────┐
      ↓                    ↓                    ↓
 Identity &           Operations            Governance
 Verification         & Support             & Security
      │                    │                    │
      ↓                    ↓                    ↓
 Doctors              Appointments           Audit
 Nurses               Orders                 Permissions
 Pharmacy             Labs                   Security
 Laboratory           Blood                  Finance
```

The Admin module should make the system:

```text
Observable
Controlled
Auditable
Secure
Recoverable
```

while preserving the principle that:

```text
Admin authority ≠ unlimited clinical authority
```

---

# 147. Final Backend Checklist

```text
[ ] Admin identity integrated with common auth
[ ] Admin roles implemented
[ ] Admin permissions implemented
[ ] Authorization policies implemented
[ ] Dashboard APIs implemented
[ ] User management APIs implemented
[ ] Verification APIs implemented
[ ] Complaint APIs implemented
[ ] Finance APIs implemented
[ ] Catalog APIs implemented
[ ] Security APIs implemented
[ ] Audit service implemented
[ ] Notification integration implemented
[ ] Concurrency protection implemented
[ ] Idempotency implemented for sensitive actions
[ ] Input validation implemented
[ ] Global exception handling implemented
[ ] Pagination implemented
[ ] Filtering implemented
[ ] Proper indexes created
[ ] Sensitive fields redacted from logs
[ ] Financial actions transactional
[ ] Critical actions audited
```

---

# 148. Final Frontend Checklist

```text
[ ] Admin Login
[ ] Dashboard
[ ] Role-aware navigation
[ ] Permission-aware action buttons
[ ] Users page
[ ] User details page
[ ] Verification queue
[ ] Verification details
[ ] Document viewer
[ ] Complaint queue
[ ] Complaint details
[ ] Appointment monitor
[ ] Pharmacy order monitor
[ ] Lab booking monitor
[ ] Blood request monitor
[ ] Finance dashboard
[ ] Withdrawal page
[ ] Refund page
[ ] Medicine catalog
[ ] Diagnostic test catalog
[ ] Service catalog
[ ] Audit page
[ ] Security events page
[ ] Admin management page
[ ] Settings page
[ ] Confirmation dialogs
[ ] Error handling
[ ] Empty states
[ ] Loading states
[ ] Pagination
[ ] Filters
[ ] Search
[ ] Responsive layout
[ ] Arabic/English localization
```

---

# 149. Final QA Checklist

```text
[ ] Authentication tests
[ ] Authorization tests
[ ] IDOR tests
[ ] Verification tests
[ ] Suspension tests
[ ] Complaint lifecycle tests
[ ] Appointment monitoring tests
[ ] Pharmacy order tests
[ ] Lab workflow monitoring tests
[ ] Blood request monitoring tests
[ ] Payment reconciliation tests
[ ] Refund tests
[ ] Withdrawal concurrency tests
[ ] Catalog tests
[ ] Audit log tests
[ ] Security event tests
[ ] File security tests
[ ] Pagination tests
[ ] Filtering tests
[ ] Performance tests
[ ] Error-state tests
```

---

# 150. Complete Admin Journey in One View

```text
ADMIN LOGIN
    ↓
AUTHENTICATION
    ↓
MFA / SESSION CHECK
    ↓
LOAD ROLE + PERMISSIONS
    ↓
DASHBOARD
    ↓
┌───────────────────────────────────────────────┐
│                                               │
│  USERS        VERIFICATION       OPERATIONS   │
│    ↓               ↓                  ↓       │
│ Patients       Doctors            Appointments│
│ Providers      Nurses             Orders       │
│ Donors         Pharmacy           Lab          │
│                Laboratory         Blood        │
│                                               │
│  SUPPORT        FINANCE            CATALOGS   │
│    ↓               ↓                  ↓       │
│ Complaints    Payments            Medicines   │
│ Reviews       Refunds              Tests       │
│ Cases         Withdrawals          Services    │
│                                               │
│  SECURITY        GOVERNANCE                    │
│    ↓               ↓                          │
│ Audit          Admin Users                    │
│ Security       Roles / Permissions            │
│ Events         Settings                       │
│                                               │
└───────────────────────────────────────────────┘
    ↓
EVERY CRITICAL ACTION
    ↓
VALIDATE
    ↓
AUTHORIZE
    ↓
EXECUTE TRANSACTION
    ↓
AUDIT
    ↓
NOTIFY WHEN REQUIRED
    ↓
DONE
```

---

# End of Document

**TechCare Admin Complete Workflow**

This document is intended to serve as the main functional specification for the Admin module and as a bridge between Product Requirements, UX/UI, ASP.NET Core APIs, EF Core entities, security policies, and QA test cases.
