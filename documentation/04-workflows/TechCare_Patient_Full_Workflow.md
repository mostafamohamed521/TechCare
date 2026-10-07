# TechCare — Patient Full Workflow

> **Document type:** Product / UX / Functional Specification
>
> **Role:** Patient
>
> **Prototype:** Web Application
>
> **Prototype domains:** Patient, Doctor, Nurse, Pharmacy, Laboratory, Blood Donation, Admin
>
> **Future:** AI Assistant, AI Triage, Emergency integrations, Telemedicine, Advanced Analytics, Mobile Apps, External Hospital/National Integrations

---

# 1. Purpose

This document defines the complete patient-side workflow in TechCare, from the first visit to the platform through account creation, profile completion, medical information, location, healthcare discovery, booking, consultation, prescriptions, pharmacy orders, laboratory services, blood-donation requests, notifications, payments, reviews, complaints, history, privacy, and logout.

The goal is to make the patient experience:

- **Simple:** clear screens and few unnecessary steps.
- **Safe:** sensitive health information is controlled and protected.
- **Connected:** all completed healthcare activities contribute to an organized patient journey.
- **Traceable:** important requests, records, payments, and outcomes have clear status and timestamps.

---

# 2. Product Principle: Patient at the Center

```text
                    TECHCARE
                       │
                  ┌────┴────┐
                  │ PATIENT │
                  └────┬────┘
                       │
       ┌───────────────┼────────────────┐
       │               │                │
     Doctor          Nurse           Pharmacy
       │               │                │
       ▼               ▼                ▼
 Consultation       Home Care        Medicines
       │                                │
       ▼                                │
 Prescription ──────────────────────────┘
       │
       ▼
     Patient Health Timeline
       ▲
       │
  Laboratory Results
       │
  Blood Donation Support
```

The patient does not need to understand the internal architecture. The UI should present healthcare tasks as simple actions and status updates.

---

# 3. Field Classification

Every patient field should have one of the following classifications:

| Classification | Meaning |
|---|---|
| **Required** | Needed to complete the current flow. |
| **Conditional** | Needed only when a particular service requires it. |
| **Recommended** | Highly useful, but can be completed later. |
| **Optional** | Personalization or additional context. |
| **System** | Generated/maintained by the backend; the patient should not edit it directly. |

## Golden Rule

> Do not collect medical information simply because it could be useful in the future. Collect the minimum information required for a clear purpose, then allow the patient to add more when needed.

---

# 4. Entry Point — Landing Page

When the patient opens TechCare for the first time:

```text
                    TECHCARE
             CARE FOR BETTER HEALTH

        [ Create Account ]   [ Login ]

 Services      About      Help      Privacy      Terms
```

### No medical data is collected here.

---

# 5. Registration Workflow

```text
Create Account
      ↓
Enter Basic Information
      ↓
Validate Form
      ↓
Verify Phone / Email
      ↓
Account Created
      ↓
Complete Profile
```

## 5.1 Registration Fields

| UI Label | Suggested Field Name | Type | Level | Validation / Rule |
|---|---|---|---|---|
| Full Name | `full_name` | String | Required | Trim; reasonable length; no meaningless repeated characters. |
| Phone Number | `phone_number` | String | Required | Normalize country code and verify ownership. |
| Email | `email` | Email | Required | Valid format. |
| Password | `password` | Password | Required | Strong password rules. |
| Confirm Password | `confirm_password` | Password | Required | Must match password. |
| Date of Birth | `date_of_birth` | Date | Required | Cannot be a future date. |
| Sex / Gender | `sex_or_gender` | Enum | Conditional / Recommended | Include only values actually needed by the product. |

### Do not make these mandatory at initial sign-up unless the operating model specifically requires them:

- National ID number
- Full medical history
- All previous diseases
- Prescription data
- Blood group
- Surgery history

---

# 6. Account Verification

Recommended:

```text
Registration Submitted
        ↓
OTP / Verification Code
        ↓
Patient Enters Code
        ↓
Verified
        ↓
Active Account
```

## Verification Fields

| Field | Suggested Name | Type | Level |
|---|---|---|---|
| OTP Code | `otp_code` | String | Required during verification |
| Phone Verified At | `phone_verified_at` | DateTime | System |
| Email Verified At | `email_verified_at` | DateTime | System / Conditional |
| Account Status | `account_status` | Enum | System |
| Created At | `created_at` | DateTime | System |
| Last Login At | `last_login_at` | DateTime | System |

Suggested account states:

```text
PENDING_VERIFICATION
ACTIVE
SUSPENDED
DEACTIVATED
```

---

# 7. Patient Basic Profile

After creating the account, the patient completes the profile.

## Personal Information

| UI Label | Suggested Name | Type | Level | Purpose |
|---|---|---|---|---|
| Profile Photo | `profile_photo` | File | Optional | Personalization. |
| Full Name | `full_name` | String | Required | Identity/display. |
| Date of Birth | `date_of_birth` | Date | Required | Age-related context. |
| Sex / Gender | `sex_or_gender` | Enum | Conditional | Only when clinically/service relevant. |
| Phone | `phone_number` | String | Required | Coordination. |
| Email | `email` | Email | Required | Account and communication. |
| Preferred Language | `preferred_language` | Enum | Recommended | Arabic / English initially, if supported. |
| Preferred Contact Method | `preferred_contact_method` | Enum | Optional | App / Phone / Email. |

