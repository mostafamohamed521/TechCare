# TechCare — Rating & Complaint Complete Workflow

> **Document Type:** Cross-System Product & Technical Workflow Specification  
> **Project:** TechCare Healthcare Platform  
> **Module:** Ratings, Reviews, Complaints, Disputes & Moderation  
> **Architecture Context:** ASP.NET Core Web API + Frontend + Relational Database  
> **Scope:** Shared platform workflow used across Doctor, Nurse, Pharmacy, Laboratory, and other eligible services.  
> **Goal:** Build a trustworthy quality, feedback, moderation, dispute-resolution, and safety system without allowing ratings or complaints to become a channel for abuse, extortion, privacy leakage, or manipulation.

---

# 1. Module Overview

The Rating & Complaint module is a shared quality-control system across TechCare.

It handles two related but different concepts:

```text
RATING / REVIEW
    = Customer feedback about a completed eligible experience

COMPLAINT
    = Formal problem report requiring investigation or resolution
```

A rating should not automatically create a complaint.
A complaint should not automatically change a rating.

The system must maintain a clear distinction:

```text
Service / Order / Booking
        ↓
Eligibility for Feedback
        ├── Rating / Review
        └── Complaint
                ↓
          Investigation
                ↓
      Resolution / Escalation
```

The module should support:

- Ratings from 1 to 5.
- Written reviews.
- Optional structured sub-ratings.
- One eligible rating per eligible transaction, according to product policy.
- Review editing within a controlled window if enabled.
- Review moderation.
- Review reports.
- Complaints tied to real TechCare resources.
- Complaint categories and priorities.
- Evidence attachments.
- Provider response.
- Admin assignment.
- Status transitions.
- Escalation.
- Refund/cancellation recommendations or integrations.
- Clinical-safety escalation when necessary.
- Audit logs.
- Abuse detection.
- Privacy controls.
- Notifications.
- Reporting and analytics.

---

# 2. Main Objectives

The module should allow TechCare to:

1. Collect trustworthy feedback after real completed services.
2. Give providers visibility into customer experience.
3. Give patients a formal channel to report service problems.
4. Link every complaint to the actual business context whenever possible.
5. Prevent fake reviews and duplicate ratings.
6. Protect private patient and provider information.
7. Separate normal dissatisfaction from safety-critical incidents.
8. Give admins a structured investigation workflow.
9. Give providers a fair right to respond.
10. Keep an immutable history of important moderation and resolution events.
11. Integrate complaints with refunds, cancellations, payment disputes, and account moderation.
12. Preserve historical ratings and complaint history even when an account is later suspended/deactivated.

---

# 3. Core Principles

## 3.1 One Real Experience → One Eligible Rating

A rating should be tied to a legitimate completed business interaction.

Examples:

```text
Doctor Appointment → Completed → Rating Eligible
Nurse Visit       → Completed → Rating Eligible
Pharmacy Order    → Completed → Rating Eligible
Laboratory Test   → Completed → Rating Eligible
```

A user should not be able to rate a provider merely because the provider appears in search.

## 3.2 Complaint Is Not a Review

A complaint requires workflow and resolution.

```text
Review:
"Service was delayed."

Complaint:
"The appointment was delayed by 90 minutes and I want TechCare to investigate."
```

A review expresses experience.
A complaint requests formal intervention.

## 3.3 Admin Does Not Automatically Become a Clinician

Admin may investigate operational facts, payment events, scheduling, communication, and policy violations.

Admin must not independently make medical judgments merely because the complaint concerns:

- diagnosis
- prescription
- laboratory result
- medication
- procedure

Clinical complaints requiring professional review should be escalated to the appropriate authorized professional/clinical workflow.

## 3.4 Negative Feedback Is Not Automatically Abuse

A low rating can be legitimate.

Moderation should target behavior such as:

- spam
- threats
- harassment
- personal data exposure
- fraud
- unrelated advertising
- impersonation
- manipulation
- repeated duplicated submissions

Do not hide a review simply because it is negative.

---

# 4. Supported Actors

## 4.1 Patient / Customer

Can:

- Rate eligible services.
- Write reviews.
- Edit reviews within policy.
- Report reviews.
- Create complaints.
- Upload allowed evidence.
- Read complaint status.
- Respond to admin questions.
- Withdraw a complaint before final resolution where policy permits.

## 4.2 Doctor

Can:

- Read own reviews.
- Respond to eligible reviews.
- View own complaints.
- Respond to complaints.
- Upload authorized evidence.
- See complaint status.
- Report abusive reviews.

## 4.3 Nurse

Same core capabilities as Doctor for eligible nursing services.

## 4.4 Pharmacy Owner / Manager

Can:

- Read pharmacy reviews.
- Respond according to policy.
- Review complaints against the pharmacy.
- Respond to complaints.
- Provide evidence.

## 4.5 Laboratory Owner / Manager

Can:

- Read lab reviews.
- Respond to reviews.
- Handle operational complaints.
- Submit evidence.

Clinical result complaints may require professional escalation.

## 4.6 Admin / Support Admin

Can:

- Review complaints.
- Assign cases.
- Request information.
- Moderate reviews.
- Apply approved resolutions.
- Escalate cases.
- Trigger refunds through authorized finance workflows.
- Suspend abusive accounts when permitted.

## 4.7 Finance Admin

Can support complaints with financial consequences:

- payment dispute
- refund
- provider earnings adjustment
- payout hold

Finance actions remain governed by the Financial module.

## 4.8 Clinical Reviewer / Authorized Professional

Future or role-specific capability for medically sensitive cases.

Can review only the clinical aspect within their professional scope.

---

# 5. Rating Eligibility

A rating should generally require:

```text
Authenticated user
    ↓
Eligible completed transaction
    ↓
Current user is authorized participant
    ↓
Rating not already submitted
    ↓
Feedback window is open
    ↓
Allow rating
```

The exact feedback window is a business configuration.

Example:

```text
CompletedAt = 10:00
FeedbackUntil = CompletedAt + configured period
```

---

# 6. Eligible Business Types

The platform may support ratings for:

```text
Doctor Appointment
Nurse Visit
Pharmacy Order
Laboratory Service / Booking
```

Blood donation coordination can have a separate experience feedback/reporting flow and should not automatically be treated as a commercial provider rating.

Admin and Authentication events are not normal rating targets.

---

# 7. Rating Creation Workflow

```text
Service Completed
      ↓
Rating Eligibility Created
      ↓
Patient Notification / Review Prompt
      ↓
Patient Opens Review
      ↓
Select Stars
      ↓
Optional Text Review
      ↓
Optional Structured Ratings
      ↓
Validation
      ↓
Submit
      ↓
Rating Stored
      ↓
Provider Aggregate Recalculated
      ↓
Provider Notification
```

---

# 8. Rating Fields

Suggested fields:

```text
RatingId
AuthorUserId
TargetType
TargetId
BusinessType
BusinessObjectId
OverallScore
ReviewText
CreatedAt
UpdatedAt
Status
ModerationState
VerifiedTransaction
```

Optional sub-ratings:

```text
CommunicationScore
PunctualityScore
ProfessionalismScore
ServiceQualityScore
```

Use only criteria that make sense for the service.

---

# 9. Rating Scale

Recommended:

```text
1 = Very Poor
2 = Poor
3 = Fair
4 = Good
5 = Excellent
```

Do not allow:

```text
0
6
3.7
-1
```

unless the product intentionally supports another scale.

---

# 10. Verified Experience Badge

Reviews tied to actual completed transactions can display:

```text
✓ Verified Experience
```

The badge should be based on server-side eligibility, not user-provided claims.

---

# 11. One Rating Per Transaction

Recommended uniqueness rule:

```text
BusinessObjectId + AuthorUserId + RatingTarget
```

This prevents:

```text
Patient rates same completed appointment 10 times
```

If editing is allowed, the same record is updated rather than creating a second rating.

---

# 12. Multiple Targets in One Transaction

The product may later support separate ratings for:

```text
Doctor Service
Nurse Service
Pharmacy Experience
Delivery Experience
Laboratory Collection Experience
```

If enabled, each target must be explicit.

Example:

```text
Pharmacy Order #100
 ├── Pharmacy Rating
 └── Delivery Rating
```

Do not accidentally count two sub-ratings as two independent customer experiences when calculating the main provider score.

---

# 13. Review Text Rules

Suggested limits:

```text
Minimum: configured if desired
Maximum: reasonable character limit
```

Review text should be:

- user-generated
- plain text or safely rendered markup
- sanitized/encoded
- free from executable content

Potential moderation triggers:

- threats
- harassment
- personal medical data of another person
- phone numbers / addresses
- payment credentials
- spam
- links to unrelated services

---

# 14. Review Editing

Optional MVP rule:

```text
One rating
One review
Editable for a configured period
```

Example flow:

```text
Original Review
   ↓
Edit
   ↓
Update
   ↓
Record UpdatedAt
```

