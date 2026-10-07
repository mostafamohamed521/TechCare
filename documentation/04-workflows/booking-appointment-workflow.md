# 📅 TechCare — Booking & Appointment Workflow

> **This document defines the complete booking and appointment lifecycle in TechCare, from service discovery and provider selection to request creation, provider response, appointment scheduling, service execution, completion, cancellation, expiration, rescheduling, payment, notifications, and post-service actions.**

---

# 1. Workflow Purpose

The Booking & Appointment Workflow is one of the core workflows of TechCare.

It connects the patient with an eligible healthcare provider and coordinates the complete service lifecycle.

The workflow acts as a bridge between multiple TechCare modules:

```text
Patient
   ↓
Search & Discovery
   ↓
Provider
   ↓
Service
   ↓
Location
   ↓
Availability
   ↓
Booking Request
   ↓
Provider Response
   ↓
Appointment
   ↓
Healthcare Service
   ↓
Completion
   ↓
Payment
   ↓
Medical / Service Record
   ↓
Rating / Review

The booking system must provide a clear, secure, consistent, and traceable process.

2. Main Objectives

The Booking & Appointment system aims to:

Allow patients to request healthcare services.
Connect patients with suitable providers.
Validate provider eligibility.
Validate service eligibility.
Validate provider availability.
Validate the requested appointment time.
Calculate applicable pricing.
Create booking requests.
Notify providers.
Allow providers to accept or reject requests.
Automatically expire unanswered requests when configured.
Prevent conflicting bookings.
Support cancellation according to business rules.
Support rescheduling where implemented.
Track appointment lifecycle.
Track service progress.
Mark services as completed.
Trigger post-completion workflows.
Integrate with payment.
Integrate with notifications.
Integrate with ratings and reviews.
Integrate with medical/service records.
3. Actors

The main actors involved are:

Patient
Doctor
Nurse
Laboratory
Pharmacy / Pharmacy Provider
Admin
System
Notification Service
Payment Service

Not every actor participates in every booking type.

4. Supported Booking Types

Depending on the active project scope, TechCare may support different healthcare service types.

Examples:

Doctor Consultation
Nurse Home Visit
Laboratory Service
Laboratory Home Collection
Other Supported Healthcare Services

The booking engine should share common lifecycle concepts while allowing service-specific rules.

5. Booking Lifecycle Overview

The general lifecycle is:

Service Discovery
      ↓
Provider Selection
      ↓
Service Selection
      ↓
Location Confirmation
      ↓
Availability Check
      ↓
Time Slot Selection
      ↓
Price Calculation
      ↓
Booking Request
      ↓
Provider Notification
      ↓
Provider Response
   ┌────┼────────┐
   ↓    ↓        ↓
Accept Reject  No Response
   │      │        │
   │      │        ▼
   │      │      Expired
   │      │
   ▼      ▼
Accepted Rejected
   │
   ▼
Scheduled
   │
   ▼
In Progress
   │
   ▼
Completed
   │
   ├────────────► Payment
   │
   ├────────────► Medical / Service Record
   │
   ├────────────► Notification
   │
   └────────────► Rating / Review
6. Booking Status Model

The booking system should use explicit statuses.

Main States
Pending
Accepted
Scheduled
In Progress
Completed
Alternative / Terminal States
Rejected
Cancelled
Expired
Failed
No-Show

The exact status set should be finalized according to the business requirements.

7. Status Definitions
7.1 Pending

The patient has created a request and is waiting for the provider's response.

Booking Created
      ↓
Pending
7.2 Accepted

The provider has accepted the request.

Pending
   ↓
Accepted
7.3 Scheduled

The appointment has been confirmed and associated with a specific date/time.

Accepted
   ↓
Scheduled
7.4 In Progress

The healthcare service has started.

Scheduled
   ↓
In Progress
7.5 Completed

The healthcare service has been successfully completed.

In Progress
   ↓
Completed
7.6 Rejected

The provider declined the request.

Pending
   ↓
Rejected
7.7 Cancelled

The booking was cancelled by an authorized actor according to business rules.

Pending / Accepted / Scheduled
          ↓
       Cancelled

The allowed cancellation states depend on the configured policy.

7.8 Expired

The request exceeded its response window without the required provider response.

Pending
   ↓
Timeout
   ↓
Expired
7.9 No-Show

An appointment was scheduled but the expected participant did not attend or the service did not occur according to the applicable business rule.

8. Booking State Machine

A simplified state machine:

                        ┌───────────┐
                        │  Pending  │
                        └─────┬─────┘
                              │
                 ┌────────────┼────────────┐
                 │            │            │
                 ▼            ▼            ▼
             Accepted      Rejected      Expired
                 │
                 ▼
             Scheduled
                 │
          ┌──────┴──────┐
          │             │
          ▼             ▼
      Cancelled     In Progress
                        │
                        ▼
                    Completed

Possible additional transitions may exist for:

Rescheduled
No-Show
Failed
9. Booking Preconditions

Before a booking request can be created, the system should verify the following where applicable:

Patient is authenticated
Provider exists
Provider account is eligible
Provider is allowed to receive requests
Service exists
Service is active
Provider offers selected service
Provider location is available where required
Patient location is available where required
Provider serves patient's location where required
Selected time slot is valid
Selected time slot is available
Price can be calculated
Patient is eligible to create the request
10. Step 1 — Patient Authentication

The patient must be authenticated before creating a protected booking request.

Patient
   ↓
Authenticated?
   /      \
 Yes       No
  │         │
  ▼         ▼
Continue   Reject

The backend must verify authentication even if the frontend hides booking actions for unauthenticated users.

11. Step 2 — Service Discovery

The patient starts the process by searching for a healthcare service.

Examples:

Doctor
Nurse
Laboratory
Pharmacy-related service
Other supported service

The patient may use:

Specialty
Service
Location
Distance
Rating
Price
Availability

This functionality is documented in:

search-discovery-workflow.md
12. Step 3 — Provider Selection

The patient selects a provider from the search results.

The system should verify:

Provider exists
Provider is active
Provider is eligible
Provider offers the selected service
Provider is not suspended from the relevant service

If the provider is not eligible, the booking must not proceed.

13. Step 4 — Service Selection

The patient selects the exact service to request.

Example:

Doctor
   ↓
Specialty
   ↓
Consultation Service

Or:

Nurse
   ↓
Nursing Service
   ↓
Home Visit

Or:

Laboratory
   ↓
Test
   ↓
Laboratory Service
14. Step 5 — Location Confirmation

For location-sensitive services, the patient must provide or confirm the service location.

Possible inputs:

Current Location
Saved Address
Manual Address
Map-selected Location

The system may use:

Patient Location
      +
Provider Location
      +
Service Radius
      ↓
Eligibility
15. Step 6 — Service Area Validation

For providers offering home services, the system checks whether the requested location falls within the provider's service area.

Patient Location
      ↓
Distance Calculation
      ↓
Provider Service Radius
      ↓
Within Allowed Area?
    /          \
  Yes           No
   │             │
   ▼             ▼
Continue       Reject / Not Eligible
16. Step 7 — Availability Validation

The system checks provider availability.

It must consider:

Working hours.
Available slots.
Existing bookings.
Blocked periods.
Service-specific availability.

Example:

Requested Slot
      ↓
Provider Available?
     /       \
   Yes        No
    │          │
    ▼          ▼
Continue     Reject
17. Step 8 — Time Slot Selection

The patient selects a supported available slot.

The system must ensure the slot is still available at the time the booking request is processed.

This is important because another patient may attempt to reserve the same slot at the same time.

18. Step 9 — Price Calculation

The system calculates the applicable price.

Possible model:

Base Service Fee
      +
Distance / Travel Fee
      +
Other Configured Charges
      =
Estimated / Final Amount

The exact calculation depends on the service configuration.

19. Example of Distance Pricing

A provider may configure:

Base Service Fee = configured amount
Travel Fee = configured amount per kilometer

The system can therefore calculate:

Service Price
+
Applicable Travel Charge
=
Total Amount

The actual values should be configurable rather than hard-coded throughout the application.

20. Step 10 — Show Booking Summary

Before submitting the request, the patient should receive a summary.

The summary may contain:

Provider
Service
Date
Time
Location
Distance
Base Price
Travel Fee
Other Charges
Final / Estimated Amount
Cancellation Rules where applicable

The patient reviews the information before confirmation.

21. Step 11 — Create Booking Request

After confirmation, the system creates the booking/request.

Conceptually:

Patient
Provider
Service
Location
Date
Time
Price
Request Metadata
       ↓
Booking Request

The initial status is generally:

Pending
22. Booking Request Uniqueness

The system should prevent unintended duplicate requests.

Example:

Patient
   ↓
Same Provider
   ↓
Same Service
   ↓
Same Time

If an equivalent active request already exists, the system should handle the duplicate according to business rules.

23. Step 12 — Provider Notification

After request creation, the provider receives a notification.

Booking Created
      ↓
Notification Service
      ↓
Provider Notification

The notification may contain:

New request indicator.
Service type.
Appointment date/time.
Relevant request information.

Sensitive information should not be unnecessarily exposed through notifications.

24. Step 13 — Provider Reviews Request

The provider opens the request details.

The provider can review permitted information such as:

Patient information required for the request.
Service.
Requested date/time.
Location information required for the service.
Applicable price.
Other relevant information.

Access must follow authorization rules.

25. Step 14 — Provider Accepts

If the provider accepts the request:

Pending
   ↓
Accepted

The system should:

Validate that the request is still active.
Re-check availability if required.
Prevent conflicting booking changes.
Update booking status.
Confirm appointment information.
Notify the patient.
26. Step 15 — Provider Rejects

If the provider rejects the request:

Pending
   ↓
Rejected

The system should:

Validate request state.
Update status.
Store rejection information where required.
Notify the patient.

The patient can then search for another provider.

27. Step 16 — Request Expiration

If the provider does not respond within the configured response window:

Pending
   ↓
Response Timeout
   ↓
Expired

The system should:

Change status to Expired.
Prevent normal acceptance of the expired request.
Record the expiration event.
Notify the patient where appropriate.
Make the patient free to search for another provider.
28. Expiration Timing

The request timeout should be treated as a configurable business rule.

Example:

Request Created
      ↓
Timer Starts
      ↓
Provider Responds?
   /            \
 Yes             No
  │               │
  ▼               ▼
Continue       Timeout
                  ↓
               Expired

The exact timeout value should not be repeated as a hard-coded constant throughout the application.

29. Concurrency and Race Conditions

Booking is a concurrency-sensitive operation.

Example:

Patient A ─────┐
               ├──► Same Provider Slot
Patient B ─────┘

Both users may attempt to book the same slot at nearly the same time.

The backend must guarantee that the final data state remains valid.

Possible protections include:

Database constraints.
Transactions.
Proper locking/concurrency strategy.
Atomic state transitions.
Duplicate-request prevention.
30. Prevent Double Booking

The system shall not allow conflicting bookings for the same provider and time according to scheduling rules.

Example:

Doctor
   │
10:00 AM
   │
Patient A → Booked
   │
Patient B → ❌ Cannot create conflicting booking

The exact conflict rules depend on appointment duration and service configuration.

31. Appointment Confirmation

Once the provider accepts the request and required validations succeed, the appointment becomes confirmed.

Conceptually:

Pending
   ↓
Accepted
   ↓
Scheduled

The appointment should contain the required data for the encounter.

Examples:

Patient
Provider
Service
Date
Time
Location
Booking Reference
Price
Status
32. Appointment Reminder

Before the appointment, the system can generate reminders.

Scheduled Appointment
        ↓
Reminder Time
        ↓
Notification
        ↓
Patient / Provider

The exact reminder timing should be configurable.

33. Appointment Day

On the scheduled appointment day, the relevant provider can start the service.

Scheduled
   ↓
Service Starts
   ↓
In Progress

The system should only allow the transition when the booking is in a valid state.

34. Start Service

An authorized provider can start an eligible service.

Examples:

Doctor
Scheduled
   ↓
Consultation Starts
Nurse
Scheduled
   ↓
Home Visit Starts
Laboratory
Scheduled
   ↓
Laboratory Service Starts
35. In Progress State

While the service is being performed:

In Progress

The system can associate relevant service data with the active encounter.

36. Service Completion

After the service is successfully provided:

In Progress
   ↓
Completed

The completion event can trigger several actions.

37. Completion Actions

After completion, the system may:

Service Completed
       │
       ├──► Update Booking
       │
       ├──► Update Financial Transaction
       │
       ├──► Update Provider Earnings
       │
       ├──► Create / Update Medical Record
       │
       ├──► Create Notification
       │
       └──► Enable Rating

The exact actions depend on the service type.

38. Doctor Completion

For a doctor consultation:

Consultation
    ↓
Diagnosis / Treatment
    ↓
Medical Record
    ↓
Complete Appointment

The relevant information is stored according to medical-record authorization rules.

39. Nurse Completion

For a nursing service:

Home Visit
    ↓
Service Performed
    ↓
Completion
    ↓
Financial Update
    ↓
Rating Eligibility

The nurse does not automatically receive doctor-only clinical functions.

40. Laboratory Completion

For a laboratory service:

Laboratory Service
      ↓
Sample / Test Processing
      ↓
Service Completion
      ↓
Result Workflow
      ↓
Patient Notification

Where result functionality exists.

41. Booking Cancellation

Cancellation is allowed only when the current state and business rules permit it.

Possible cancellation sources:

Patient
Provider
Admin
System

The allowed actor depends on the booking state and cancellation policy.

42. Patient Cancellation

A patient may cancel an eligible booking.

Example:

Scheduled
   ↓
Patient Requests Cancellation
   ↓
Validate Cancellation Rules
   ↓
Cancelled

The system may also need to process:

Notifications.
Financial effects.
Refund rules where applicable.
Provider availability restoration.
43. Provider Cancellation

A provider may cancel in an eligible state according to platform rules.

Possible effects:

Patient notification.
Booking cancellation.
Availability handling.
Financial adjustment if applicable.
Complaint eligibility where appropriate.
44. System Cancellation

The system may automatically cancel a booking when business or operational rules require it.

Examples:

Request expiration.
Invalidated provider.
Service cancellation.
Administrative intervention.
45. Cancellation Restrictions

The system should not allow cancellation in every state.

Example:

Completed
   ↓
Cancel
❌ Invalid

The exact allowed states must be defined by business rules.

46. Rescheduling

Where rescheduling is supported, the flow is:

Existing Appointment
        ↓
Request Reschedule
        ↓
Validate Eligibility
        ↓
Find New Availability
        ↓
Select New Slot
        ↓
Confirm
        ↓
Update Appointment
        ↓
Notify Relevant Users
47. Rescheduling Validation

Before rescheduling, the system should validate:

Current booking state.
Patient eligibility.
Provider availability.
New time slot.
Cancellation/rescheduling policy.
Pricing changes where applicable.
48. Rescheduling and Price Recalculation

If the new appointment changes the service location or pricing conditions, the system may need to recalculate the amount.

Example:

Old Location
   ↓
Old Distance
   ↓
Old Price

New Location
   ↓
New Distance
   ↓
New Price

The patient should receive clear information about any applicable price difference.

49. No-Show Handling

A booking may become a No-Show when the expected service does not occur due to absence.

Possible flow:

Scheduled
   ↓
Appointment Time Passed
   ↓
Service Did Not Occur
   ↓
No-Show

No-show rules should be defined separately for:

Patient no-show.
Provider no-show.

Financial and complaint effects may apply according to business rules.

50. Provider Availability After Booking

When a slot becomes booked:

Available Slot
      ↓
Booking Confirmed
      ↓
Slot Becomes Unavailable

The system must ensure that other patients do not see the same slot as freely available when it is no longer bookable.

51. Availability Restoration

When an eligible booking is cancelled:

Booked Slot
     ↓
Cancellation
     ↓
Slot May Become Available

This depends on:

Cancellation rules.
Timing.
Provider availability.
Business logic.
52. Booking and Notification Integration

Important booking events should trigger notifications.

Examples:

Request Created
      ↓
Provider Notification

Request Accepted
      ↓
Patient Notification

Request Rejected
      ↓
Patient Notification

Request Expired
      ↓
Patient Notification

Appointment Confirmed
      ↓
Patient + Provider Notification

Appointment Reminder
      ↓
Patient + Provider Notification

Service Completed
      ↓
Patient Notification
53. Booking and Payment Integration

Booking may interact with payment depending on the service.

Possible flow:

Booking
   ↓
Price Calculation
   ↓
Payment Requirement
   ↓
Payment
   ↓
Booking Confirmation

or:

Booking
   ↓
Service
   ↓
Completion
   ↓
Payment

The exact flow depends on the payment policy.

54. Payment Failure

If payment is required before confirmation and payment fails:

Booking Request
      ↓
Payment
      ↓
Failed

The system must not incorrectly mark the booking as successfully paid.

The booking state should follow defined failure rules.

55. Booking and Medical Records Integration

Doctor-related completed consultations may trigger medical record operations.

Completed Consultation
       ↓
Consultation Record
       ↓
Medical Record

Only authorized healthcare information should be created or exposed.

56. Booking and Rating Integration

A completed eligible service can become rateable.

Booking
   ↓
Completed
   ↓
Rating Eligible
   ↓
Patient Can Rate

Ratings should not be available for invalid or unrelated interactions.

57. Booking and Complaint Integration

If a problem occurs during a booking/service lifecycle, the patient or provider may be able to submit a complaint.

Booking / Service
      ↓
Issue
      ↓
Complaint
      ↓
Admin Review
58. Booking History

Patients and providers should be able to view relevant booking history.

Patient history may include:

Previous services.
Appointment date.
Provider.
Service.
Status.
Price.
Completion state.

Provider history may include:

Received requests.
Accepted bookings.
Completed services.
Cancelled services.
Financial information related to eligible bookings.
59. Access Control for Bookings

Users should only access bookings they are authorized to view.

Patient

Can access their own bookings.

Provider

Can access bookings relevant to their services.

Admin

Can access bookings according to administrative permissions.

Other users:

❌ No Access
60. Booking Data Ownership

The system should clearly distinguish:

Booking Owner / Requester
Provider
Service
Appointment
Financial Data
Medical Data

Not all booking-related information is visible to all participants.

61. Sensitive Booking Information

Depending on the workflow, booking data may contain sensitive information such as:

Patient address.
Precise service location.
Healthcare context.
Financial information.
Medical information.

The system should expose only the information necessary for the relevant operation.

62. Booking and Provider Verification

A provider should only receive bookings according to their verification and account status.

Example:

Provider
   ↓
Verification Status
   ↓
Approved?
   /    \
 Yes     No
  │       │
  ▼       ▼
Eligible Not Eligible

A suspended or rejected provider should not receive restricted booking functionality.

63. Booking Eligibility Rules

A booking can be considered eligible when:

Patient is authorized
AND
Provider is active
AND
Provider is verified where required
AND
Service is active
AND
Provider offers service
AND
Location is serviceable where required
AND
Time slot is valid
AND
Time slot is available
AND
Pricing can be calculated
AND
No conflicting booking exists
64. Invalid Booking Scenarios

The booking request should be rejected when:

Patient is not authenticated.
Provider does not exist.
Provider is inactive.
Provider is not verified where verification is required.
Service does not exist.
Service is disabled.
Provider does not offer the service.
Location is outside service area.
Time slot is invalid.
Time slot is unavailable.
Booking conflicts with another appointment.
Request violates business rules.
Required payment fails where payment is mandatory.
65. Booking Error Examples

Examples of errors include:

Provider Not Found
Provider Unavailable
Service Not Found
Service Unavailable
Invalid Time Slot
Slot Already Booked
Outside Service Area
Booking Not Allowed
Request Already Exists
Payment Failed
Booking Already Cancelled
Booking Already Completed
66. Booking Notifications Matrix
Event	Patient	Provider	Admin
Request Created	✅	✅	Optional
Request Accepted	✅	✅	Optional
Request Rejected	✅	✅	Optional
Request Expired	✅	Optional	Optional
Appointment Confirmed	✅	✅	Optional
Appointment Reminder	✅	✅	Optional
Cancellation	✅	✅	Optional
Service Completed	✅	✅	Optional
Payment Update	✅	✅	Optional
Complaint Created	✅	✅	✅
67. Booking Timeline Example

Example of a normal doctor appointment:

10:00
Patient Searches

10:02
Doctor Selected

10:03
Availability Checked

10:04
Price Calculated

10:05
Booking Request Created

10:05
Doctor Notified

10:07
Doctor Accepts

10:07
Appointment Confirmed

09:30 Next Day
Reminder Sent

10:00
Consultation Starts

10:45
Consultation Completed

10:46
Medical Record Updated

10:47
Payment / Transaction Updated

10:47
Rating Becomes Available

The exact times are only an example of the sequence.

68. Home Nursing Booking Example
Patient
   ↓
Search Nurse
   ↓
Location
   ↓
Service Radius
   ↓
Nursing Service
   ↓
Availability
   ↓
Price
   ↓
Booking Request
   ↓
Nurse Notification
   ↓
Accept
   ↓
Appointment
   ↓
Home Visit
   ↓
Complete Service
   ↓
Payment
   ↓
Rating
69. Laboratory Booking Example
Patient
   ↓
Search Test
   ↓
Laboratory
   ↓
Service
   ↓
Location
   ↓
Availability
   ↓
Booking
   ↓
Laboratory Acceptance
   ↓
Appointment
   ↓
Test
   ↓
Result Workflow
   ↓
Notification
70. Alternative Booking Path

A patient may not always complete a booking with the first provider.

Example:

Search Provider A
       ↓
Request
       ↓
Provider A Rejects
       ↓
Search Provider B
       ↓
Request
       ↓
Provider B Accepts
       ↓
Appointment

This is an important part of the user experience.

71. Expired Booking Path
Search Provider
      ↓
Request
      ↓
Pending
      ↓
No Provider Response
      ↓
Timeout
      ↓
Expired
      ↓
Patient Searches Again
72. Cancelled Booking Path
Booking
   ↓
Scheduled
   ↓
Patient / Provider Cancellation
   ↓
Cancellation Validation
   ↓
Cancelled
   ↓
Notifications
   ↓
Financial Adjustment if Applicable
   ↓
Availability Update if Applicable
73. Completed Booking Path
Scheduled
    ↓
In Progress
    ↓
Completed
    ↓
Financial Update
    ↓
Medical / Service Record
    ↓
Notification
    ↓
Rating
74. Booking Security Rules

The booking module must protect against:

Unauthorized booking creation.
Unauthorized booking modification.
Access to another patient's bookings.
Access to another provider's bookings.
Manipulation of booking IDs.
Bypassing provider verification.
Price manipulation from frontend.
Invalid status changes.
Double booking.
Duplicate request submission.
75. Backend Must Be Authoritative

The frontend may display:

Available

but the backend must verify availability again.

The frontend may display:

Price = X

but the backend must calculate or validate the authoritative price.

The frontend may display:

Accept Request

but the backend must verify the provider is authorized to accept that request.

76. Price Integrity

The system should not trust a price sent from the browser.

Incorrect:

Frontend
   ↓
Price = 100 EGP
   ↓
Backend accepts it

Correct:

Frontend
   ↓
Booking Data
   ↓
Backend Pricing Rules
   ↓
Authoritative Price
77. Status Integrity

The system should not trust a status sent from the frontend.

Incorrect:

Frontend
   ↓
Status = Completed
   ↓
Backend saves Completed

Correct:

Frontend Action
      ↓
Authorization
      ↓
Current Status Validation
      ↓
Business Rule
      ↓
Allowed Transition?
      ↓
Update Status
78. Booking Auditability

Important booking state changes should be traceable where required.

Examples:

Created
Accepted
Rejected
Expired
Cancelled
Rescheduled
Started
Completed

Audit information can help understand:

Who performed the action.
When it happened.
What state changed.
What triggered the change.
79. Automated System Actions

The booking workflow includes system-driven actions.

Examples:

Request Timeout
      ↓
System → Expire Request
Appointment Time
      ↓
System → Reminder
Service Completion
      ↓
System → Enable Rating
80. Background Processing

Some booking operations may be better handled asynchronously or by background processes.

Potential examples:

Expiring old requests.
Sending reminders.
Sending notifications.
Processing delayed financial actions.

The exact implementation depends on the backend infrastructure.

81. Booking Consistency

The system should maintain consistency across:

Booking
Appointment
Availability
Payment
Notifications
Medical / Service Records
Ratings

A critical event should not leave related entities in contradictory states.

82. Example of Invalid State

Bad state:

Booking = Cancelled
Appointment = Active
Provider Slot = Still Booked
Payment = Completed

The application should define how these related states are updated and reconciled.

83. Booking and Transactions

When a booking operation changes multiple related records, appropriate transaction boundaries should be used.

Examples:

Accept Booking
   +
Reserve Slot
   +
Update Appointment

or:

Complete Service
   +
Update Financial Record
   +
Provider Earnings
84. Booking API Responsibilities

The booking API should support operations such as:

Create Booking
Get Booking
List Bookings
Accept Booking
Reject Booking
Cancel Booking
Reschedule Booking
Start Service
Complete Service

The exact endpoints are documented separately in the API documentation.

85. Booking Database Responsibilities

The database must store enough information to support:

Request lifecycle.
Appointment.
Provider.
Patient.
Service.
Time.
Location.
Status.
Pricing.
Financial relationships.
Completion.
Cancellation.
Audit information where required.

The exact schema is documented separately.

86. Booking and Search Integration

Search provides the provider.

Search
   ↓
Provider
   ↓
Booking

Booking must revalidate the provider instead of trusting stale search results.

87. Booking and Location Integration

Location provides:

Patient Location
Provider Location
Distance
Service Radius

Booking uses this information to determine eligibility and pricing when applicable.

88. Booking and Availability Integration

Availability determines whether the provider can serve the requested time.

Provider
   ↓
Availability
   ↓
Booking

Booking should always revalidate the latest availability.

89. Booking and Payment Integration

Payment uses booking/service information to calculate or process applicable financial events.

Booking
   ↓
Pricing
   ↓
Payment
   ↓
Transaction
90. Booking and Notification Integration

Every important state transition can trigger notification events.

Booking State Change
      ↓
Notification Event
      ↓
Relevant User
91. Booking and Rating Integration

Only eligible completed services should become rateable.

Completed Booking
      ↓
Rating Eligibility
      ↓
Rating
92. Booking and Complaint Integration

Users can report problems related to eligible bookings and services.

Booking
   ↓
Problem
   ↓
Complaint
93. Booking and Medical Record Integration

Doctor consultations may produce healthcare records:

Booking
   ↓
Consultation
   ↓
Diagnosis / Treatment
   ↓
Medical Record

The booking system itself should not expose medical information unnecessarily.

94. Cancellation Financial Considerations

Cancellation may affect:

Payment.
Refund.
Provider earnings.
Transaction state.

These rules must be defined by the financial workflow and applied consistently.

95. Appointment Reminder Rules

Reminder generation should consider:

Appointment status.
Appointment date/time.
User role.
Reminder configuration.
Cancellation state.

Cancelled or expired appointments should not continue receiving normal appointment reminders.

96. Expiration and Notification Rules

When a request expires:

Pending
   ↓
Expired

The provider should no longer be able to accept it through the standard flow.

The patient should receive an appropriate update.

97. Provider Rejection and Search Continuation

After rejection:

Rejected
   ↓
Patient Notification
   ↓
Back to Search

The system should make it easy for the patient to continue the discovery process.

98. Appointment Data Visibility

Different users may see different booking fields.

Patient

May see:

Provider.
Service.
Date/time.
Location relevant to their service.
Price.
Status.
Provider

May see:

Patient information required for the service.
Appointment details.
Relevant location.
Service.
Price and financial information where applicable.
Admin

May have broader access according to administrative permissions.

99. Booking UX Principles

The booking experience should be:

Clear.
Simple.
Predictable.
Transparent.
Secure.

The patient should understand:

What service?
Which provider?
When?
Where?
How much?
What happens next?
100. Complete Booking Workflow

The complete TechCare booking workflow is:

┌───────────────────────┐
│ Patient               │
│ Authenticated         │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Search Service        │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Select Provider       │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Select Service         │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Confirm Location       │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Validate Service Area  │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Check Availability     │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Select Time Slot       │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Calculate Price        │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Booking Summary        │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Create Request         │
│ Status = Pending       │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ Notify Provider        │
└───────────┬───────────┘
            ↓
      ┌─────┴─────┐
      ↓           ↓
   Accept       Reject
      │           │
      ▼           ▼
 Scheduled      Rejected
      │
      │
      ├───────────────► No Response
      │                      ↓
      │                   Expired
      │
      ▼
 Appointment
      │
      ▼
 In Progress
      │
      ▼
 Completed
      │
 ┌────┼───────┬────────────┐
 ▼    ▼       ▼            ▼
Pay  Record  Notify      Rating
101. Booking Workflow Checklist

Before creating a booking:

[ ] Patient authenticated
[ ] Provider exists
[ ] Provider active
[ ] Provider verified if required
[ ] Service exists
[ ] Service active
[ ] Provider offers service
[ ] Location available if required
[ ] Service area valid
[ ] Slot exists
[ ] Slot available
[ ] No booking conflict
[ ] Price calculated
[ ] Booking rules satisfied

Before accepting:

[ ] Request is Pending
[ ] Provider is still eligible
[ ] Request has not expired
[ ] Slot is still valid
[ ] No conflict exists
[ ] Provider is authorized

Before completion:

[ ] Booking is in valid state
[ ] Provider is authorized
[ ] Service can be completed
[ ] Completion data is valid
102. Definition of Done — Booking

The Booking feature should not be considered complete until the relevant parts are implemented:

[ ] Booking UI
[ ] Booking API
[ ] Authentication
[ ] Authorization
[ ] Validation
[ ] Provider eligibility
[ ] Service validation
[ ] Availability
[ ] Location
[ ] Pricing
[ ] Request creation
[ ] Accept / Reject
[ ] Expiration
[ ] Cancellation
[ ] Appointment
[ ] Completion
[ ] Notifications
[ ] Financial integration
[ ] Medical/service record integration
[ ] Rating integration
[ ] Error handling
[ ] Concurrency protection
[ ] Tests
[ ] Documentation
103. Cross-Documentation References

The Booking & Appointment Workflow interacts with:

authentication-workflow.md
authorization-workflow.md
search-discovery-workflow.md
location-geo-workflow.md
payment-workflow.md
notification-workflow.md
rating-workflow.md
complaint-workflow.md
medical-records-consultation-workflow.md
provider-verification-workflow.md

These workflows should remain consistent with the booking lifecycle.

104. Core Business Principle

The Booking module is not simply a calendar.

It is a coordination system connecting:

Patient
   +
Provider
   +
Service
   +
Location
   +
Availability
   +
Time
   +
Price
   +
Notifications
   +
Payment
   +
Healthcare / Service Record
   +
Rating
105. Final Workflow Principle

The most important principle is:

A booking is a controlled lifecycle, not a single database record.

Every important transition must be:

Validated.
Authorized.
Traceable.
Consistent.
Secure.
Integrated with the related platform services.
🩺 TechCare

Discover → Request → Confirm → Serve → Complete → Record → Pay → Review