---

# 8. Accessibility Profile

Because TechCare is designed partly around people who have difficulty moving, accessibility should be a first-class profile capability.

| UI Label | Suggested Name | Level | Notes |
|---|---|---|---|
| Accessibility Needs | `accessibility_needs` | Optional | Structured options + free-text note when needed. |
| Mobility Assistance | `mobility_assistance_needed` | Optional | Helps home-service planning. |
| Communication Support | `communication_support` | Optional | Useful for visit coordination. |
| Preferred Provider Gender | `preferred_provider_gender` | Optional | Only if product policy supports it. |
| Caregiver Involvement | `has_caregiver` | Optional | Used when another person coordinates care. |

Prefer storing **what assistance is needed** over collecting unnecessary disability details.

---

# 9. Address Book

A patient should have multiple saved addresses rather than one address field.

```text
Address Book
├── Home
├── Work
└── Other
```

## Address Fields

| UI Label | Suggested Name | Type | Level |
|---|---|---|---|
| Address Label | `label` | String | Required |
| Governorate | `governorate` | Enum/String | Required for structured address |
| City | `city` | String | Required |
| District / Area | `district` | String | Required |
| Street | `street` | String | Required |
| Building Number | `building_number` | String | Recommended |
| Floor | `floor` | String | Optional |
| Apartment / Unit | `unit_number` | String | Optional |
| Landmark | `landmark` | String | Optional |
| Additional Instructions | `additional_instructions` | Text | Optional |
| Latitude | `latitude` | Decimal | Conditional |
| Longitude | `longitude` | Decimal | Conditional |
| Address Type | `address_type` | Enum | Required |
| Default Address | `is_default` | Boolean | System / Required behavior |

---

# 10. Location Workflow

The patient may grant location access:

```text
Use Current Location?

[ Allow While Using ]
[ Not Now ]
```

## Location Data

| Field | Suggested Name | Level |
|---|---|---|
| Latitude | `latitude` | Conditional |
| Longitude | `longitude` | Conditional |
| Accuracy | `accuracy_meters` | System / Optional |
| Source | `source` | System | GPS / Manual / Saved Address |
| Captured At | `captured_at` | System |

### Location principles

- Use live location only when a feature needs it.
- Do not expose a patient's exact coordinates on public provider-search screens.
- A saved address is different from a live location.
- An active visit/order may require exact destination information only after the request reaches an appropriate state.
- Location sharing should be purpose-limited and time-limited where practical.

---

# 11. Medical Profile

The patient can build the medical profile progressively.

```text
Medical Profile
├── Conditions
├── Previous Diseases
├── Current Medications
├── Allergies
├── Previous Surgeries
├── Blood Group
└── Medical Notes
```

The product must clearly distinguish **patient-reported information** from **clinician-created or laboratory-published information**.

---

# 12. Medical Conditions

Do not store only:

> "I have sugar and pressure."

Use structured entries.

| Field | Suggested Name | Level |
|---|---|---|
| Condition | `condition_name` | Required when creating an entry |
| Status | `condition_status` | Recommended |
| Approximate Onset | `onset_date` | Optional |
| Source | `source_type` | System |
| Self Reported | `is_self_reported` | System / true for patient-created entry |
| Notes | `notes` | Optional |

Example:

```text
Condition: Diabetes
Status: Active
Source: Patient Reported
```

---

# 13. Previous Diseases

| Field | Suggested Name | Level |
|---|---|---|
| Disease / Condition | `condition_name` | Required for entry |
| Diagnosis Date / Year | `diagnosed_at` | Optional |
| Status | `status` | Recommended |
| Treating Provider | `provider_id` | Optional |
| Notes | `notes` | Optional |

Always allow:

```text
Unknown
I do not remember
```

The patient should never be forced to invent a date.

---

# 14. Current Medications

This section is structured because authorized doctors may need it during consultations.

| Field | Suggested Name | Level | Notes |
|---|---|---|---|
| Medicine | `medicine_id` / `medicine_name` | Required | Prefer a standardized medicine catalog. |
| Strength | `strength` | Recommended | e.g. 500 mg. |
| Dosage Form | `dosage_form` | Recommended | Tablet, capsule, syrup, etc. |
| Dose | `dose` | Recommended | Amount per administration. |
| Frequency | `frequency` | Recommended | e.g. once daily. |
| Route | `route` | Conditional | Oral, topical, etc. |
| Start Date | `start_date` | Optional | Allow unknown. |
| End Date | `end_date` | Optional | Empty for ongoing use. |
| Prescriber | `prescriber_id` | Optional | Doctor account if available. |
| Source | `source_type` | System | Patient Reported / Prescription / Imported. |
| Notes | `notes` | Optional | Context. |

### Trust rule

```text
Patient-entered medication
       ≠
Verified doctor prescription
```

---

# 15. Allergies

Allergies should never be buried inside general notes.