Important moderation history should not be destroyed.

---

# 15. Review Revision History

For stronger auditability:

```text
Review
   ↓
ReviewRevision
```

Fields:

```text
RevisionId
ReviewId
PreviousText
PreviousScore
ChangedAt
ChangedBy
ChangeReason
```

Do not necessarily show full revision history publicly.

---

# 16. Review Statuses

Recommended:

```text
PUBLISHED
FLAGGED
UNDER_REVIEW
HIDDEN
RESTORED
DELETED_BY_AUTHOR
ARCHIVED
```

A hidden review remains in history for audit and moderation.

---

# 17. Review Moderation Workflow

```text
Review Published
      ↓
User / Provider / System flags it
      ↓
FLAGGED
      ↓
Admin reviews
      ↓
┌────────────┬─────────────┐
│            │             │
Valid      Violation     Unclear
│            │             │
↓            ↓             ↓
Keep       Hide/Remove   More Review
VISIBLE       │
              ↓
          MODERATION LOG
```

---

# 18. Review Report

A provider or user can report a review.

Fields:

```text
ReportId
ReporterUserId
ReviewId
Category
Description
CreatedAt
Status
```

Categories:

```text
SPAM
HARASSMENT
PERSONAL_DATA
FRAUD
IRRELEVANT
IMPERSONATION
DUPLICATE
OTHER
```

---

# 19. Provider Response to Review

Provider can respond to an eligible review.

Example:

```text
Patient Review
"The visit was late."

Doctor Response
"We apologize for the delay and are reviewing the scheduling issue."
```

The provider should not:

- expose patient's medical information
- disclose private appointment details
- threaten the reviewer
- reveal private contact information

Provider response must follow the same moderation rules.

---

# 20. Complaint Creation Workflow

```text
User opens eligible transaction
        ↓
Report a Problem
        ↓
Select Category
        ↓
Describe Issue
        ↓
Optional Evidence
        ↓
Submit
        ↓
Complaint Created
        ↓
Case Number Generated
        ↓
Admin Queue
        ↓
Acknowledgement Notification
```

---

# 21. Complaint Must Be Linked to Context

Whenever possible, complaint should reference a real resource:

```text
AppointmentId
OrderId
LabBookingId
BloodRequestId
PaymentId
ProviderId
```

This allows Admin to investigate actual records instead of relying only on free text.

---

# 22. Complaint Fields

Recommended:

```text
ComplaintId
ReporterUserId
TargetType
TargetId
BusinessType
BusinessObjectId
Category
Subcategory
Subject
Description
Priority
Status
AssignedAdminId
CreatedAt
UpdatedAt
ResolvedAt
ClosedAt
ResolutionCode
```

---

# 23. Complaint Categories

Suggested universal categories:

```text
SERVICE_QUALITY
DELAY
NO_SHOW
CANCELLATION
PRICING
PAYMENT
REFUND
COMMUNICATION
STAFF_BEHAVIOR
PRIVACY
FRAUD
TECHNICAL_ISSUE
DELIVERY
PRESCRIPTION
LAB_RESULT
MEDICAL_SAFETY
OTHER
```

The actual categories should remain configurable.

---

# 24. Priority Levels

```text
LOW
NORMAL
HIGH
CRITICAL
```

## LOW

Minor service dissatisfaction.

## NORMAL

Standard customer-service issue.

## HIGH

Significant financial/service problem.

## CRITICAL

Potential safety/security incident requiring immediate escalation.

Do not allow every patient to simply choose CRITICAL without validation.

---

# 25. Complaint State Machine

Recommended:

```text
SUBMITTED
    ↓
OPEN
    ↓
ASSIGNED
    ↓
UNDER_REVIEW
    ├── WAITING_FOR_PATIENT
    ├── WAITING_FOR_PROVIDER
    ├── WAITING_FOR_FINANCE
    └── ESCALATED
            ↓
        RESOLUTION_RECOMMENDED
            ↓
           RESOLVED
            ↓
           CLOSED
```

Alternative terminal states:

```text
REJECTED
DUPLICATE
CANCELLED
```

---

# 26. Complaint Submission Validation

Before creation:

```text
User authenticated
   ↓
User owns/participates in referenced transaction
   ↓
Transaction exists
   ↓
Complaint allowed for current state
   ↓
Category valid
   ↓
Description valid
   ↓
Create complaint
```

Do not accept a random `ProviderId` and assume the complaint is legitimate.

---

# 27. Anonymous Complaints

Recommended MVP rule:

```text
Complaint = authenticated user only
```

This improves traceability and abuse prevention.

Anonymous reporting may be added later for selected safety channels.

---

# 28. Complaint Evidence

Evidence can include:

```text
Images
PDFs
Screenshots
Order receipts
Payment references
Other approved files
```

Do not allow arbitrary executable files.

Files must be securely stored.

---

# 29. Evidence Security

Rules:

- Private storage.
- Authorization checks.
- File type validation.
- Size limits.
- Secure download.
- Audit access when highly sensitive.
- No predictable public URLs.

---

# 30. Complaint Conversation

A complaint may need structured communication.

Entities:

```text
Complaint
   ↓
ComplaintMessage
```

Message fields:

```text
MessageId
ComplaintId
SenderUserId
SenderType
Message
IsInternal
CreatedAt
```

---

# 31. Public vs Internal Notes

Critical distinction:

```text
Public Message
```
can be shown to parties in the complaint.

```text
Internal Note
```
is visible only to authorized Admins.

Never accidentally expose internal notes to patients/providers.

---

# 32. Complaint Assignment

```text
Complaint Created
      ↓
Admin Queue
      ↓
Assign to Support Admin
      ↓
AssignedAdminId
      ↓
Notification
```

Assignment fields:

```text
AssignedTo
AssignedBy
AssignedAt
```

---

# 33. Reassignment

An Admin can reassign a case if authorized.

Example:

```text
Support Admin
      ↓
Complex Finance Issue
      ↓
Reassign to Finance Admin
```

The reassignment must be audited.

---

# 34. Complaint Investigation Workflow

```text
Open Complaint
     ↓
Identify related transaction
     ↓
Read status history
     ↓
Read relevant notifications
     ↓
Check payment status if relevant
     ↓
Check provider/account status
     ↓
Review evidence
     ↓
Request additional information
     ↓
Evaluate policy
     ↓
Select resolution
     ↓
Execute authorized actions
     ↓
Notify parties
     ↓
Resolve
```

---

# 35. Related Timeline

Admin investigation should have a timeline.

Example:

```text
09:00 Booking created
09:03 Payment succeeded
09:04 Provider accepted
10:00 Appointment started
10:55 Appointment completed
11:10 Complaint submitted
11:20 Admin assigned
```

The timeline must use authoritative records.

---

# 36. Complaint Resolution Options

Possible resolution categories:

```text
NO_ACTION_REQUIRED
EXPLANATION_PROVIDED
USER_APOLOGY
PROVIDER_WARNING
PROVIDER_POLICY_ACTION
REFUND_APPROVED
PARTIAL_REFUND_APPROVED
APPOINTMENT_RESCHEDULED
ORDER_REPLACEMENT
ACCOUNT_RESTRICTION
ESCALATED_TO_CLINICAL_REVIEW
ESCALATED_TO_SECURITY
DUPLICATE_CASE_CLOSED
```

Resolution codes should be separate from free-text notes.

---

# 37. Refund Integration

A complaint should not directly modify payment data.

Correct:

```text
Complaint
   ↓
Resolution Recommendation
   ↓
Finance / Refund Service
   ↓
Refund Request
   ↓
Financial Workflow
```

This maintains separation of duties.

---

# 38. Provider Warning

For repeated complaints:

```text
Complaint Resolved
   ↓
Policy Warning
   ↓
Provider account receives warning event
```

Warnings must be:

- evidence-based
- auditable
- policy-driven

---

# 39. Provider Suspension from Complaints

Repeated verified severe violations may lead to:

```text
ACTIVE
   ↓
UNDER_REVIEW
   ↓
RESTRICTED
   ↓
SUSPENDED
```

Suspension is an Admin/account workflow, not a simple complaint status.

---

# 40. Clinical Complaint Escalation

Examples:

- Suspected medication harm.
- Serious diagnosis dispute.
- Lab result safety issue.
- Suspected clinical malpractice.

Workflow:

```text
Complaint
   ↓
Category = MEDICAL_SAFETY
   ↓
Immediate Priority Review
   ↓
Authorized Clinical Reviewer / Appropriate Facility
   ↓
Clinical Review
   ↓
Operational/Admin Resolution
```

Admin should not independently declare a medical fact.

---

# 41. Privacy Complaint

Example:

```text
Patient claims provider shared private health information.
```

Workflow:

```text
Complaint
   ↓
PRIVACY
   ↓
HIGH / CRITICAL priority assessment
   ↓
Review access logs
   ↓
Review messages/files
   ↓
Security/Privacy escalation
   ↓
Containment if needed
   ↓
Resolution
```