| Field | Suggested Name | Level |
|---|---|---|
| Allergen | `allergen` | Required for entry |
| Type | `allergy_type` | Recommended | Drug / Food / Environmental / Other |
| Reaction | `reaction` | Recommended |
| Severity | `severity` | Optional |
| Source | `source_type` | System |
| Notes | `notes` | Optional |

---

# 16. Previous Surgeries

| Field | Suggested Name | Level |
|---|---|---|
| Procedure | `procedure_name` | Required for entry |
| Date / Year | `procedure_date` | Optional |
| Facility | `facility_name` | Optional |
| Provider | `provider_id` | Optional |
| Notes | `notes` | Optional |

---

# 17. Blood Group in Patient Profile

| Field | Suggested Name | Level |
|---|---|---|
| Blood Group | `blood_group` | Recommended / Conditional |
| Verification Status | `verification_status` | System |
| Verified At | `verified_at` | System |
| Verification Source | `verification_source` | System |

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
Unknown
```

### Critical safety rule

A patient-entered blood group must not automatically be treated as medically verified for transfusion decisions.

---

# 18. Medical Notes

| Field | Suggested Name | Level |
|---|---|---|
| Note | `note` | Optional |
| Created At | `created_at` | System |
| Updated At | `updated_at` | System |
| Source | `source_type` | System |

Structured fields should be preferred over a giant unrestricted text box.

---

# 19. Emergency / Caregiver Information

The prototype should collect a basic emergency contact even before future emergency integrations are built.

| Field | Suggested Name | Level |
|---|---|---|
| Emergency Contact Name | `emergency_contact_name` | Recommended |
| Relationship | `emergency_contact_relationship` | Recommended |
| Phone | `emergency_contact_phone` | Recommended |
| Alternate Phone | `emergency_contact_alt_phone` | Optional |
| Can Coordinate Care? | `can_coordinate_care` | Optional |

Future caregiver accounts should use explicit permissions rather than shared passwords.

---

# 20. Patient Dashboard

## Dashboard Sections

```text
Dashboard
│
├── Welcome / Profile Summary
├── Quick Actions
│   ├── Find Doctor
│   ├── Find Nurse
│   ├── Find Medicine
│   ├── Book Lab Test
│   └── Blood Request
│
├── Upcoming Appointment
├── Active Requests
├── Active Pharmacy Orders
├── Latest Prescription
├── Latest Laboratory Result
├── Notifications
└── Points / Recognition
```

## Dashboard priority

1. What the patient can do now.
2. What is happening now.
3. What needs attention.
4. What happened recently.

---

# 21. Patient Navigation

```text
Dashboard

Profile
Medical Profile
Medical History
Prescriptions

Doctors
Nurses

Pharmacy
  ├── Medicines
  └── Orders

Laboratory
  ├── Tests
  ├── Bookings
  └── Results

Blood Donation
  └── Requests

Appointments

Payments
Transactions

Points & Rewards

Notifications
Reviews
Complaints

Settings
Privacy & Security
```

---

# 22. Doctor Discovery

## Search Flow

```text
Doctors
   ↓
Choose Specialty
   ↓
Use Location
   ↓
Apply Filters
   ↓
View Verified Doctors
```

## Search Fields

| Field | Suggested Name | Level |
|---|---|---|
| Specialty | `specialty_id` | Required |
| Search Text | `query` | Optional |
| Location Context | `location_context` | Required for nearby search |
| Radius | `radius_km` | Optional |
| Minimum Rating | `min_rating` | Optional |
| Maximum Price | `max_price` | Optional |
| Availability | `availability_filter` | Optional |
| Home Visit Only | `home_visit_only` | Optional |
| Sort | `sort` | Optional |

### Result card

```text
Dr. Ahmed Mohamed
Orthopedic Specialist
✓ Verified
⭐ 4.8 (125 reviews)
📍 3.2 KM
500 EGP
🏠 Home Visit
Available Today
```

---

# 23. Doctor Profile

Patient sees:

## Identity

- Profile photo
- Doctor name
- Verified status
- Specialty

## Professional information

- Qualifications
- Experience summary
- Professional description
- Services

## Service information

- Service name
- Base price
- Home-visit availability
- Distance pricing summary
- Available slots

## Trust information

- Rating
- Review count
- Reviews

Action:

```text
[ Choose Service ]
```

---

# 24. Doctor Appointment Selection

| Field | Suggested Name | Required |
|---|---|---:|
| Doctor | `provider_id` | Yes |
| Service | `service_id` | Yes |
| Date | `appointment_date` | Yes |
| Time Slot | `slot_id` | Yes |
| Service Address | `address_id` | Yes for home visit |
| Patient Note | `patient_note` | Optional |

The server must verify that the selected slot is still available at booking time.

---

# 25. Doctor Price Breakdown

Example:

```text
Base Service Price       500 EGP
Distance Fee              75 EGP
────────────────────────────────
Total                     575 EGP
```

## Pricing Inputs

```text
base_price
included_distance_km
distance_km
billable_distance_km
distance_rate
extra_fees
discount
final_total
pricing_policy_version
```

The client should display the calculation, but the server remains the source of truth.

---

# 26. Doctor Booking Request

When the patient confirms:

```text
Booking
status     = PENDING
created_at = server time
expires_at = server-calculated deadline
```

The patient sees:

```text
Waiting for doctor response...
```

The **5-minute value** is a business-rule response window, not the duration of the medical appointment.

## Booking States

```text
PENDING
   │
   ├── ACCEPTED → CONFIRMED → IN_PROGRESS → COMPLETED
   │
   ├── REJECTED
   │
   ├── EXPIRED
   │
   └── CANCELLED
```

The frontend countdown is visual only. The backend enforces expiration.

---

# 27. Doctor Appointment — Before the Visit

Patient sees:

```text
Appointment Confirmed ✓

Doctor
Service
Date
Time
Address
Distance
Base Price
Distance Fee
Total
```

Possible actions:

```text
View Details
Cancel (if policy allows)
Contact / Chat (if enabled)
```

---

# 28. Consultation Experience

```text
Appointment
      ↓
Consultation Started
      ↓
Doctor Review
      ↓
Consultation
      ↓
Diagnosis + Treatment
      ↓
Prescription
      ↓
Consultation Completed
```

### Patient-side view

The patient may see:

- Consultation status
- Doctor details
- Chat/messages if enabled
- Final consultation record
- Diagnosis
- Treatment plan
- Prescription
- Follow-up instructions

Clinical records should not exist only as chat messages.

---

# 29. Patient View of Doctor-Created Medical Record

After completion:

```text
Consultation
├── Date
├── Doctor
├── Diagnosis
├── Treatment Plan
├── Prescription
└── Follow-up
```

The patient can save and review the record later.

---

# 30. Prescription Workflow

```text
Doctor Finalizes Prescription
          ↓
Prescription Saved
          ↓
Patient Notification
          ↓
Patient Opens Prescription
          ↓
[ Find My Medicines ]
```

## Prescription fields

| Field | Suggested Name |
|---|---|
| Prescription ID | `prescription_id` |
| Doctor | `prescriber_id` |
| Consultation | `consultation_id` |
| Medicine | `medicine_id` |
| Strength | `strength` |
| Dosage Form | `dosage_form` |
| Dose | `dose` |
| Frequency | `frequency` |
| Duration | `duration` |
| Instructions | `instructions` |
| Created At | `created_at` |
| Status | `status` |

---

# 31. Pharmacy — Medicine Search

Patient can search directly:

```text
Pharmacy
   ↓
Search Medicine
```

Or indirectly:

```text
Prescription
   ↓
Find My Medicines
```

## Search Fields

| Field | Suggested Name | Level |
|---|---|---|
| Query | `query` | Required |
| Medicine | `medicine_id` | Conditional |
| Location | `location_context` | Required for nearby search |
| Radius | `radius_km` | Optional |
| Delivery Only | `delivery_only` | Optional |
| Pickup Only | `pickup_only` | Optional |
| Maximum Price | `max_price` | Optional |
| Sort | `sort` | Optional |

## Result

```text
Medicine
Pharmacy
✓ Available
Price
Distance
Rating
Delivery / Pickup
```

---

# 32. Pharmacy Comparison

Example:

```text
Panadol Extra

Pharmacy A
1.2 KM
75 EGP
Available
Delivery
⭐ 4.7

Pharmacy B
2.1 KM
72 EGP
Available
Pickup + Delivery
⭐ 4.5
```

The patient can sort by:

- Nearest
- Lowest price
- Highest rating
- Best overall

---

# 33. Pharmacy Order

## Cart / Order fields

| Field | Suggested Name | Level |
|---|---|---|
| Pharmacy | `pharmacy_id` | Required |
| Medicine / Product | `medicine_id` | Required |
| Quantity | `quantity` | Required |
| Unit Price | `unit_price_snapshot` | System |
| Prescription | `prescription_id` | Conditional |
| Fulfillment Method | `fulfillment_method` | Required |
| Delivery Address | `address_id` | Required for delivery |
| Pickup Slot | `pickup_slot_id` | Conditional |
| Delivery Notes | `delivery_notes` | Optional |
| Patient Note | `patient_note` | Optional |

## Price

```text
Medicine Subtotal
+ Delivery Fee
+ Approved Additional Charges
- Discount
= Final Total
```

Historical order prices must remain unchanged when pharmacy prices change later.

---

# 34. Pharmacy Order Lifecycle

```text
PENDING
   ↓
ACCEPTED
   ↓
RESERVED / PREPARING
   ↓
READY FOR PICKUP / OUT FOR DELIVERY
   ↓
COMPLETED
```

Possible alternate states:

```text
REJECTED
CANCELLED
REFUND_PENDING
REFUNDED
```

---

# 35. Nurse Discovery

Patient selects:

```text
Nurses
   ↓
Choose Nursing Service
   ↓
Choose Location
   ↓
Choose Date / Time
   ↓
Compare Nurses
```

## Search fields

| Field | Suggested Name | Level |
|---|---|---|
| Nursing Service | `service_id` | Required |
| Location | `location_context` | Required |
| Date | `service_date` | Required |
| Time Slot | `slot_id` | Required |
| Radius | `radius_km` | Optional |
| Minimum Rating | `min_rating` | Optional |
| Maximum Price | `max_price` | Optional |

---

# 36. Nurse Service Request

```text
Select Nurse
    ↓