---

# 42. Payment Complaint

```text
Patient claims payment succeeded but service failed.
```

Admin reviews:

```text
Business Status
Payment Status
Refund Status
Ledger
Notification History
```

If refund is justified:

```text
Complaint → Finance → Refund
```

---

# 43. Pharmacy Complaint

Potential issue:

```text
Patient ordered medicine
↓
Pharmacy rejected due to stock
↓
Payment succeeded
```

Workflow:

```text
Complaint
 ↓
Open Order
 ↓
Check Inventory State History
 ↓
Check Payment
 ↓
Check Pharmacy Response
 ↓
Determine Refund / Resolution
```

Do not let the Admin rewrite order history to hide the failure.

---

# 44. Laboratory Complaint

Potential issue:

```text
Sample collected
↓
Result delayed
```

Admin reviews:

```text
Booking
Collector Assignment
Sample Timeline
Lab Processing State
Result Publication State
```

If clinically sensitive:

```text
Operational Admin Review
      ↓
Clinical / Lab Professional Review
```

---

# 45. Doctor Complaint

Potential issues:

- Late arrival.
- No-show.
- Communication behavior.
- Appointment cancellation.
- Prescription issue.
- Payment issue.

Separate:

```text
Operational Complaint
vs
Clinical Complaint
```

---

# 46. Nurse Complaint

Potential issues:

- Late home arrival.
- Visit not completed.
- Communication.
- Service mismatch.
- Safety concern.

Use the same formal complaint lifecycle.

---

# 47. Patient Complaint Against Patient

Some complaints can involve another patient, such as:

- harassment
- abusive messages
- fake requests
- blood donation abuse

Workflow:

```text
Report
 ↓
Safety Review
 ↓
Account Investigation
 ↓
Warning / Restriction / Suspension
```

---

# 48. Complaint Anti-Spam

Controls:

- Rate limits.
- Duplicate detection.
- Active-case detection.
- Abuse scoring signals.
- Require reference to real transaction when applicable.

Do not automatically ban users because they submit many legitimate complaints.

---

# 49. Duplicate Complaint Detection

Possible duplicate key:

```text
Reporter
+ Business Object
+ Category
+ Active Status
```

If a duplicate is likely:

```text
Offer existing case
```

rather than silently creating another complaint.

---

# 50. Duplicate Review Detection

The system should prevent identical ratings for the same eligible transaction.

Possible duplicate signals for spam moderation:

- identical text
- rapid repeated reviews
- many accounts targeting one provider
- impossible transaction linkage

These are signals, not proof of fraud.

---

# 51. Rating Aggregation

Provider rating should be calculated from eligible visible ratings according to a defined rule.

Basic model:

```text
Average Rating
=
Sum(Eligible Visible Scores)
÷
Count(Eligible Visible Scores)
```

The calculation should not include:

- hidden reviews
- invalid/duplicate ratings
- unauthorized ratings

unless product policy says otherwise.

---

# 52. Rating Precision

Public display may show:

```text
4.8 ★
```

while the system stores the underlying exact score from eligible ratings.

Do not round individual ratings before aggregation.

---

# 53. Rating Count

Show:

```text
4.8 ★
(127 reviews)
```

The count should match the same eligibility/filter rules used for the average.

---

# 54. New Provider Rating Protection

A provider with:

```text
1 review = 5.0
```

should not necessarily appear equivalent to a provider with:

```text
500 reviews = 4.9
```

The search-ranking system can later use a confidence/volume signal.

Do not manipulate the displayed rating itself to hide legitimate scores.

---

# 55. Provider Rating Dashboard

Doctor/Nurse/Pharmacy/Lab can see:

```text
Average Rating
Total Reviews
Recent Reviews
Rating Distribution
Sub-ratings if enabled
Unanswered Reviews
Complaints Summary
```

Example distribution:

```text
5 ★  78%
4 ★  15%
3 ★   4%
2 ★   2%
1 ★   1%
```

---

# 56. Patient Rating Dashboard

Patient can see:

```text
My Reviews
Eligible Services Awaiting Review
Edited Reviews
Reported Reviews
```

---

# 57. Review Prompt Workflow

After completion:

```text
Service Completed
      ↓
Check rating eligibility
      ↓
Create ReviewPromptEvent
      ↓
Notification
      ↓
Patient opens rating screen
```

Do not spam the patient repeatedly.

---

# 58. Review Prompt Deduplication

Use:

```text
BusinessObjectId + PromptType
```

so one appointment does not create 20 review prompts.

---

# 59. Review Reminder

Optional reminders can be sent if no rating exists.

Example:

```text
First prompt
↓
configured delay
↓
one reminder
↓
stop after rating / expiration
```

Do not send reminders indefinitely.

---

# 60. Complaint Reminder

If an Admin is waiting for information:

```text
WAITING_FOR_PATIENT
       ↓
Reminder
       ↓
Still no response
       ↓
Auto-close / pause according to policy
```

The business policy should define whether closing a complaint affects the user's ability to reopen it.

---

# 61. Provider Complaint Response

Provider can respond:

```text
Complaint
  ↓
WAITING_FOR_PROVIDER
  ↓
Provider submits response
  ↓
UNDER_REVIEW
```

Provider response is part of the case record.

---

# 62. Patient Complaint Response

Admin may request:

- more details
- documents
- order/booking confirmation
- clarification

Patient responds through the case.

The system should preserve message history.

---

# 63. Evidence Integrity

Once an evidence file is submitted:

- Store a reference.
- Record uploader.
- Record timestamp.
- Prevent silent replacement without audit.

If replacement is allowed:

```text
Old file remains in audit history
New file becomes active evidence
```

---

# 64. Complaint Attachments Privacy

Attachments may contain sensitive medical data.

Only show them to:

```text
Reporter
Authorized Provider/Organization participant
Assigned Admin
Authorized Reviewer
```

according to the case permissions.

---

# 65. Complaint Notifications

Events:

```text
ComplaintCreated
ComplaintAssigned
ComplaintResponseRequested
ComplaintProviderResponded
ComplaintEscalated
ComplaintResolved
ComplaintClosed
ComplaintReopened
```

---

# 66. Complaint Reopen

A resolved complaint may be reopened only according to policy.

Example:

```text
RESOLVED
   ↓
Reopen request within allowed window
   ↓
REOPENED
   ↓
UNDER_REVIEW
```

Reopening must not erase the original resolution history.

---

# 67. Escalation Rules

Automatic or manual escalation can be based on:

- priority
- category
- financial value
- safety signal
- privacy incident
- repeated provider complaints
- repeated patient abuse
- unresolved case age

The exact thresholds are configurable.

---

# 68. Complaint SLA Tracking

Even without hardcoding a fixed duration in the product, cases can record:

```text
CreatedAt
DueAt
CurrentAge
Overdue
```

This supports operational dashboards.

---

# 69. Admin Complaint Queue

Suggested columns:

```text
Complaint ID
Category
Priority
Reporter
Target
Business Object
Status
Assigned Admin
Created At
Age
```

Sort by:

```text
Priority DESC
Overdue DESC
CreatedAt ASC
```

---

# 70. Admin Complaint Details Page

Sections:

```text
Overview
Related Transaction
Timeline
Messages
Evidence
Provider Response
Financial Impact
Security Signals
Resolution
Audit History
```

Sensitive sections are permission-controlled.

---

# 71. Complaint Financial Impact

If finance is affected, display:

```text
Original Payment
Refund Requested
Refund Approved
Provider Earning Impact
Platform Fee Impact
```

Do not allow Support Admin to directly mutate those values.

---

# 72. Complaint Security Impact

For privacy/fraud issues, show authorized security information:

```text
Suspicious login events
Authorization failures
Reported account history
Relevant audit events
```

Do not expose secrets.

---

# 73. Moderation Action Types

Possible moderation actions:

```text
KEEP_VISIBLE
HIDE_REVIEW
RESTORE_REVIEW
WARN_USER
RESTRICT_USER
SUSPEND_USER
REMOVE_INVALID_CONTENT
ESCALATE
```

Every action requires an appropriate permission.

---

# 74. Review Moderation Reason Codes

```text
SPAM
THREAT
HARASSMENT
PERSONAL_DATA
FRAUD
IMPERSONATION
ADVERTISING
DUPLICATE
OFF_TOPIC
OTHER
```

---

# 75. Provider Appeal

A provider may appeal a moderation decision where the product supports it.

Flow:

```text
Review Hidden
   ↓
Provider Appeal
   ↓
Admin Review
   ↓
Keep Hidden / Restore
   ↓
Final Audit
```

---

# 76. Patient Appeal

For significant complaint resolutions:

```text
Resolved
   ↓
Patient Appeal
   ↓
Senior Review
   ↓
Final Decision
```

Avoid endless appeal loops.

---

# 77. Separation of Duties

Sensitive case actions should be permissioned separately.

Examples:

```text
Complaint.Read
Complaint.Assign
Complaint.Resolve
Complaint.Escalate
Review.Moderate
Finance.Refund
User.Suspend
```

A Support Admin who can resolve complaints does not automatically get permission to execute a refund or suspend an account.

---

# 78. Admin Authorization

Every complaint/review endpoint must validate:

```text
Authentication
     ↓
Permission
     ↓
Resource scope
     ↓
Business state
```

Frontend visibility does not replace backend authorization.

---

# 79. Object-Level Authorization Examples

### Patient

```text
GET /reviews/123
```

Must verify the patient owns/is authorized to view review 123.

### Provider

Must verify review targets the provider's own profile/service.

### Pharmacy Staff

Must verify organization membership before accessing pharmacy complaint/order records.

### Admin

Must verify admin permission and sensitive-data scope.

---

# 80. IDOR Protection

Test:

```text
Patient A requests Complaint B
Provider A requests Complaint of Provider B
Pharmacy A requests Review of Pharmacy B
Admin role A requests Finance-only complaint data
```

Unauthorized requests must fail.

---

# 81. Complaint API Groups

Suggested routes:

```http
POST   /api/v1/complaints
GET    /api/v1/complaints/my
GET    /api/v1/complaints/{id}
POST   /api/v1/complaints/{id}/messages
POST   /api/v1/complaints/{id}/attachments
POST   /api/v1/complaints/{id}/cancel
POST   /api/v1/complaints/{id}/reopen
POST   /api/v1/complaints/{id}/appeal
```

Provider/organization routes can use scoped endpoints where appropriate.

---

# 82. Review API Groups

```http
POST   /api/v1/reviews
GET    /api/v1/reviews/{id}
PUT    /api/v1/reviews/{id}
DELETE /api/v1/reviews/{id}
POST   /api/v1/reviews/{id}/report
POST   /api/v1/reviews/{id}/response
```

Final endpoint names depend on API conventions.

---

# 83. Admin Complaint APIs

```http
GET  /api/v1/admin/complaints
GET  /api/v1/admin/complaints/{id}
POST /api/v1/admin/complaints/{id}/assign
POST /api/v1/admin/complaints/{id}/status
POST /api/v1/admin/complaints/{id}/request-info
POST /api/v1/admin/complaints/{id}/resolve
POST /api/v1/admin/complaints/{id}/escalate
POST /api/v1/admin/complaints/{id}/close
```

---

# 84. Admin Review APIs

```http
GET  /api/v1/admin/reviews/flagged
GET  /api/v1/admin/reviews/{id}
POST /api/v1/admin/reviews/{id}/hide
POST /api/v1/admin/reviews/{id}/restore
POST /api/v1/admin/reviews/{id}/moderate
```

---

# 85. Database Entities

Recommended:

```text
Review
ReviewRevision
ReviewResponse
ReviewReport
ReviewModerationAction
RatingEligibility

Complaint
ComplaintMessage
ComplaintAttachment
ComplaintStatusHistory
ComplaintAssignment
ComplaintResolution
ComplaintAppeal

Notification
AuditLog
```

---

# 86. Review Entity

Suggested fields:

```text
Id
AuthorUserId
TargetType
TargetId
BusinessType
BusinessObjectId
OverallScore
ReviewText
Status
ModerationState
VerifiedExperience
CreatedAt
UpdatedAt
PublishedAt
HiddenAt
```

---

# 87. RatingEligibility Entity

Suggested:

```text
Id
UserId
BusinessType
BusinessObjectId
TargetType
TargetId
EligibleAt
ExpiresAt
UsedAt
CreatedAt
```

Unique rule should prevent duplicate eligibility records for the same experience/target unless the product intentionally supports multiple targets.

---

# 88. ReviewReport Entity

```text
Id
ReporterUserId
ReviewId
Category
Description
Status
AssignedAdminId
CreatedAt
ResolvedAt
```

---

# 89. ReviewModerationAction Entity

```text
Id
ReviewId
AdminUserId
ActionType
ReasonCode
Note
CreatedAt
```

This creates a clean moderation history.

---

# 90. Complaint Entity

Suggested:

```text
Id
ReporterUserId
TargetType
TargetId
BusinessType
BusinessObjectId
Category
Subcategory
Priority
Subject
Description
Status
AssignedAdminId
CreatedAt
UpdatedAt
ResolvedAt
ClosedAt
ResolutionCode
```

---

# 91. ComplaintMessage Entity

```text
Id
ComplaintId
SenderUserId
SenderType
Message
IsInternal
CreatedAt
EditedAt
```

Internal messages must never be returned by public complaint endpoints.

---

# 92. ComplaintAttachment Entity

```text
Id
ComplaintId
UploadedByUserId
FileNameSafe
StorageKey
MimeType
Size
IsActive
UploadedAt
```

Use storage keys, not public URLs, as the core database reference.

---

# 93. ComplaintStatusHistory Entity

```text
Id
ComplaintId
PreviousStatus
NewStatus
ChangedByUserId
ReasonCode
Note
CreatedAt
```

---

# 94. ComplaintResolution Entity

```text
Id
ComplaintId
ResolutionCode
Summary
ResolvedByUserId
CreatedAt
```

Financial effects should link to finance entities rather than being hidden inside the resolution text.

---

# 95. Complaint Appeal Entity

```text
Id
ComplaintId
RequestedByUserId
Reason
Status
ReviewedByAdminId
Decision
CreatedAt
ResolvedAt
```

---

# 96. Database Relationships

```text
User
 ├──< Review
 ├──< ReviewReport
 ├──< Complaint
 ├──< ComplaintMessage
 └──< RatingEligibility

Review
 ├──< ReviewRevision
 ├──< ReviewResponse
 ├──< ReviewReport
 └──< ReviewModerationAction

Complaint
 ├──< ComplaintMessage
 ├──< ComplaintAttachment
 ├──< ComplaintStatusHistory
 ├──< ComplaintAssignment
 ├──< ComplaintResolution
 └──< ComplaintAppeal
```

---

# 97. Rating Aggregation Strategy

The application can maintain an aggregate projection for performance:

```text
ProviderRatingSummary
```

Fields:

```text
TargetId
AverageScore
ReviewCount
LastUpdatedAt
```

But the underlying `Review` records remain the source for recalculation.

---

# 98. Recalculate Rating Safely

When a rating is created/hidden/restored:

```text
Review change
   ↓
Recalculate affected target
   ↓
Update summary
```

If using a cached aggregate, keep a way to rebuild it from source reviews.

---

# 99. Rating Aggregation Concurrency

Two users submit ratings simultaneously.

The system must avoid lost updates.

Safer patterns:

```text
Transactional insert
     ↓
Aggregate update
```

or an asynchronous projection that can rebuild from source data.

Do not let two concurrent requests overwrite the aggregate with stale counts.

---

# 100. Complaint Concurrency

Two Admins try to resolve the same complaint.

Expected:

```text
Admin A → resolves successfully
Admin B → receives state/concurrency conflict
```

There should be one valid final resolution event.

---

# 101. Review Moderation Concurrency

Two Admins moderate the same review.

The review state transition must be protected.

Example:

```text
VISIBLE
   ↓
Admin A → HIDDEN
Admin B → HIDDEN
```

Second action should not create inconsistent history.

---

# 102. Complaint Rate Limiting

Protect:

```text
POST /complaints
POST /complaints/{id}/messages
POST /reviews
POST /reviews/{id}/report
```

Use different thresholds for normal users and trusted internal services/admins where appropriate.

---

# 103. Review Abuse Protection

Signals:

- Many reviews from one account.
- Many accounts rating one provider in a suspicious pattern.
- Reviews without completed transactions.
- Repeated identical text.
- Very fast rating bursts.
- Reviews from accounts with suspicious activity.

Signals should feed moderation workflows and should not automatically determine guilt.

---

# 104. Provider Retaliation Protection

Providers must not be able to see unnecessary private patient identity information in a review.

Example public review:

```text
"Service was late."
```

Provider should not receive:

```text
Patient's full medical history
Home address
Private account data
```

unless required by another authorized workflow.

---

# 105. Patient Safety Protection

If a provider response contains:

- threats
- personal data
- retaliation
- abusive language

the response can be reported and moderated separately.

---

# 106. No Review Extortion

A provider must not condition refunds or service continuation on receiving a positive review.

Similarly, a patient should not threaten to submit a false review in exchange for money/service.

Fraud signals can be investigated by Admin.

---

# 107. Review Edit Abuse

A user may attempt:

```text
Positive review
↓
Threat to change to 1 star
↓
Demand compensation
```

The platform should treat explicit coercion as an abuse signal and preserve revision history.

---

# 108. Complaint vs Review Decision Guide

Use **Review** when:

```text
User wants to express service experience.
```

Use **Complaint** when:

```text
User wants an investigation or intervention.
```

Use **Safety Report** when:

```text
There is immediate abuse/privacy/security risk.
```