Select Service
    ↓
Select Date / Time
    ↓
Select Address
    ↓
Review Price + Distance
    ↓
Confirm Request
```

## Request fields

```text
provider_id
service_id
service_date
slot_id
address_id
base_price
distance_fee
final_total
patient_note
```

## Status

```text
PENDING
   ↓
ACCEPTED / REJECTED / EXPIRED
   ↓
CONFIRMED
   ↓
IN_PROGRESS
   ↓
COMPLETED
```

There is no diagnosis/prescription workflow attached to the nurse role in this prototype.

---

# 37. Laboratory Discovery

Patient enters:

```text
Laboratory
   ↓
Find Test
```

Possible entry points:

```text
Search test manually
OR
Open a doctor-linked lab request
```

---

# 38. Laboratory Search Fields

| Field | Suggested Name | Level |
|---|---|---|
| Laboratory Test | `lab_test_id` | Required |
| Location | `location_context` | Required |
| Home Collection | `home_collection` | Required for home visit |
| Date | `collection_date` | Required |
| Time Slot | `slot_id` | Required |
| Laboratory | `laboratory_id` | Required after selection |
| Referral / Doctor Request | `referral_id` | Conditional |
| Patient Note | `patient_note` | Optional |

---

# 39. Laboratory Booking

```text
Choose Test
    ↓
Compare Labs
    ↓
Choose Home Collection
    ↓
Choose Address
    ↓
Choose Date / Time
    ↓
Review Price
    ↓
Confirm Booking
```

---

# 40. Laboratory Order Lifecycle

```text
BOOKED
   ↓
CONFIRMED
   ↓
COLLECTOR ASSIGNED
   ↓
ON THE WAY
   ↓
SAMPLE COLLECTED
   ↓
RECEIVED BY LAB
   ↓
PROCESSING
   ↓
RESULT READY
```

---

# 41. Laboratory Result

The patient gets a notification when the result is available.

## Result fields

| Field | Suggested Name |
|---|---|
| Result ID | `result_id` |
| Test | `lab_test_id` |
| Laboratory | `laboratory_id` |
| Collection Date | `collected_at` |
| Resulted Date | `resulted_at` |
| Structured Result | `result_values` |
| Reference Range | `reference_range` |
| Report File | `report_file_id` |
| Result Status | `result_status` |
| Published By | `published_by` |
| Published At | `published_at` |

The system must distinguish:

```text
Patient-uploaded document
vs.
Laboratory-published result
```

---

# 42. Blood Donation — Patient Request

Patient selects:

```text
Blood Donation
   ↓
Create Blood Request
```

The workflow is designed as **voluntary donor coordination**, not buying and selling blood.

## Request fields

| Field | Suggested Name | Level |
|---|---|---|
| Requester | `requester_id` | System |
| Patient Display Name | `patient_display_name` | Conditional |
| Required Blood Group | `required_blood_group` | Required |
| Requested Units | `requested_units` | Conditional |
| Hospital / Facility | `facility_id` | Required where verified facilities are modeled |
| Facility Name | `facility_name` | Conditional |
| Facility Location | `facility_location` | Required |
| Urgency | `urgency_level` | Required |
| Needed By | `needed_by` | Conditional |
| Request Notes | `request_notes` | Optional |
| Contact Method | `contact_method` | Required |

---

# 43. Blood Matching

The platform may use:

```text
Required Blood Group
       +
Location
       +
Donor Availability
       +
Eligibility Status
```

But the platform should **not** be the final authority on donor medical eligibility or transfusion safety.

The final medical screening, testing, collection, and transfusion decisions belong to the authorized healthcare/blood service.

## Patient view

```text
Request ID
Blood Group
Hospital
Location
Urgency
Current Status
Number of Responses (privacy-safe)
Last Update
```

Do not reveal donor private information or exact personal location publicly.

---

# 44. Blood Request States

```text
DRAFT
  ↓
OPEN
  ↓
MATCHED
  ↓
COORDINATING
  ↓
FULFILLED
  ↓
CLOSED
```

Alternative ending states:

```text
CANCELLED
EXPIRED
```

---

# 45. Blood Rewards / Points

If the platform uses points for donor recognition:

```text
Verified Completed Donation
          ↓
Recognition Event
          ↓
Points Added
```

Points should be modeled independently from any blood-unit price.

Example:

```text
Current Points: 1,250
Donation Recognition: +100
New Balance: 1,350
```

The exact reward policy must be approved before real-world operation.

---

# 46. Patient Appointments Page

```text
Appointments
├── Upcoming
├── Pending Requests
├── Completed
├── Cancelled
└── Expired
```

Each item shows:

```text
Provider
Service
Date
Time
Location
Price
Status
```

---

# 47. Patient Prescription Page

```text
Prescriptions
├── Active
├── Historical
└── Expired
```

Each prescription should show:

```text
Doctor
Date
Diagnosis / consultation reference
Medicines
Instructions
Status
```

---

# 48. Patient Medical History

Recommended structure:

```text
Medical History
│
├── Conditions
├── Medications
├── Allergies
├── Surgeries
├── Consultations
│   ├── Diagnosis
│   ├── Treatment
│   └── Prescription
│
└── Laboratory Results
```

---

# 49. Patient Health Timeline

This is one of the strongest product features.

Example:

```text
05 Oct
Doctor Consultation
        ↓