Use **Clinical Escalation** when:

```text
Issue requires professional clinical review.
```

---

# 109. Unified “Report Problem” UX

A single patient action can open a routing choice:

```text
What do you want to do?

[Rate Service]
[Report a Problem]
[Report Safety/Abuse]
```

Then the correct workflow is launched.

---

# 110. Complaint Wizard

Step 1:

```text
Select Issue Type
```

Step 2:

```text
Select Specific Problem
```

Step 3:

```text
Describe What Happened
```

Step 4:

```text
Attach Evidence
```

Step 5:

```text
Review Privacy Notice
```

Step 6:

```text
Submit
```

---

# 111. Patient Complaint Form Fields

Recommended:

```text
Related Appointment/Order/Booking
Category
Subject
Description
Desired Outcome (optional)
Attachments
```

Examples of desired outcomes:

```text
Explanation
Refund Review
Reschedule
Provider Review
Technical Fix
```

The desired outcome is not automatically granted.

---

# 112. Provider Complaint Response Form

Fields:

```text
Response
Attachments
Action Taken Internally (optional)
```

Do not allow the provider to edit the patient's original complaint.

---

# 113. Admin Resolution Form

Fields:

```text
Resolution Code
Public Summary
Internal Note
Related Finance Action if any
Escalation Decision
```

Sensitive internal information remains private.

---

# 114. Complaint Notifications

### Patient

- Complaint submitted.
- Admin assigned.
- More information requested.
- Provider response available where applicable.
- Complaint escalated.
- Complaint resolved.
- Appeal update.

### Provider

- Complaint received.
- Response requested.
- Complaint resolved.
- Moderation action.

### Admin

- New complaint.
- Critical complaint.
- Escalation.
- Overdue case.

---

# 115. Rating Notifications

Provider receives:

```text
You received a new review.
[View Review]
```

Never include unnecessary patient personal data.

---

# 116. Review Moderation Notification

If provider/user content is hidden:

```text
Your review was moderated because it violated the platform content policy.
```

Where appropriate, provide a reason category and appeal link.

---

# 117. Complaint Privacy Notification

Sensitive complaint notifications should use minimal text.

Example:

```text
Your complaint has been updated.
Open TechCare to view the latest details securely.
```

Avoid putting sensitive clinical facts in SMS.

---

# 118. Deep Links

Examples:

```text
/reviews/{id}
/complaints/{id}
/orders/{id}
/appointments/{id}
/lab-results/{id}
```

The linked API must re-check authorization.

---

# 119. Notification-to-Complaint Security

A notification is not permission to view the underlying complaint.

Example:

```text
Attacker obtains notification URL
      ↓
GET /complaints/123
      ↓
Authorization check
      ↓
Denied
```

---

# 120. API Error Codes

Suggested:

```text
RATING_NOT_ELIGIBLE
RATING_ALREADY_SUBMITTED
RATING_WINDOW_EXPIRED
REVIEW_NOT_FOUND
REVIEW_NOT_AUTHORIZED
REVIEW_ALREADY_MODERATED
COMPLAINT_NOT_FOUND
COMPLAINT_NOT_AUTHORIZED
COMPLAINT_INVALID_STATE
COMPLAINT_ALREADY_EXISTS
COMPLAINT_RATE_LIMITED
ATTACHMENT_INVALID
ATTACHMENT_TOO_LARGE
MODERATION_NOT_AUTHORIZED
APPEAL_NOT_ALLOWED
CONCURRENCY_CONFLICT
```

---

# 121. Example Rating Error

```json
{
  "success": false,
  "error": {
    "code": "RATING_NOT_ELIGIBLE",
    "message": "This service is not currently eligible for a review."
  }
}
```

---

# 122. Example Complaint Error

```json
{
  "success": false,
  "error": {
    "code": "COMPLAINT_ALREADY_EXISTS",
    "message": "An active complaint already exists for this issue.",
    "existingComplaintId": "..."
  }
}
```

Only return an existing complaint reference when the authenticated user is authorized to see it.

---

# 123. Frontend Pages — Ratings

Recommended:

```text
/reviews
/reviews/create/{businessType}/{id}
/reviews/{id}
/reviews/{id}/edit
```

Provider:

```text
/provider/reviews
/provider/reviews/{id}
```

---

# 124. Frontend Pages — Complaints

Patient:

```text
/complaints
/complaints/create
/complaints/{id}
/complaints/{id}/appeal
```

Provider:

```text
/provider/complaints
/provider/complaints/{id}
```

Admin:

```text
/admin/complaints
/admin/complaints/{id}
```

---

# 125. Provider Rating Dashboard UX

Suggested cards:

```text
Average Rating
Total Reviews
5-Star Rate
Recent Trend
Open Complaints
```

Charts are informational and should not manipulate the underlying score.

---

# 126. Admin Moderation Dashboard UX

Sections:

```text
Flagged Reviews
Open Complaints
High Priority
Critical Safety Cases
Overdue Cases
Escalated
Recently Resolved
```

---

# 127. Admin Review Moderation Screen

Show:

```text
Review
Source Transaction
Review Author (only as permitted)
Provider
Report Reasons
Prior Reports
Moderation History
```

Action panel:

```text
Keep Visible
Hide
Restore
Warn
Escalate
```

---

# 128. Complaint Details Screen

Recommended tabs:

```text
Overview
Timeline
Conversation
Evidence
Financial
Security
Resolution
Audit
```

Tabs should be permission-aware.

---

# 129. Complaint Search Filters

```text
Status
Priority
Category
Target Type
Provider
Reporter
Assigned Admin
Date Range
Overdue
```

---

# 130. Complaint Reporting

Reports:

```text
Complaints by Category
Complaints by Provider
Complaints by Service
Resolution Time
Escalation Rate
Refund Rate
Repeat Complaint Rate
```

---

# 131. Rating Reporting

Reports:

```text
Average Rating by Provider
Average Rating by Role
Rating Distribution
Review Volume
Verified Review Rate
Reported Review Rate
Moderation Rate
```

---

# 132. Trend Analytics

Track:

```text
Rating trend over time
Complaint trend over time
Complaint per 100 completed services
Low-rating rate
Resolution trend
```

Do not compare providers without normalizing for service volume and context where necessary.

---

# 133. Quality KPI Examples

Useful KPIs:

```text
Average Rating
Review Response Rate
Complaint Rate
Complaint Resolution Rate
Escalation Rate
Repeat Complaint Rate
Refund Due to Complaint Rate
Safety Incident Rate
```

These should be interpreted carefully and not treated as perfect measures of provider quality.

---

# 134. Complaint Rate Normalization

Better metric:

```text
Complaint Rate
=
Verified Complaints
÷
Completed Eligible Transactions
× 100
```

This is more meaningful than raw complaint count alone.

---

# 135. Rating Bias Awareness

Ratings can be influenced by:

- service type
- patient expectations
- timing
- waiting time
- price
- technical issues

Analytics should treat rating as one quality signal, not the only signal.

---

# 136. Provider Quality Flag

Future scoring may combine:

```text
Rating
Complaint Rate
Cancellation Rate
No-Show Rate
Verified Complaints
Response Rate
```

Any score used for ranking/suspension should be transparent and policy-approved.

---

# 137. Complaint Fraud Detection Signals

Potential signals:

- many complaints from one account in a short window
- multiple accounts sharing suspicious patterns
- repeated refund requests
- complaint immediately after every transaction
- provider and patient collusion patterns

These should create review signals rather than automatic punishment.

---

# 138. Review Fraud Detection Signals

Potential signals:

- review without transaction
- many reviews within seconds/minutes
- identical wording across accounts
- abnormal rating concentration
- account clusters targeting one provider

---

# 139. Account-Level Safety Integration

Repeated verified abuse can integrate with Account Management:

```text
Safety Evidence
      ↓
Admin Case
      ↓
Policy Decision
      ↓
Account Restriction/Suspension
      ↓
Audit
```

---

# 140. No Automatic Medical Judgment

A complaint stating:

```text
"Doctor gave me the wrong medicine."
```

is a user allegation, not automatically a verified fact.

Admin workflow should:

```text
Receive allegation
      ↓
Preserve evidence
      ↓
Escalate to authorized clinical review when required
      ↓
Separate operational resolution from clinical conclusion
```

---

# 141. Sensitive Data Minimization

Complaint text may contain:

- medical history
- diagnoses
- prescriptions
- addresses
- phone numbers

The platform should encourage users to provide only information relevant to the case and protect whatever they submit.

---

# 142. Data Access Matrix

| Resource | Patient | Doctor/Nurse | Pharmacy/Lab | Support Admin | Clinical Reviewer | Finance Admin |
|---|---|---|---|---|---|---|
| Own review | Full | Full | Full | Scoped | Scoped | No |
| Provider public review | Read | Own | Own org | Yes | Yes | No |
| Own complaint | Full | Full | Full | Yes | Scoped | Scoped |
| Another user's complaint | No | No | No | Scoped | Scoped | Scoped |
| Internal complaint notes | No | No | No | Yes | Yes if assigned | No |
| Financial complaint data | Own | Own | Org scope | Limited | No | Yes |
| Clinical complaint content | Own | Relevant | Relevant | Restricted | Yes when assigned | No |