Diagnosis
        ↓
Prescription

06 Oct
Pharmacy Order
        ↓
Completed

10 Oct
Laboratory Test
        ↓
Result Ready

15 Oct
Nurse Home Visit
        ↓
Completed
```

Every event should have:

```text
event_id
event_type
source_module
patient_id
event_date
status
summary
related_record_id
created_at
```

---

# 50. Patient Notifications

## Account

- Account verification.
- Security changes.
- New login/session alerts where enabled.

## Doctor

- Booking request submitted.
- Request accepted.
- Request rejected.
- Request expired.
- Appointment reminder.
- Consultation completed.
- Prescription ready.

## Nurse

- Request submitted.
- Request accepted/rejected.
- Request expired.
- Visit reminder.
- Visit status update.
- Service completed.

## Pharmacy

- Order submitted.
- Order accepted/rejected.
- Preparing.
- Ready for pickup.
- Out for delivery.
- Completed.
- Payment/refund update.

## Laboratory

- Booking confirmed.
- Collector assigned.
- Collector on the way.
- Sample collected.
- Result ready.

## Blood Donation

- Request created.
- Matching update.
- Donor response update.
- Request fulfilled/closed.

### Privacy rule

A notification preview should not unnecessarily expose diagnoses, medications, or other sensitive health information on a locked device.

---

# 51. Patient Payments

## Transaction fields

| Field | Suggested Name |
|---|---|
| Transaction ID | `transaction_id` |
| Service Type | `service_type` |
| Reference ID | `reference_id` |
| Subtotal | `subtotal` |
| Distance Fee | `distance_fee` |
| Delivery Fee | `delivery_fee` |
| Discount | `discount_amount` |
| Final Amount | `final_amount` |
| Payment Method | `payment_method` |
| Payment Status | `payment_status` |
| Created At | `created_at` |
| Completed At | `completed_at` |
| Refund Amount | `refund_amount` |

Patient view:

```text
Service
Date
Subtotal
Additional Fees
Discount
Total
Payment Status
```

The backend should calculate the final amount. The frontend should not be the authority for money.

---

# 52. Ratings

Only an eligible completed service should become rateable.

## Fields

| Field | Suggested Name | Level |
|---|---|---|
| Rating | `rating` | Required |
| Review Text | `review_text` | Optional |
| Booking / Order | `reference_id` | System |
| Created At | `created_at` | System |

Possible categories:

```text
Overall
Service Quality
Communication
Punctuality
```

---

# 53. Complaints

```text
Complaints
   ↓
New Complaint
   ↓
Choose Related Service / Order
   ↓
Choose Category
   ↓
Write Description
   ↓
Attach Evidence if needed
   ↓
Submit
```

## Fields

| Field | Suggested Name | Level |
|---|---|---|
| Related Booking / Order | `reference_id` | Recommended |
| Category | `category` | Required |
| Description | `description` | Required |
| Attachments | `attachments` | Optional |
| Preferred Contact Method | `contact_method` | Optional |

## Status

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

---

# 54. Patient Support / Help

The patient should always have access to:

```text
Help Center
Contact Support
My Complaints
Report an Issue
```

When possible, support requests should reference the booking/order that caused the issue.

---

# 55. Patient Privacy Controls

Recommended:

```text
Privacy
├── Location Permission
├── Data Sharing Preferences
├── Notification Privacy
├── Address Management
└── Available Data Requests
```

The UI must not show privacy switches that have no backend effect.

---

# 56. Patient Security Controls

```text
Security
├── Change Password
├── Forgot Password
├── Active Sessions
├── Login Alerts
└── Log Out All Devices
```

High-risk actions may later require stronger authentication.

---

# 57. Health Data Access Rules

The patient is the owner of the care experience, but not every provider should automatically access the entire medical history.

## Access concept

```text
Provider
   ↓
Role
   ↓
Valid care relationship
   ↓
Purpose of access
   ↓
Relevant data scope
   ↓
Authorization
   ↓
Audit Log
```

### Example

**Doctor:** relevant medical history, allergies, current medicines, relevant consultations and prescriptions.

**Nurse:** only information needed for the authorized nursing service.

**Pharmacy:** information needed to fulfill the order/prescription, not the full patient chart.

**Laboratory:** information needed to identify and fulfill the laboratory service, not unrelated clinical history.

---

# 58. Patient Error Handling

## No doctor found

```text
No verified doctors were found nearby.

[ Expand Search Radius ]
[ Change Specialty ]
[ Try Another Time ]
```

## Doctor request expired

```text
The doctor did not respond in time.

[ Find Another Doctor ]
```

## Slot unavailable

```text
This time slot is no longer available.

[ Choose Another Slot ]
```

## Medicine unavailable

```text
This medicine is no longer available at this pharmacy.

[ See Other Pharmacies ]
```

## Lab slot unavailable

```text
The selected collection slot is no longer available.

[ Choose Another Slot ]
```

## Payment failure

```text
Payment could not be completed.

[ Try Again ]
[ Choose Another Method ]
```

## Network failure

The client must not create duplicate bookings or orders when the user retries after an uncertain network response.

---

# 59. Patient Data Model — High Level

```text
User
  │
  └── PatientProfile
       │
       ├── PatientAddress
       ├── Location
       ├── AccessibilityProfile
       ├── EmergencyContact
       │
       ├── MedicalCondition
       ├── Medication
       ├── Allergy
       ├── SurgeryHistory
       ├── MedicalNote
       ├── BloodGroup
       │
       ├── Appointment
       │      └── Consultation
       │           ├── Diagnosis
       │           └── Prescription
       │                └── PrescriptionItem
       │
       ├── PharmacyOrder
       │      └── PharmacyOrderItem
       │
       ├── LaboratoryOrder
       │      └── LaboratoryResult
       │
       ├── BloodRequest
       │      └── DonorMatch
       │
       ├── Payment / Transaction
       ├── Rating
       ├── Complaint
       ├── Notification
       └── HealthTimelineEvent
```

---

# 60. Patient Dashboard API Concept

The dashboard should return a summary, not the complete medical record.

Example:

```json
{
  "patient": {
    "display_name": "...",
    "profile_completion": 82
  },
  "upcoming_appointment": {},
  "active_requests": [],
  "active_orders": [],
  "latest_prescription": {},
  "latest_lab_result": {},
  "unread_notifications": 3,
  "points_balance": 0
}
```

The API should not return unrelated sensitive information just because the dashboard endpoint can technically query it.

---

# 61. Profile Completion Logic

The patient should be able to finish essential information without being blocked by non-essential profile sections.

Example:

```text
Basic Account        ✅
Phone Verification   ✅
Address              ✅
Location             ✅
Medical Profile      70%
Emergency Contact    ✅
```

When a particular service requires missing data, the UI should request it at that point.

Example:

```text
Book Home Nurse
      ↓
No service address found
      ↓
[ Add Address ]
```

---

# 62. Complete Patient Navigation Map

```text
                         PATIENT
                            │
                    ┌───────┴────────┐
                    ▼                ▼
                Dashboard         Profile
                    │                │
        ┌───────────┼───────────┐    ├── Personal
        │           │           │    ├── Addresses
        ▼           ▼           ▼    └── Accessibility
      Doctor      Nurse      Pharmacy
        │           │           │
        ▼           ▼           ▼
      Booking     Booking     Medicine Order
        │                        │
        ▼                        ▼
   Consultation              Delivery/Pickup
        │
        ▼
  Diagnosis + Rx
        │
        ▼
  Medical History
        ▲
        │
        └──────────── Laboratory Results

              Blood Donation
                    │
                    ▼
               Blood Request
                    │
                    ▼
               Donor Matching

Shared:
Notifications • Payments • Ratings • Complaints • Settings • Privacy
```

---

# 63. Complete Patient Scenario — Example

The following is the full story of a realistic patient journey.

## Step 1 — Registration

The patient opens TechCare, creates an account, verifies the phone number, and completes their personal profile.

## Step 2 — Profile

The patient adds their home address and allows location access for nearby-service discovery.

## Step 3 — Medical Profile

The patient records known conditions, current medicines, allergies, previous surgeries, and blood group if known.

The system marks patient-entered information as self-reported until a trusted clinical source exists.

## Step 4 — Doctor Search

The patient needs an orthopedic consultation.

```text
Doctors
→ Orthopedics
→ Nearby
→ Verified
→ Compare rating / price / availability
```

## Step 5 — Choose Doctor

The patient opens a verified doctor profile and chooses a home-visit slot.

## Step 6 — Price

The system calculates:

```text
Base Service = 500 EGP
Distance Fee = 75 EGP
Total = 575 EGP
```

## Step 7 — Request

The patient sends the request.

```text
PENDING
```

The doctor has the configured response window. If accepted:

```text
ACCEPTED
→ CONFIRMED
```

If unanswered:

```text
EXPIRED
```

## Step 8 — Consultation

The doctor visits the patient and records a structured consultation.

## Step 9 — Prescription

The doctor creates a prescription.

The patient receives a notification and can open the prescription in their account.

## Step 10 — Medicine Search

The patient chooses:

```text
Find My Medicines
```

The platform searches nearby verified pharmacies and compares availability, distance, rating, and price.

## Step 11 — Pharmacy Order

The patient orders the medicine for delivery.

```text
PENDING
→ ACCEPTED
→ PREPARING
→ OUT FOR DELIVERY
→ COMPLETED
```

## Step 12 — Laboratory

The doctor recommends a laboratory test.

The patient books a home sample collection.

```text
BOOKED
→ CONFIRMED
→ COLLECTOR ASSIGNED
→ SAMPLE COLLECTED
→ PROCESSING
→ RESULT READY
```

The result is added to the patient's health history.

## Step 13 — Nursing

The patient later needs home nursing and books a nurse using the same location/availability/request model.

## Step 14 — Blood Request

A separate blood need arises. The patient creates a blood-donation request and the platform coordinates matching with relevant nearby donors.

The platform does not sell the blood or determine the final medical suitability of the donor.

## Step 15 — Continuity

The patient opens the Health Timeline:

```text
Doctor Consultation
      ↓