Exact access depends on final permission policy.

---

# 143. Complaint Authorization Example

A Doctor should access:

```text
Complaint.TargetProviderId == CurrentDoctorId
```

A Pharmacy employee should access:

```text
Complaint.TargetOrganizationId == CurrentPharmacyId
```

A Patient should access:

```text
Complaint.ReporterUserId == CurrentUserId
```

Admin uses explicit permissions.

---

# 144. Review Authorization Example

A patient can edit a review only if:

```text
Review.AuthorUserId == CurrentUserId
AND
Review.BusinessObject is still eligible
AND
EditWindow is open
AND
Review.Status allows edit
```

---

# 145. Complaint Editing Rule

Do not allow the patient to edit the original complaint indefinitely.

Better:

```text
Original Complaint
     ↓
Immutable core description
     ↓
Follow-up messages
```

If editing is needed, preserve revisions.

---

# 146. Complaint Deletion Rule

Do not hard-delete a formal complaint from normal UI.

Use:

```text
Cancelled
Closed
Archived
```

Financial/security/medical audit relationships may require retention.

---

# 147. Review Deletion Rule

Patient may remove their own review only according to product policy.

The system should preserve enough moderation/audit context to identify historical actions when required.

---

# 148. Audit Requirements

Audit:

```text
Review Created
Review Edited
Review Reported
Review Hidden
Review Restored
Provider Response Created
Complaint Created
Complaint Assigned
Complaint Status Changed
Complaint Escalated
Complaint Resolved
Complaint Closed
Complaint Reopened
Admin Note Added
Refund Triggered from Complaint
Account Action Triggered from Complaint
```

---

# 149. Audit Fields

```text
ActorUserId
ActorRole
Action
ResourceType
ResourceId
PreviousState
NewState
ReasonCode
CorrelationId
CreatedAt
```

Do not put full sensitive medical content into every audit event.

---

# 150. Event-Driven Integration

Recommended domain events:

```text
ServiceCompletedEvent
RatingEligibilityCreatedEvent
ReviewCreatedEvent
ReviewReportedEvent
ReviewModeratedEvent
ProviderReviewResponseEvent
ComplaintCreatedEvent
ComplaintAssignedEvent
ComplaintEscalatedEvent
ComplaintResolvedEvent
ComplaintReopenedEvent
```

Financial events:

```text
ComplaintRefundRequestedEvent
ComplaintRefundCompletedEvent
```

Security events:

```text
AbuseFlagRaisedEvent
ProviderSuspensionTriggeredEvent
```

---

# 151. Notification Integration

Example:

```text
ReviewCreated
   ↓
Notification → Provider
```

```text
ComplaintCreated
   ↓
Notification → Admin Queue
```

```text
ComplaintResolved
   ↓
Notification → Reporter
```

The Notification module owns actual delivery.

---

# 152. Outbox Integration

For important events:

```text
Business Transaction
    ↓
Write Review/Complaint state
    ↓
Write Outbox Event
    ↓
Commit
    ↓
Notification/Event Worker
```

This prevents successful complaint creation from losing its notification because an external service is temporarily unavailable.

---

# 153. Background Jobs

Potential jobs:

```text
ReviewPromptWorker
ReviewReminderWorker
ComplaintReminderWorker
ComplaintEscalationWorker
OverdueComplaintWorker
RatingAggregateRepairWorker
AbuseSignalWorker
```

Every job must be idempotent.

---

# 154. Complaint Reminder Deduplication

Example key:

```text
complaint-reminder:{complaintId}:{stage}
```

This avoids duplicate reminders.

---

# 155. Review Prompt Deduplication

Example:

```text
review-prompt:{businessObjectId}:{targetType}
```

One completed appointment should not create multiple identical prompts.

---

# 156. API Idempotency

Important operations:

```text
Create Review
Create Complaint
Add Complaint Message
Moderate Review
Resolve Complaint
Request Appeal
```

Some actions may use request IDs/idempotency keys to protect against duplicate submissions.

---

# 157. Frontend Duplicate Submission

User double-clicks Submit:

```text
Click 1 → Request sent
Click 2 → Request sent
```

Backend should recognize the duplicate logical operation when appropriate.

Frontend should also disable submit while the request is processing.

---

# 158. Complaint Attachment Validation

Validate:

```text
Extension
MIME type
File size
Number of attachments
Storage permissions
```

Potentially dangerous content should be rejected/scanned by the infrastructure chosen for the deployment.

---

# 159. Review Content Safety

Before publishing, apply:

```text
Encoding / sanitization
Length validation
Content policy checks
```

Automated moderation can flag content for human review.

It should not silently invent medical conclusions.

---

# 160. Search Indexing

Provider reviews may be indexed for search, but hidden reviews should be excluded from public provider search/aggregates according to policy.

Complaint text must generally not appear in public search indexes.

---

# 161. Caching

Safe candidates:

```text
Provider rating summary
Public review count
Public recent reviews
```

Do not cache sensitive complaint data broadly.

Cache invalidation must happen when moderation changes visibility.

---

# 162. Performance

Use pagination for:

```text
Provider reviews
Patient review history
Admin complaints
Admin review queues
Complaint messages
```

Do not load thousands of reviews in one request.

---

# 163. Database Indexes

Candidates:

```text
Review(TargetType, TargetId, Status, CreatedAt)
Review(AuthorUserId, CreatedAt)
Review(BusinessObjectId, AuthorUserId)
ReviewReport(Status, CreatedAt)
Complaint(Status, Priority, CreatedAt)
Complaint(TargetType, TargetId, Status)
Complaint(ReporterUserId, CreatedAt)
Complaint(AssignedAdminId, Status)
ComplaintMessage(ComplaintId, CreatedAt)
ComplaintStatusHistory(ComplaintId, CreatedAt)
```

Exact indexes should follow actual query patterns.

---

# 164. Search API

Provider reviews:

```http
GET /api/v1/providers/{id}/reviews?page=1&pageSize=20
```

Patient complaints:

```http
GET /api/v1/complaints/my?page=1&pageSize=20
```

Admin complaints:

```http
GET /api/v1/admin/complaints?status=OPEN&priority=HIGH&page=1&pageSize=20
```

---

# 165. Sorting Reviews

Options:

```text
Newest
Highest Rated
Lowest Rated
Most Helpful (future)
```

Public ranking should not manipulate review visibility without a moderation rule.

---

# 166. Helpful / Useful Review (Future)

A future feature may allow users to mark reviews as helpful.

Do not include this in the MVP unless needed.

It must be protected against mass manipulation.

---

# 167. Provider Response Rate

Useful metric:

```text
Reviews with provider response
÷
Eligible reviews
```

This can be displayed on provider dashboards without affecting rating itself unless product policy explicitly uses it for ranking.

---

# 168. Complaint Resolution Metrics

Track:

```text
Average resolution age
Median resolution age
Open case count
Escalation rate
Reopen rate
Refund outcome rate
```

---

# 169. Provider Complaint History

Provider can see their own case history:

```text
Open
Resolved
Closed
Escalated
```

Do not display internal security investigation details.

---

# 170. Patient Complaint History

Patient can see:

```text
My Complaints
Status
Last Update
Resolution
```

The patient should not see confidential internal Admin notes.

---

# 171. Complaint Resolution Communication

When resolved:

```text
Complaint resolved.
Resolution: [safe summary]
```

Do not expose internal policy details that create security/privacy risk.

---

# 172. Complaint Closure Confirmation

Admin can close case after resolution.

Optionally:

```text
Patient notified
Provider notified
```

The case remains auditable.

---

# 173. Complaint Appeal Rules

To avoid infinite cases:

- appeal allowed only once or within policy
- appeal reason required
- final reviewer may differ from original reviewer
- original decision remains in history

---

# 174. Senior Review

For high-severity cases:

```text
Support Admin
    ↓
Escalate
    ↓
Senior Admin / Specialized Reviewer
    ↓
Final Decision
```

---

# 175. Safety Incident Workflow

For severe incidents:

```text
Complaint / Report
      ↓
Safety classification
      ↓
CRITICAL
      ↓
Immediate notification to authorized team
      ↓
Preserve evidence
      ↓
Containment where necessary
      ↓
Clinical/Security escalation
      ↓
Resolution
      ↓
Post-incident audit
```

---

# 176. Provider Safety Investigation

Possible temporary action:

```text
Provider remains logged in
BUT
New bookings disabled
```

This can be safer than immediately destroying the provider account while investigation is underway.

---

# 177. Patient Safety Investigation

Likewise:

```text
Patient account may remain accessible
BUT
Specific abusive capability can be restricted
```