Diagnosis
      ↓
Prescription
      ↓
Pharmacy Order
      ↓
Laboratory Test
      ↓
Laboratory Result
      ↓
Nurse Visit
      ↓
Blood Request
```

The patient can now understand their healthcare journey from one place.

---

# 64. What the Patient Must Never Be Asked to Do

The product should not force the patient to:

- Enter the same address repeatedly.
- Re-enter the same medicine list for every doctor.
- Explain the same consultation history to every screen.
- Calculate the final price manually.
- Understand provider state-machine terminology.
- Upload the same prescription repeatedly when a verified in-platform prescription already exists.
- Share exact live location publicly.
- Decide whether a donor is medically eligible.

The system should absorb complexity and show the patient only what is necessary.

---

# 65. What the System Must Never Assume

```text
Patient diagnosis entered by patient
        ≠
Confirmed diagnosis

Patient medication entry
        ≠
Verified prescription

Patient blood group entry
        ≠
Transfusion-ready verification

GPS location
        ≠
Permanent address

Provider account
        ≠
Verified provider

Displayed pharmacy stock
        ≠
Guaranteed real-time stock

Uploaded lab file
        ≠
Validated lab result
```

---

# 66. Patient Definition of Done

The patient module is complete only when all of the following work:

- [ ] Registration and validation.
- [ ] Phone/email verification as configured.
- [ ] Profile editing.
- [ ] Address book.
- [ ] Location permission handling.
- [ ] Accessibility information.
- [ ] Medical conditions.
- [ ] Current medications.
- [ ] Allergies.
- [ ] Previous surgeries.
- [ ] Blood group with verification distinction.
- [ ] Emergency contact.
- [ ] Doctor discovery.
- [ ] Doctor profile.
- [ ] Doctor booking.
- [ ] Server-side booking expiry.
- [ ] Double-booking prevention.
- [ ] Consultation result visibility.
- [ ] Prescription access.
- [ ] Prescription-to-pharmacy flow.
- [ ] Medicine search.
- [ ] Pharmacy order lifecycle.
- [ ] Nurse discovery and booking.
- [ ] Laboratory discovery and booking.
- [ ] Home sample collection status.
- [ ] Laboratory result delivery.
- [ ] Blood request creation.
- [ ] Donor matching and status tracking.
- [ ] Notifications.
- [ ] Payments and transaction history.
- [ ] Ratings.
- [ ] Complaints.
- [ ] Health timeline.
- [ ] Privacy controls.
- [ ] Security controls.
- [ ] Server-side authorization for protected records.
- [ ] Critical error/retry handling.
- [ ] Automated tests for critical patient journeys.

---

# 67. Final Patient Workflow

```text
OPEN TECHCARE
      ↓
Register / Login
      ↓
Verify Account
      ↓
Personal Profile
      ↓
Address Book
      ↓
Location Setup
      ↓
Medical Profile
      ↓
Emergency / Caregiver Information
      ↓
PATIENT DASHBOARD
      │
      ├───────────────────────┐
      │                       │
      ▼                       ▼
Find Doctor               Find Nurse
      │                       │
      ▼                       ▼
Compare Providers       Compare Nurses
      │                       │
      ▼                       ▼
Choose Slot              Choose Slot
      │                       │
      ▼                       ▼
Review Price + Distance Review Price + Distance
      │                       │
      ▼                       ▼
Send Request             Send Request
      │                       │
      ▼                       ▼
Accept / Reject / Expire Accept / Reject / Expire
      │                       │
      ▼                       ▼
Consultation             Home Nursing
      │
      ▼
Diagnosis + Prescription
      │
      ▼
Medical Record
      │
      ▼
Find Medicines
      │
      ▼
Nearby Pharmacies
      │
      ▼
Compare / Order
      │
      ▼
Delivery / Pickup

────────────────────────────────────────────

Laboratory
   ↓
Choose Test
   ↓
Book Home Collection
   ↓
Sample Collected
   ↓
Processing
   ↓
Result Ready
   ↓
Medical Timeline

────────────────────────────────────────────

Blood Donation
   ↓
Create Request
   ↓
Blood Group + Facility + Location + Urgency
   ↓
Compatible Nearby Donor Matching
   ↓
Donor Response
   ↓
Authorized Medical Coordination
   ↓
Request Closed

────────────────────────────────────────────

Every Completed Service
   ↓
Payment / Transaction
   ↓
Rating / Complaint
   ↓
Notification / History
   ↓
PATIENT HEALTH TIMELINE
```

---

# 68. Product Summary

The patient experience in TechCare can be summarized in one sentence:

> **The patient tells TechCare what they need, discovers the appropriate verified healthcare service, requests or books it, receives the service, and keeps the important outcome inside one organized healthcare journey.**