Use the narrowest necessary restriction when possible.

---

# 178. Feature-Level Restriction

Example:

```text
Account ACTIVE
BloodRequest.Create = disabled
```

This is a useful alternative to full account suspension when abuse is isolated to one feature.

---

# 179. Complaint-based Restriction

Restrictions should be based on:

```text
Verified evidence
Policy
Authorized Admin decision
```

not simply the existence of a complaint.

---

# 180. Rating Visibility Rules

Public visibility can depend on:

```text
Valid transaction
Published status
Not hidden
Not duplicated
```

Do not expose moderation notes publicly.

---

# 181. Search Ranking Integration

Provider ranking may use:

```text
Average rating
Review count
Complaint rate
Completion rate
Distance
Availability
Verification
```

But do not use raw complaint counts without normalizing by transaction volume.

---

# 182. Complaint Impact on Ranking

A verified severe policy violation may affect provider discoverability through an approved policy.

A single unverified complaint should not automatically lower a provider's public ranking.

---

# 183. Rating Summary Rebuild

Provide an internal repair operation:

```http
POST /api/v1/internal/ratings/{targetType}/{targetId}/rebuild
```

or an admin repair workflow.

This is useful if the cached summary becomes inconsistent.

---

# 184. Data Consistency Invariants

The system should enforce:

```text
One eligible rating per eligible transaction/target
```

```text
Visible rating must belong to an eligible experience
```

```text
Complaint must reference a valid reporter
```

```text
Complaint resolution cannot occur from an invalid state
```

```text
Admin cannot execute actions without required permission
```

---

# 185. Test Scenarios — Ratings

```text
Valid completed appointment → rating allowed
Incomplete appointment → rating blocked
Cancelled appointment → rating blocked according to policy
Expired feedback window → rating blocked
Duplicate rating → rejected
Edit own review → allowed only within policy
Edit another user's review → forbidden
Provider response → allowed
Hidden review → excluded from public aggregate
Restored review → included again according to policy
```

---

# 186. Test Scenarios — Reviews

```text
Review with valid text
Review with empty text
Review over max length
Review with XSS payload
Review with personal data
Review report
Review moderation
Review restore
Duplicate review report
Concurrent moderation
```

---

# 187. Test Scenarios — Complaints

```text
Valid complaint
Complaint against unrelated transaction
Duplicate active complaint
Complaint with attachment
Invalid attachment
Provider response
Patient response
Admin assignment
Status transition
Escalation
Resolution
Reopen
Appeal
```

---

# 188. Security Test Scenarios

```text
Patient reads another patient's complaint → denied
Provider reads another provider's complaint → denied
Pharmacy reads another pharmacy's case → denied
Admin without finance permission triggers refund → denied
Internal note returned to patient → must never happen
Private attachment URL guessed → denied
Hidden review visible publicly → must not happen
Deep link used without authorization → denied
```

---

# 189. Concurrency Test Scenarios

```text
Two identical review submissions
Two identical complaint submissions
Two Admins resolve same complaint
Two Admins moderate same review
Concurrent rating aggregate updates
Concurrent appeal submission
```

Expected result must remain consistent.

---

# 190. Abuse Test Scenarios

```text
Rapid repeated complaints
Rapid repeated reviews
Many fake reviews
Provider attempts retaliation
Patient attempts extortion
Repeated duplicate reports
Automated spam submissions
```

System should rate-limit, flag, or route for review according to policy.

---

# 191. Financial Integration Test Scenarios

```text
Complaint leads to approved refund
Complaint resolved without refund
Refund fails
Refund duplicate request
Provider earning adjusted correctly
Historical payment unchanged
```

---

# 192. Clinical Escalation Test Scenarios

```text
Medical-safety complaint
Lab-result complaint
Prescription complaint
Clinical reviewer assignment
Clinical reviewer response
Admin cannot independently alter clinical conclusion
```

---

# 193. Notification Integration Tests

```text
Review created → Provider notified
Complaint created → Admin notified
Admin requests info → Patient notified
Complaint resolved → Patient notified
Moderation decision → Relevant party notified
```

Failure of email/SMS must not roll back the complaint/review database transaction.

---

# 194. Background Job Tests

```text
Review reminder runs once
Duplicate worker run does not duplicate reminder
Complaint overdue job flags case once
Escalation job is idempotent
Notification failure retries safely
```

---

# 195. Recommended Unit Test Structure

```text
RatingEligibilityServiceTests
ReviewServiceTests
ReviewModerationServiceTests
ComplaintServiceTests
ComplaintWorkflowServiceTests
ComplaintResolutionServiceTests
ComplaintEscalationServiceTests
ComplaintAuthorizationTests
RatingAggregationServiceTests
AbuseDetectionTests
```

---

# 196. Recommended API Integration Test Structure

```text
ReviewsApiTests
ComplaintsApiTests
ModerationApiTests
ComplaintAuthorizationApiTests
AttachmentSecurityApiTests
ComplaintConcurrencyApiTests
```

---

# 197. Definition of Done — Rating

```text
[ ] Rating eligibility works
[ ] Rating tied to real completed transaction
[ ] Duplicate rating prevention
[ ] Review text validation
[ ] Review publishing
[ ] Provider response
[ ] Review report
[ ] Moderation
[ ] Rating aggregate
[ ] Aggregate repair strategy
[ ] Authorization
[ ] Audit
[ ] Notifications
[ ] Tests
```

---

# 198. Definition of Done — Complaint

```text
[ ] Complaint creation
[ ] Transaction linkage
[ ] Categories
[ ] Priority
[ ] Attachments
[ ] Messages
[ ] Provider response
[ ] Admin assignment
[ ] Status state machine
[ ] Escalation
[ ] Resolution
[ ] Reopen/appeal if enabled
[ ] Refund integration
[ ] Clinical escalation path
[ ] Security controls
[ ] Audit
[ ] Notifications
[ ] Tests
```

---

# 199. MVP Scope

## Must Have

```text
Rating 1–5
Written review
Completed-service eligibility
One rating per transaction
Provider review viewing
Provider response
Report review
Complaint creation
Complaint categories
Complaint status
Admin complaint queue
Admin assignment
Admin response
Evidence attachments
Complaint resolution
Basic refund integration
Notifications
Audit logs
Authorization
```

## Future

```text
Advanced abuse detection
Automated moderation assistance
Helpful votes
Advanced sentiment analysis
Provider quality score
Advanced analytics
Multi-stage appeals
Specialized clinical review portal
External regulatory case integration
```

---

# 200. Recommended Implementation Order

```text
1. Rating eligibility model
2. Review entity + APIs
3. Review aggregate
4. Provider response
5. Review report/moderation
6. Complaint entity
7. Complaint workflow/state machine
8. Complaint messages
9. Attachments
10. Admin queue/assignment
11. Resolution
12. Notification integration
13. Refund integration
14. Safety/clinical escalation
15. Analytics
16. Abuse signals
17. Advanced appeals
```

---

# 201. Recommended Frontend Module Structure

```text
features/
└── feedback/
    ├── reviews/
    │   ├── pages/
    │   ├── components/
    │   ├── services/
    │   └── models/
    │
    ├── complaints/
    │   ├── pages/
    │   ├── components/
    │   ├── services/
    │   └── models/
    │
    └── moderation/
```

---

# 202. Recommended Backend Module Structure

```text
Application/
└── Feedback/
    ├── Reviews/
    │   ├── Commands
    │   ├── Queries
    │   ├── Validators
    │   └── Services
    │
    ├── Complaints/
    │   ├── Commands
    │   ├── Queries
    │   ├── Validators
    │   └── Services
    │
    └── Moderation/

Domain/
└── Feedback/
    ├── Review
    ├── Complaint
    ├── ValueObjects
    └── Policies
```

---

# 203. Service Boundaries

Suggested services:

```text
IRatingEligibilityService
IReviewService
IReviewModerationService
IComplaintService
IComplaintAssignmentService
IComplaintResolutionService
IComplaintEscalationService
IAbuseSignalService
IRatingAggregationService
```

Finance, Notifications, Identity, and Clinical modules remain separate.

---

# 204. Cross-Module Ownership

```text
Doctor/Nurse/Pharmacy/Lab
    = owns service experience

Feedback module
    = owns ratings/complaints

Finance module
    = owns payments/refunds

Notification module
    = owns message delivery

Auth module
    = owns identity/authorization foundation

Admin module
    = owns administrative operations
```

This prevents the feedback module from becoming a monolith.

---

# 205. Full Patient Rating Journey

```text
Patient completes service
      ↓
ServiceCompletedEvent
      ↓
Rating eligibility generated
      ↓
Notification
      ↓
Patient opens review screen
      ↓
Select stars
      ↓
Write review
      ↓
Submit
      ↓
Backend validates ownership/eligibility
      ↓
Review created
      ↓
Rating aggregate updated
      ↓
Provider notified
      ↓
Review visible according to moderation policy
```

---

# 206. Full Provider Review Journey

```text
Provider receives notification
      ↓
Opens Reviews
      ↓
Reads review
      ↓
Optional response
      ↓
Response validated
      ↓
Response published
      ↓
Patient notified if policy allows
```

---

# 207. Full Patient Complaint Journey

```text
Patient opens completed transaction
      ↓
Report a Problem
      ↓
Select category
      ↓
Description
      ↓
Attach evidence
      ↓
Submit
      ↓
Complaint ID generated
      ↓
Admin queue
      ↓
Assigned
      ↓
Investigation
      ↓
Provider/Finance/Clinical input if needed
      ↓
Resolution
      ↓
Patient notified
      ↓
Case closed
```

---

# 208. Full Provider Complaint Journey

```text
Provider receives complaint notification
      ↓
Open case
      ↓
Review patient complaint
      ↓
Submit response/evidence
      ↓
Admin reviews
      ↓
Resolution
      ↓
Provider notified
```

The provider cannot delete the complaint.

---

# 209. Full Admin Complaint Journey

```text
Admin login
      ↓
Complaint queue
      ↓
Filter by priority/status
      ↓
Open case
      ↓
Review linked transaction
      ↓
Review timeline
      ↓
Review evidence
      ↓
Request information
      ↓
Assign/escalate
      ↓
Choose resolution
      ↓
Trigger authorized downstream workflow
      ↓
Notify parties
      ↓
Audit
      ↓
Close
```

---

# 210. Full Medical-Safety Complaint Journey

```text
Patient reports potential clinical harm
      ↓
Complaint = MEDICAL_SAFETY
      ↓
Priority assessment
      ↓
Immediate safety routing
      ↓
Preserve case/evidence
      ↓
Authorized clinical reviewer
      ↓
Clinical review
      ↓
Admin operational actions
      ↓
Finance/refund if independently justified
      ↓
Provider/account policy review if necessary
      ↓
Resolution
      ↓
Audit
```

---

# 211. Full Privacy Complaint Journey

```text
User reports private data exposure
      ↓
PRIVACY complaint
      ↓
Security event created if needed
      ↓
Access-log review
      ↓
Containment
      ↓
Security/Admin investigation
      ↓
Affected-account protection
      ↓
Resolution
      ↓
Audit
```

---

# 212. Full Rating Moderation Journey

```text
Review published
      ↓
Provider/User reports review
      ↓
Flag created
      ↓
Admin queue
      ↓
Review evidence
      ↓
Decision
 ┌─────┼──────────┐
 ↓     ↓          ↓
Keep  Hide     Restore
 │     │          │
 └─────┴──────────┘
        ↓
Moderation Audit
        ↓
Aggregate recalculated
        ↓
Notification when required
```

---

# 213. Final State Diagram — Review

```text
DRAFT
  ↓
PUBLISHED
  ├── FLAGGED
  │     ↓
  │  UNDER_REVIEW
  │    ├── KEEP_VISIBLE
  │    ├── HIDDEN
  │    └── RESTORED
  │
  └── DELETED_BY_AUTHOR / ARCHIVED
```

---

# 214. Final State Diagram — Complaint

```text
SUBMITTED
   ↓
OPEN
   ↓
ASSIGNED
   ↓
UNDER_REVIEW
   ├── WAITING_FOR_PATIENT
   ├── WAITING_FOR_PROVIDER
   ├── WAITING_FOR_FINANCE
   └── ESCALATED
           ↓
     RESOLUTION_RECOMMENDED
           ↓
        RESOLVED
           ↓
         CLOSED

Possible alternatives:
SUBMITTED → REJECTED
OPEN → DUPLICATE
OPEN → CANCELLED
RESOLVED → REOPENED → UNDER_REVIEW
```

---

# 215. Final State Diagram — Review Report

```text
SUBMITTED
   ↓
OPEN
   ↓
UNDER_REVIEW
 ┌─┴───────────┐
 ↓             ↓
DISMISSED    ACTION_TAKEN
                 ↓
               CLOSED
```

---

# 216. Final Quality Architecture

```text
                       SERVICE COMPLETED
                              │
                    ┌─────────┴──────────┐
                    │                    │
                    ▼                    ▼
                 RATING              COMPLAINT
                    │                    │
                    ▼                    ▼
              REVIEW SYSTEM        CASE MANAGEMENT
                    │                    │
              ┌─────┴─────┐      ┌─────┼──────────┐
              │           │      │     │          │
           Provider     Moderation Finance Clinical Security
           Response       │       │       │         │
              │           │       │       │         │
              └──────┬────┴──────┴───────┴─────────┘
                     ▼
                ADMIN GOVERNANCE
                     │
                     ▼
               AUDIT + ANALYTICS
```

---

# 217. Final Product Principles

```text
1. Real transaction → eligible feedback.
2. One experience → one rating per target.
3. Complaints are formal cases, not comments.
4. Negative reviews are not automatically abusive.
5. Clinical complaints require clinical review when needed.
6. Financial outcomes go through the Financial module.
7. Notifications go through the Notification module.
8. Authentication/authorization comes from the shared Auth module.
9. Every sensitive Admin action is auditable.
10. Private patient/provider information stays private.
11. Historical records are preserved.
12. State transitions are server-controlled.
13. Duplicate and concurrent submissions are safe.
14. Abuse signals are investigated, not blindly treated as proof.
15. Quality metrics should be based on normalized, trustworthy data.
```

---

# 218. Final Definition of Done — Entire Module

```text
[ ] Patient can rate eligible completed service.
[ ] Provider can view own ratings.
[ ] Provider can respond to eligible reviews.
[ ] Users can report abusive reviews.
[ ] Admin can moderate reported reviews.
[ ] Rating aggregate updates correctly.
[ ] Duplicate ratings are blocked.
[ ] Complaint can be created from a real transaction.
[ ] Complaint has category/priority/status.
[ ] Complaint can include attachments.
[ ] Provider can respond.
[ ] Admin can assign and investigate.
[ ] Admin can request more information.
[ ] Admin can escalate.
[ ] Resolution can be recorded.
[ ] Refund integration works without direct finance mutation.
[ ] Clinical-safety escalation path exists.
[ ] Complaint notifications work.
[ ] Review notifications work.
[ ] Sensitive data access is restricted.
[ ] Object-level authorization is tested.
[ ] Concurrency is handled.
[ ] Idempotency/deduplication is implemented.
[ ] Audit logs exist.
[ ] Security/abuse scenarios are tested.
[ ] Frontend handles loading/error/empty states.
[ ] APIs are documented.
[ ] Database indexes are reviewed.
[ ] End-to-end workflow passes.
```

---

# 219. Final TechCare Integration Map

```text
PATIENT
  │
  ├── completes Appointment
  ├── completes Nurse Visit
  ├── completes Pharmacy Order
  └── completes Laboratory Service
              │
              ▼
        RATING ELIGIBILITY
              │
        ┌─────┴──────┐
        ▼            ▼
      REVIEW       COMPLAINT
        │            │
        │      ┌─────┼────────────┐
        │      ▼     ▼            ▼
        │   Support Finance    Clinical
        │      │       │            │
        │      └───────┼────────────┘
        │              │
        ▼              ▼
   Provider        Resolution
   Feedback            │
        │              ▼
        └──────────> AUDIT
                       │
                       ▼
                   ANALYTICS
```

---

# 220. Final Outcome

The TechCare Rating & Complaint system should provide a trustworthy quality loop:

```text
REAL SERVICE
    ↓
REAL CUSTOMER EXPERIENCE
    ↓
RATING / REVIEW
    ↓
QUALITY SIGNAL

OR

PROBLEM
    ↓
COMPLAINT
    ↓
INVESTIGATION
    ↓
RESOLUTION
    ↓
PREVENTION / IMPROVEMENT
```

The most important architectural separation is:

```text
Rating & Complaint
        ≠
Payment
        ≠
Authentication
        ≠
Clinical Decision
        ≠
Notification Delivery
```

They communicate through explicit events and service contracts.

---

# 221. Developer Handoff Checklist

A developer should be able to answer all of the following before implementation:

```text
Who can rate?
What makes a transaction eligible?
How many ratings can one transaction receive?
Can the review be edited?
How are reviews moderated?
Who can report a review?
Who can create a complaint?
Must a complaint reference a transaction?
What are complaint categories?
How are priorities assigned?
Who can respond?
Who can see internal notes?
How are attachments protected?
How is the complaint assigned?
What are valid status transitions?
How does escalation work?
How does refund integration work?
When is clinical review required?
How is abuse detected?
How are ratings aggregated?
How are hidden reviews handled?
How are duplicate submissions prevented?
How are concurrent Admin actions handled?
How are notifications generated?
How is everything audited?
```

This document provides the baseline answers and implementation boundaries for the TechCare Rating & Complaint module.

---

# END OF DOCUMENT

**TechCare — Rating & Complaint Complete Workflow**
