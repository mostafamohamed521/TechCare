# TechCare Frontend

TechCare is a healthcare web platform designed to connect patients with healthcare providers and healthcare services through a simple, accessible, trustworthy, and professional user experience.

The frontend is responsible for the complete user-facing experience: public pages, authentication, role registration, dashboards, shared UI components, responsive behavior, validation, API communication, notifications, loading and error states, accessibility, and frontend documentation.

---

## 1. Frontend Overview

The TechCare frontend is built as a modular multi-role web application.

### Frontend Stack

- HTML5
- CSS3
- Vanilla JavaScript (ES6+)
- REST API integration with ASP.NET Core backend
- JSON for API data exchange
- Browser APIs where required, such as Geolocation and File APIs

### Main Frontend Goals

The frontend must be:

- Professional
- Simple
- Responsive
- Accessible
- Consistent
- Secure by design
- Easy to maintain
- Easy for the five team members to develop in parallel
- Ready to integrate with the ASP.NET Core backend

---

# 2. TechCare Roles

The frontend supports the following main roles:

| Role | Description |
|---|---|
| Patient | Searches for healthcare services, books providers, manages profile and medical information |
| Doctor | Provides medical services and manages professional requests and consultations |
| Nurse | Provides nursing and home-care services |
| Pharmacy / Pharmacist | Manages pharmacy information, branches, services, and medicine availability |
| Laboratory | Provides laboratory services and manages lab information |
| Blood Donor | Registers as a donor and participates in blood donation matching workflows |
| Admin | Manages platform users, providers, verification, complaints, moderation, and platform operations |

---

# 3. Frontend Architecture

The frontend is organized around four major concepts:

```text
Pages
   ↓
Page JavaScript
   ↓
Services / API Layer
   ↓
ASP.NET Core API
```

Shared UI is separated from role-specific UI:

```text
Shared Components
        +
Shared Authentication
        +
Role-Specific Pages
        +
Role-Specific Services
        ↓
Complete TechCare Frontend
```

The frontend should not mix page markup, API requests, and business logic in one large JavaScript file.

---

# 4. Recommended Frontend Structure

```text
frontend/
│
├── documentation/
│   ├── 01-authentication/
│   │   ├── login.md
│   │   ├── register.md
│   │   ├── role-selection.md
│   │   ├── otp-verification.md
│   │   ├── password-recovery.md
│   │   └── logout.md
│   │
│   ├── 02-registration-forms/
│   │   ├── patient/
│   │   │   └── patient-registration.md
│   │   ├── doctor/
│   │   │   └── doctor-registration.md
│   │   ├── nurse/
│   │   │   └── nurse-registration.md
│   │   ├── pharmacy/
│   │   │   └── pharmacy-registration.md
│   │   ├── laboratory/
│   │   │   └── laboratory-registration.md
│   │   └── blood-donor/
│   │       └── blood-donor-registration.md
│   │
│   ├── 03-dashboards/
│   │   ├── patient/
│   │   │   └── patient-dashboard.md
│   │   ├── doctor/
│   │   │   └── doctor-dashboard.md
│   │   ├── nurse/
│   │   │   └── nurse-dashboard.md
│   │   ├── pharmacy/
│   │   │   └── pharmacy-dashboard.md
│   │   ├── laboratory/
│   │   │   └── laboratory-dashboard.md
│   │   ├── blood-donor/
│   │   │   └── blood-donor-dashboard.md
│   │   └── admin/
│   │       └── admin-dashboard.md
│   │
│   ├── 04-dashboard-features/
│   │   ├── profile.md
│   │   ├── notifications.md
│   │   ├── payments.md
│   │   ├── ratings.md
│   │   ├── complaints.md
│   │   ├── bookings.md
│   │   ├── medical-records.md
│   │   ├── search.md
│   │   └── location.md
│   │
│   ├── 05-shared-components/
│   │   ├── navbar.md
│   │   ├── sidebar.md
│   │   ├── footer.md
│   │   ├── buttons.md
│   │   ├── forms.md
│   │   ├── modals.md
│   │   ├── tables.md
│   │   ├── cards.md
│   │   ├── alerts.md
│   │   └── loading-states.md
│   │
│   ├── 06-ui-ux/
│   │   ├── design-system.md
│   │   ├── colors.md
│   │   ├── typography.md
│   │   ├── spacing.md
│   │   ├── responsive-design.md
│   │   └── accessibility.md
│   │
│   └── 07-frontend-standards/
│       ├── folder-structure.md
│       ├── naming-conventions.md
│       ├── javascript-standards.md
│       ├── css-standards.md
│       ├── html-standards.md
│       ├── api-integration.md
│       ├── error-handling.md
│       └── git-workflow.md
│
├── pages/
│   ├── public/
│   ├── auth/
│   ├── patient/
│   ├── doctor/
│   ├── nurse/
│   ├── pharmacy/
│   ├── laboratory/
│   ├── blood-donor/
│   └── admin/
│
├── components/
│   ├── navbar/
│   ├── sidebar/
│   ├── footer/
│   ├── buttons/
│   ├── forms/
│   ├── cards/
│   ├── modals/
│   ├── alerts/
│   ├── tables/
│   ├── loaders/
│   ├── steppers/
│   └── uploaders/
│
├── css/
│   ├── base/
│   ├── components/
│   ├── pages/
│   ├── auth/
│   ├── patient/
│   ├── doctor/
│   ├── nurse/
│   ├── pharmacy/
│   ├── laboratory/
│   ├── blood-donor/
│   └── admin/
│
├── js/
│   ├── core/
│   ├── utils/
│   ├── components/
│   ├── services/
│   ├── auth/
│   └── pages/
│       ├── patient/
│       ├── doctor/
│       ├── nurse/
│       ├── pharmacy/
│       ├── laboratory/
│       ├── blood-donor/
│       └── admin/
│
├── assets/
│   ├── images/
│   ├── icons/
│   ├── logos/
│   ├── illustrations/
│   └── documents/
│
└── README.md
```

The exact physical implementation can evolve, but ownership and module boundaries must remain clear.

---

# 5. Page Organization

Each role should have its own frontend page area.

Example:

```text
pages/
└── nurse/
    └── registration/
        ├── nurse-register.html
        ├── nurse-personal-info.html
        ├── nurse-professional-info.html
        ├── nurse-service-info.html
        ├── nurse-location-availability.html
        ├── nurse-documents.html
        ├── nurse-review.html
        └── nurse-registration-success.html
```

The same principle applies to Doctor, Pharmacy, Patient, Laboratory, and Blood Donor.

Shared pages belong to shared areas:

```text
pages/
├── public/
└── auth/
```

---

# 6. Authentication Frontend

Authentication is centralized.

There must be one shared authentication experience rather than a separate login and OTP implementation for every role.

### Authentication pages

```text
pages/auth/
├── login.html
├── register.html
├── role-selection.html
├── otp-verification.html
├── forgot-password.html
├── reset-password.html
└── logout handling
```

### Authentication responsibilities

The frontend authentication experience covers:

- Account entry
- Login
- Registration entry
- Role selection
- OTP verification
- Password recovery
- Password reset
- Logout
- Authentication state
- Session/token handling at the frontend integration layer
- Unauthorized/expired-session UI

### Authentication flow

```text
Open TechCare
    ↓
Register / Login
    ↓
Role Selection
    ↓
Authentication / Verification
    ↓
Authenticated User
    ↓
Role-Based Frontend
```

Admin public registration is not part of the normal public registration flow unless explicitly enabled by platform rules.

---

# 7. Registration Forms

TechCare has six registration modules.

```text
Registration Forms
│
├── Patient
├── Doctor
├── Nurse
├── Pharmacy
├── Laboratory
└── Blood Donor
```

All registrations should follow the same high-level UX pattern:

```text
Role Selection
      ↓
Account
      ↓
Verification
      ↓
Role Information
      ↓
Review
      ↓
Submit
      ↓
Success / Verification State
```

Role-specific forms may contain different steps and fields.

---

# 8. Nurse Registration

Responsible: **Ahmed**

Documentation:

```text
documentation/02-registration-forms/nurse/nurse-registration.md
```

Recommended pages:

```text
pages/nurse/registration/
├── nurse-register.html
├── nurse-personal-info.html
├── nurse-professional-info.html
├── nurse-service-info.html
├── nurse-location-availability.html
├── nurse-documents.html
├── nurse-review.html
└── nurse-registration-success.html
```

Nurse registration collects:

### Account

- Full Name
- Email
- Phone
- Password
- Confirm Password
- Terms and Privacy Agreement

### Personal Information

- First Name
- Middle Name
- Last Name
- Date of Birth
- Gender
- National ID Number
- Profile Picture
- Governorate
- City
- Detailed Address

### Professional Information

- Professional Title
- Nursing Qualification
- Professional License Number
- Years of Experience
- Skills
- Services Offered
- Professional Bio

### Services and Pricing

- Service Type
- Base Service Price
- Price Per Kilometer
- Service Radius
- Visit Duration
- Additional Service Fees
- Currency

### Location and Availability

- Governorate
- City
- Detailed Address
- Latitude
- Longitude
- Current Location option
- Working Days
- Start Time
- End Time
- Home Visit Availability

### Documents

- National ID Front
- National ID Back
- Nursing Certificate
- Professional/Practice License
- Additional Supporting Document

### Final states

```text
Submitted
   ↓
PENDING_VERIFICATION
   ↓
Admin Review
   ↓
Approved / Rejected / Additional Information Required
```

---

# 9. Doctor Registration

Responsible: **Ahd / عهد**

Documentation:

```text
documentation/02-registration-forms/doctor/doctor-registration.md
```

Recommended pages:

```text
pages/doctor/registration/
├── doctor-register.html
├── doctor-personal-info.html
├── doctor-professional-info.html
├── doctor-specialization-info.html
├── doctor-service-info.html
├── doctor-location-availability.html
├── doctor-documents.html
├── doctor-review.html
└── doctor-registration-success.html
```

Doctor registration includes:

### Account

- Full Name
- Email
- Phone
- Password
- Confirm Password
- Terms and Privacy Agreement

### Personal

- First Name
- Middle Name
- Last Name
- Date of Birth
- Gender
- National ID
- Profile Picture
- Governorate
- City
- Detailed Address

### Professional

- Professional Title
- Medical Qualification
- University/Institution
- Graduation Year
- Professional License Number
- Years of Experience
- Professional Bio

### Specialization

- Main Specialty
- Sub-Specialty
- Areas of Expertise
- Specialty Description

### Services and Pricing

- Service Type
- Service Price
- Price Per Kilometer when applicable
- Service Radius when applicable
- Consultation Duration
- Additional Fees
- Currency
- Services Offered
- Home Visit option when applicable

### Location and Availability

- Governorate
- City
- Detailed Address
- Latitude
- Longitude
- Current Location
- Home Visit Availability
- Working Days
- Start Time
- End Time

### Verification Documents

- National ID Front
- National ID Back
- Medical Degree/Graduation Certificate
- Professional Practice License
- Additional supporting documents according to business rules

### Final lifecycle

```text
Submitted
   ↓
PENDING_VERIFICATION
   ↓
Verification
   ↓
Approved / Rejected / Additional Information Required
```

---

# 10. Pharmacy Registration

Responsible: **Alaa / آلاء**

Documentation:

```text
documentation/02-registration-forms/pharmacy/pharmacy-registration.md
```

Recommended pages:

```text
pages/pharmacy/registration/
├── pharmacy-register.html
├── pharmacist-info.html
├── pharmacy-info.html
├── branch-info.html
├── medicine-services.html
├── location-hours.html
├── pharmacy-documents.html
├── pharmacy-review.html
└── pharmacy-registration-success.html
```

The Pharmacy module must distinguish between:

```text
Pharmacist
    +
Pharmacy
    +
Branch
```

### Pharmacist Information

- First Name
- Middle Name
- Last Name
- Date of Birth
- Gender
- National ID
- Pharmacist License Number
- Qualification
- Experience

### Pharmacy Information

- Pharmacy Name
- Pharmacy Type
- Pharmacy License Number
- Commercial Registration where required
- Pharmacy Phone
- Pharmacy Email
- Pharmacy Description
- Pharmacy Logo

### Primary Branch

- Branch Name
- Branch Phone
- Governorate
- City
- Detailed Address
- Latitude
- Longitude

### Medicines and Services

- Services Provided
- Medicine Availability mode
- Home Delivery
- Delivery Radius when applicable
- Service Description

Registration should not become full inventory management.

Inventory management belongs to the Pharmacy Dashboard.

### Location and Hours

- Governorate
- City
- Address
- Coordinates
- Current Location
- Operating Days
- Opening Time
- Closing Time
- 24-hour option

### Verification Documents

- Pharmacist ID Front/Back
- Pharmacist Qualification
- Pharmacist License
- Pharmacy License
- Commercial Registration when required
- Ownership/Authorization documentation when required
- Additional supporting documents

---

# 11. Patient Registration

Responsible: **Eman / إيمان**

Documentation:

```text
documentation/02-registration-forms/patient/patient-registration.md
```

Recommended pages:

```text
pages/patient/registration/
├── patient-register.html
├── patient-personal-info.html
├── patient-medical-info.html
├── patient-location.html
├── patient-emergency-contact.html
├── patient-review.html
└── patient-registration-success.html
```

### Account

- Full Name
- Email
- Phone
- Password
- Confirm Password
- Terms and Privacy Agreement

### Personal

- First Name
- Middle Name
- Last Name
- Date of Birth
- Gender
- National ID according to business rules
- Profile Picture

### Medical Information

- Blood Type
- Previous Medical Conditions
- Current Medications
- Allergies
- Additional Medical Notes

Medical data is sensitive and must be handled carefully.

### Location

- Governorate
- City
- Address
- Latitude
- Longitude
- Current Location

### Emergency Contact

- Contact Name
- Relationship
- Phone Number
- Alternative Phone where applicable

### Final state

Patient registration creates the patient profile according to backend account rules. Provider-style professional verification is not required for a normal patient role.

---

# 12. Blood Donor Registration

Responsible: **Eman / إيمان**

Documentation:

```text
documentation/02-registration-forms/blood-donor/blood-donor-registration.md
```

Recommended pages:

```text
pages/blood-donor/registration/
├── blood-donor-register.html
├── donor-personal-info.html
├── donor-blood-info.html
├── donor-location.html
├── donor-availability.html
├── donor-review.html
└── donor-registration-success.html
```

### Personal

- First Name
- Middle Name
- Last Name
- Date of Birth
- Gender
- National ID according to business rules
- Profile Picture

### Blood and Donation Information

- Blood Type
- Rh Factor
- Previous Donation Experience
- Last Donation Date when applicable
- Donation Availability
- Additional Information

### Location

- Governorate
- City
- Area/Address
- Latitude
- Longitude
- Current Location

### Availability

- Current Availability
- Preferred Donation Days
- Preferred Time
- Notification Preferences according to product capabilities

The frontend must not make a medical eligibility decision. Eligibility and donation rules are authoritative backend/business logic.

---

# 13. Laboratory Registration

Responsible: **Mostafa**

Documentation:

```text
documentation/02-registration-forms/laboratory/laboratory-registration.md
```

Recommended frontend module:

```text
pages/laboratory/registration/
```

The Laboratory registration experience should cover:

### Account

- Authorized representative/account information
- Email
- Phone
- Password
- Verification

### Laboratory Profile

- Laboratory Name
- Laboratory Type
- License information
- Contact details
- Description
- Logo where applicable

### Services

- Laboratory Test Categories
- Services Offered
- Home Sample Collection when supported
- Service pricing model where applicable

### Location

- Governorate
- City
- Address
- Coordinates
- Coverage/service radius where applicable

### Operating Hours

- Working Days
- Opening Time
- Closing Time
- 24-hour option where supported

### Documents

- Identity
- Professional/License documentation
- Laboratory license
- Supporting documentation

### Final state

```text
Submitted
   ↓
Pending Verification
   ↓
Admin Review
```

The exact field contract must be aligned with the backend implementation.

---

# 14. Admin Frontend

Admin UI is responsible for platform operations.

Recommended area:

```text
pages/admin/
```

Admin dashboard may include:

- Overview
- User Management
- Provider Management
- Verification Queue
- Complaints
- Ratings/Moderation
- Booking Monitoring
- Payment Monitoring
- Notifications
- Reports
- Platform Configuration
- Audit-oriented views

Admin pages must be protected by backend authorization and must never rely only on hidden frontend controls.

---

# 15. Shared Dashboard Features

The frontend documentation reserves common dashboard feature specifications in:

```text
documentation/04-dashboard-features/
```

The shared feature areas include:

```text
Profile
Notifications
Payments
Ratings
Complaints
Bookings
Medical Records
Search
Location
```

The same capability may be presented differently depending on the role.

Example:

```text
Patient → My Bookings
Doctor  → Booking Requests / Appointments
Nurse   → Home Visit Requests
Pharmacy → Pharmacy/Branch operations
Admin   → Platform-wide monitoring
```

---

# 16. Booking Frontend

Booking is a core TechCare interaction.

General flow:

```text
Search
  ↓
Provider
  ↓
Service
  ↓
Location
  ↓
Availability
  ↓
Select Slot
  ↓
Price Preview
  ↓
Create Request
  ↓
Pending
  ↓
Accepted / Rejected / Expired
  ↓
Scheduled
  ↓
In Progress
  ↓
Completed
```

The frontend must visually support request statuses.

Possible UI status values:

```text
PENDING
ACCEPTED
REJECTED
EXPIRED
SCHEDULED
IN_PROGRESS
COMPLETED
CANCELLED
```

The final enum values must match the backend contract.

---

# 17. Search and Discovery Frontend

Search is a shared patient-facing capability.

Search may support:

```text
Doctors
Nurses
Pharmacies
Medicines
Laboratories
Blood Donation
```

Common search UI:

```text
Search Input
     ↓
Filters
     ↓
Sort
     ↓
Results
     ↓
Provider/Service Details
```

Useful filters may include:

- Location
- Specialty
- Service
- Availability
- Rating
- Price
- Distance

The frontend should not calculate or trust sensitive/authoritative values when those values come from backend logic.

---

# 18. Location UX

Location is a shared frontend concern.

The frontend may support:

```text
Current Location
+
Manual Address
+
Latitude
+
Longitude
```

General flow:

```text
User requests current location
        ↓
Browser permission
        ↓
Geolocation result
        ↓
Latitude / Longitude
        ↓
Form state
        ↓
Backend
```

Possible states:

- Detecting
- Success
- Permission denied
- Unavailable
- Timeout
- Unsupported
- Manual-entry fallback

Exact location should not automatically be displayed to unauthorized users.

---

# 19. Medical Records Frontend

Medical records are sensitive.

The frontend should support role-aware access such as:

```text
Patient → Own Medical Records
Doctor  → Authorized Relevant Records
Admin   → Controlled operational views only where explicitly permitted
```

The frontend must not assume that a record is accessible simply because an ID appears in the URL.

Avoid patterns such as:

```text
/medical-record?id=123
```

as a security mechanism.

Authorization must come from the backend.

---

# 20. Notifications Frontend

Notification UI should provide:

```text
Notification List
Unread Count
Read / Unread State
Notification Detail
Time
Related Entity/Action
```

Examples:

```text
Booking request received
Booking accepted
Appointment reminder
Verification status changed
Complaint update
Payment status update
Blood donation request
```

Notification states should be visually clear without relying only on color.

---

# 21. Payments Frontend

Payment UI may include:

```text
Service Price
Travel Fee
Additional Fee
Total
Payment Status
Payment Method
Transaction Status
```

The frontend can preview a total but the backend remains the authority for financial calculations.

Never trust client-provided prices.

Example:

```text
Base Service Price
+
Travel Fee
+
Additional Fee
=
Server-Validated Total
```

---

# 22. Ratings Frontend

Rating UI should support:

```text
Star Rating
Optional Comment
Submit Rating
Edit where allowed
Average Rating Display
```

Only completed/eligible transactions or bookings should be able to generate ratings according to backend rules.

---

# 23. Complaints Frontend

Complaint UI should support:

```text
Create Complaint
Select Related Booking / Provider
Enter Description
Submit
View Status
View Updates
```

Possible statuses:

```text
OPEN
UNDER_REVIEW
RESOLVED
REJECTED
```

The exact state names must follow the backend contract.

---

# 24. Shared Components

The frontend should maintain reusable UI components.

### Navigation

- Navbar
- Sidebar
- Footer
- Breadcrumbs

### Forms

- Text Input
- Email Input
- Phone Input
- Password Input
- Number Input
- Date Picker
- Select
- Multi-select
- Checkbox
- Radio
- Toggle
- Textarea
- File Upload

### Feedback

- Alerts
- Toasts
- Modals
- Confirmation dialogs
- Loading states
- Skeleton states
- Empty states
- Error states
- Success states

### Data Display

- Cards
- Tables
- Badges
- Status indicators
- Pagination

### Registration

- Progress stepper
- Form section
- Review summary
- Upload component

---

# 25. Component State Standard

Interactive components should consider these states:

```text
Default
Hover
Focus
Filled
Selected
Success
Error
Disabled
Loading
```

Example input:

```text
Label
Input
Helper Text
Error / Success Feedback
```

Do not rely on color alone for a state.

---

# 26. UI/UX Design System

The frontend design system is documented in:

```text
documentation/06-ui-ux/
```

It contains:

```text
design-system.md
colors.md
typography.md
spacing.md
responsive-design.md
accessibility.md
```

### General visual direction

TechCare should feel:

- Clean
- Trustworthy
- Professional
- Modern
- Healthcare-oriented
- Human
- Accessible
- Calm

The TechCare logo should be the source of the primary brand colors.

The interface should use the brand palette while keeping the UI design original and not simply reproducing the logo's shapes or geometry.

---

# 27. Color Usage

Brand colors should be used with restraint.

Recommended hierarchy:

```text
Neutral surfaces
    ↓
Primary brand color
    ↓
Secondary brand color
    ↓
Status colors
```

Brand colors can be used for:

- Primary buttons
- Active navigation
- Focus states
- Selected controls
- Progress indicators
- Important links
- Small brand accents

Avoid flooding entire screens with strong color.

---

# 28. Typography

Typography should prioritize:

- Readability
- Strong hierarchy
- Comfortable line height
- Clear labels
- Accessible sizing
- Consistency

Hierarchy example:

```text
Page Title
    ↓
Section Heading
    ↓
Field Label
    ↓
Body
    ↓
Helper / Error Text
```

Avoid decorative/futuristic fonts.

---

# 29. Spacing

Use a consistent spacing system rather than arbitrary margins.

Spacing should remain consistent across:

- Forms
- Cards
- Sections
- Navigation
- Buttons
- Dashboard content
- Mobile layouts

Whitespace is a major part of TechCare's visual identity.

---

# 30. Responsive Design

Every frontend page must support at least:

```text
Desktop
Tablet
Mobile
```

Typical behavior:

### Desktop

- Multi-column layout where useful
- Spacious container
- Visible sidebar/progress when appropriate

### Tablet

- Reduced columns
- Flexible widths
- Simplified side navigation

### Mobile

- Single-column layout
- Large touch targets
- Full-width primary controls
- Compact progress indicator
- No horizontal scrolling
- Comfortable spacing

Never treat responsive design as only shrinking desktop elements.

---

# 31. Accessibility

Accessibility is a core frontend requirement.

The frontend should support:

- Semantic HTML
- Correct labels
- Logical tab order
- Keyboard navigation
- Visible focus states
- Accessible buttons
- Accessible inputs
- Accessible file uploads
- Sufficient contrast
- Clear validation
- No color-only meaning
- Large touch targets
- Readable text

TechCare must be comfortable for users with different levels of digital and physical ability.

---

# 32. Forms Standard

Every form should follow the same basic pattern:

```text
Page Title
↓
Short Explanation
↓
Section
↓
Label
↓
Input
↓
Helper Text
↓
Validation
↓
Navigation
```

Required fields should be clearly identified.

Example:

```text
Professional License Number *
[____________________________]

Enter your official professional license number.

This field is required.
```

---

# 33. Form Validation

Validation occurs at two levels:

```text
Frontend Validation
        +
Backend Validation
```

Frontend validation improves UX.

Backend validation is authoritative.

Typical frontend validation includes:

- Required fields
- Format
- Length
- Matching values
- Positive/non-negative numeric input
- Valid date
- Conditional fields
- File type and size where applicable

Do not treat JavaScript validation as a security boundary.

---

# 34. Error Handling

Frontend errors should be human-readable.

Avoid:

```text
Error 400
Invalid input
Request failed
```

Prefer:

```text
Please enter a valid phone number.

This field is required.

We couldn't submit your registration. Please review the highlighted fields and try again.
```

The frontend should support:

```text
Validation Error
Network Error
Server Error
Authentication Error
Session Expiration
Upload Error
Duplicate Data Error
Unexpected Error
```

---

# 35. Loading States

Asynchronous operations must have visible states.

Examples:

```text
Loading cities...
Detecting location...
Uploading document...
Verifying code...
Submitting registration...
```

Disable actions when necessary to prevent duplicate requests.

---

# 36. Duplicate Submission Protection

When a user submits:

```text
Submit
  ↓
Disable Button
  ↓
Show Loading
  ↓
Send Request
  ↓
Wait for Response
```

Do not allow accidental repeated requests through double-clicking.

The backend must also protect against duplicate operations.

---

# 37. File Upload Standards

File uploads must show:

- Required/optional status
- Accepted types
- Size guidance
- Filename
- File size
- Upload progress
- Success state
- Error state
- Replace
- Remove

Sensitive documents should not be unnecessarily previewed or exposed.

The backend must validate file type, size, content, and permissions.

---

# 38. API Integration

Frontend API communication should be centralized in service modules.

Example:

```text
js/services/
├── auth.service.js
├── patient.service.js
├── doctor.service.js
├── nurse.service.js
├── pharmacy.service.js
├── laboratory.service.js
└── blood-donor.service.js
```

Recommended flow:

```text
HTML Page
    ↓
Page JavaScript
    ↓
Service Layer
    ↓
HTTP Request
    ↓
ASP.NET Core API
```

Pages should not contain repeated raw fetch logic wherever a reusable service is appropriate.

---

# 39. API Request Responsibilities

A service module should handle:

- Request construction
- Endpoint integration
- Headers
- Authentication integration
- Request body
- Response handling
- Error normalization where appropriate

Example conceptual service:

```javascript
const nurseService = {
    createRegistration: async (payload) => {
        // request
    },

    uploadDocument: async (file) => {
        // request
    },

    submitRegistration: async (payload) => {
        // request
    }
};
```

Exact endpoints must be taken from the backend API contract.

Do not invent endpoint URLs in the frontend.

---

# 40. Frontend Authentication State

The frontend must know the current UI authentication state.

Possible states:

```text
Unauthenticated
Authenticating
Authenticated
Session Expired
Unauthorized
```

Role-aware frontend routing should redirect users to the correct experience.

Examples:

```text
Patient → Patient UI
Doctor → Doctor UI
Nurse → Nurse UI
Pharmacy → Pharmacy UI
Laboratory → Laboratory UI
Blood Donor → Donor UI
Admin → Admin UI
```

The backend remains authoritative for roles and permissions.

---

# 41. Authorization in the Frontend

The frontend can hide or show UI based on role/state for user experience.

However:

```text
Frontend Hide/Show
        ≠
Security
```

Every protected operation must also be authorized by the backend.

Never rely on:

- Hidden buttons
- Hidden inputs
- Client-side role strings
- URL manipulation
- Local UI state

as a security control.

---

# 42. Sensitive Data Rules

Sensitive information may include:

- National ID
- Medical conditions
- Medications
- Allergies
- Professional licenses
- Verification documents
- Addresses
- Location coordinates
- Payment information

Frontend rules:

- Do not log sensitive data to the browser console
- Do not put sensitive data unnecessarily in URLs
- Do not expose private data to unauthorized users
- Mask sensitive values where practical
- Avoid unnecessary local persistence
- Use authenticated API calls
- Respect backend authorization
- Avoid displaying sensitive information in generic error messages

---

# 43. Local Storage and Session Storage

Storage must be used carefully.

Safe candidates may include non-sensitive UI preferences.

Avoid storing highly sensitive information such as:

```text
Medical records
National IDs
Uploaded verification documents
Private passwords
```

Authentication storage strategy must follow the backend/frontend security design agreed by the project team.

---

# 44. Naming Conventions

### HTML Files

Use lowercase kebab-case:

```text
nurse-register.html
doctor-personal-info.html
pharmacy-documents.html
```

### CSS Files

Use lowercase kebab-case:

```text
nurse-registration.css
doctor-registration.css
```

### JavaScript Files

Use lowercase kebab-case:

```text
nurse-registration.js
```

### Service Files

Use:

```text
nurse.service.js
doctor.service.js
patient.service.js
```

### IDs and Classes

Prefer descriptive names:

```text
registration-form
license-number
submit-registration
```

Avoid meaningless names such as:

```text
box1
div2
x
test123
```

---

# 45. JavaScript Standards

Use modern JavaScript.

Prefer:

```javascript
const
let
async/await
modules
destructuring
template literals
Array methods
```

Avoid unnecessary:

```javascript
var
global variables
duplicated fetch code
giant page functions
inline event handlers
```

Keep functions focused.

Bad:

```text
One 500-line function that handles:
UI + validation + API + routing + uploads
```

Better:

```text
Validation
UI rendering
State handling
API service
Navigation
```

are separated logically.

---

# 46. CSS Standards

Use:

- Reusable classes
- CSS variables
- Consistent spacing
- Component styles
- Role/page-specific styles where needed
- Responsive media queries
- Clear naming

Avoid:

- Excessive inline styles
- !important everywhere
- Duplicating the same CSS across role pages
- Random color values throughout the code

Prefer centralized design tokens.

Example:

```css
:root {
    --color-primary: ...;
    --color-secondary: ...;
    --color-text: ...;
    --color-muted: ...;
    --color-border: ...;
    --color-success: ...;
    --color-error: ...;

    --radius-sm: ...;
    --radius-md: ...;

    --space-1: ...;
    --space-2: ...;
    --space-3: ...;
}
```

Actual brand values must come from the final TechCare design system.

---

# 47. HTML Standards

Use semantic HTML whenever possible.

Prefer:

```html
<header>
<nav>
<main>
<section>
<form>
<label>
<button>
```

Inputs must have accessible labels.

Example:

```html
<label for="professionalLicense">
    Professional License Number
</label>

<input
    id="professionalLicense"
    name="professionalLicense"
    type="text"
>
```

Avoid using clickable `<div>` elements when a semantic `<button>` or `<a>` is appropriate.

---

# 48. Frontend Documentation

All important frontend behavior must be documented.

The documentation root is:

```text
frontend/documentation/
```

Documentation should explain:

- Purpose
- Page structure
- Fields
- Validation
- User interactions
- UI states
- Navigation
- Responsive behavior
- Accessibility
- API integration
- Ownership boundaries
- Related modules

Documentation is part of the project and should be updated with significant UI behavior changes.

---

# 49. Registration Documentation Ownership

| Module | Owner |
|---|---|
| Authentication | Mostafa |
| Nurse Registration | Ahmed |
| Doctor Registration | Ahd |
| Pharmacy Registration | Alaa |
| Patient Registration | Eman |
| Blood Donor Registration | Eman |
| Laboratory Registration | Mostafa |

---

# 50. Authentication Documentation Ownership

Authentication documentation:

```text
documentation/01-authentication/
```

Primary ownership:

```text
Mostafa
```

Files:

```text
login.md
register.md
role-selection.md
otp-verification.md
password-recovery.md
logout.md
```

Shared authentication behavior must stay centralized.

---

# 51. UI/UX Documentation Ownership

The UI/UX documentation defines:

```text
Design System
Colors
Typography
Spacing
Responsive Design
Accessibility
```

Every contributor must follow it.

When a role-specific UI needs a new component, the team should reuse an existing component whenever possible before creating a new one.

---

# 52. Frontend Git Workflow

The team uses GitHub and works in parallel.

Each contributor should work on their assigned module/branch rather than editing the same role implementation at the same time.

General workflow:

```text
Update Main
    ↓
Create / Switch to Your Branch
    ↓
Implement Feature
    ↓
Test
    ↓
Commit
    ↓
Push
    ↓
Pull Request
    ↓
Code Review
    ↓
Merge
```

---

# 53. Team Ownership Boundaries

### Mostafa

Owns:

```text
Authentication
Authorization-related frontend integration
Laboratory Registration
```

### Ahmed

Owns:

```text
Nurse Registration
```

### Ahd

Owns:

```text
Doctor Registration
```

### Alaa

Owns:

```text
Pharmacy Registration
```

### Eman

Owns:

```text
Patient Registration
Blood Donor Registration
```

Each developer should avoid directly changing another developer's role-specific module unless the team agrees.

Shared component changes should be coordinated before being merged.

---

# 54. Shared vs Role-Specific Code

### Shared

Examples:

```text
Navbar
Footer
Sidebar
Buttons
Inputs
Modals
Alerts
Loading
Stepper
Auth UI
Global utility functions
API client conventions
```

### Role-specific

Examples:

```text
Nurse registration
Doctor specialization
Pharmacy branches
Patient medical information
Blood donor availability
Laboratory services
```

Avoid duplicating shared code in each role.

---

# 55. Registration UX Rules

Every registration module should provide:

```text
Clear Page Title
Short Explanation
Progress Indicator
Well-Grouped Fields
Required Indicators
Inline Validation
Back
Continue
Loading State
Error State
Review
Submit
Success State
```

The user should always understand:

```text
Where am I?
What do I need?
Why is it needed?
What is wrong?
How do I fix it?
What happens next?
```

---

# 56. Success and Verification States

Professional provider registrations must distinguish between:

```text
Registration Submitted
```

and:

```text
Provider Approved
```

Example:

```text
Registration Submitted
        ↓
Pending Verification
        ↓
Admin Review
        ↓
Approved
```

Do not redirect an unverified professional directly into a fully activated provider dashboard unless the backend authorizes the account.

---

# 57. Frontend Testing Strategy

Every frontend module should be tested for:

### Functional behavior

- Navigation
- Form submission
- Validation
- File upload
- API integration
- Error handling
- State persistence

### Responsive behavior

- Desktop
- Tablet
- Mobile

### Accessibility

- Keyboard navigation
- Focus
- Labels
- Errors
- Contrast
- Touch targets

### Edge cases

- Empty values
- Invalid formats
- Very long values
- Duplicate requests
- Network failure
- API failure
- Expired sessions
- Upload failure
- Location denied

---

# 58. Manual QA Checklist

Before marking a frontend feature complete:

```text
[ ] UI matches approved design
[ ] Desktop works
[ ] Tablet works
[ ] Mobile works
[ ] All required fields exist
[ ] Required indicators are clear
[ ] Validation works
[ ] Error messages are understandable
[ ] Loading states exist
[ ] Empty states exist where needed
[ ] Success states exist
[ ] Buttons cannot trigger duplicate requests
[ ] Back navigation works
[ ] Continue navigation works
[ ] Form data is preserved
[ ] API integration works
[ ] Unauthorized states are handled
[ ] Sensitive data is not logged
[ ] Accessibility basics are satisfied
```

---

# 59. Definition of Done — Frontend

A feature is not complete simply because the page visually exists.

A frontend feature is considered done when:

```text
UI
+
Responsive Behavior
+
Validation
+
States
+
API Integration
+
Error Handling
+
Accessibility
+
Security-Aware Implementation
+
Documentation
+
Testing
```

are all addressed.

---

# 60. Design Review Checklist

Before approving a screen:

```text
[ ] Is the hierarchy clear?
[ ] Is the primary action obvious?
[ ] Is there unnecessary decoration?
[ ] Are forms easy to scan?
[ ] Are labels readable?
[ ] Are inputs comfortable to use?
[ ] Are error states understandable?
[ ] Does mobile work properly?
[ ] Is the page consistent with TechCare branding?
[ ] Does the screen reuse shared components?
```

---

# 61. Performance Principles

The frontend should avoid unnecessary performance issues.

Guidelines:

- Optimize image sizes
- Use appropriately sized assets
- Avoid loading unnecessary resources
- Keep JavaScript modular
- Avoid repeated API calls
- Avoid unnecessary DOM updates
- Lazy-load heavy resources where appropriate
- Keep CSS organized
- Avoid excessive third-party libraries

The frontend should remain fast on normal mobile connections.

---

# 62. Asset Management

Recommended structure:

```text
assets/
├── images/
├── icons/
├── logos/
├── illustrations/
└── documents/
```

Use meaningful filenames.

Examples:

```text
techcare-logo.svg
nurse-placeholder.webp
location-pin.svg
verification-shield.svg
```

Do not use:

```text
image1.png
new2.png
final-final.png
abc.png
```

---

# 63. Frontend Security Principles

The frontend must be security-aware, but security enforcement belongs to the backend.

Never trust client-side:

```text
Role
User ID
Provider Status
Verification Status
Price
Permissions
Ownership
```

The backend must validate and authorize these values.

Frontend responsibilities include:

- Avoiding sensitive logs
- Handling tokens/session state according to project security design
- Avoiding sensitive URLs
- Displaying only authorized data returned by backend
- Clearing UI state on logout/session expiration
- Handling unauthorized responses correctly

---

# 64. General Navigation Model

Public:

```text
Home
Services
Search / Discovery
About / Informational Pages
Login
Register
```

Authenticated navigation changes by role.

Patient:

```text
Dashboard
Search
Bookings
Medical Records
Notifications
Payments
Profile
```

Provider:

```text
Dashboard
Requests / Appointments
Services
Availability
Earnings / Payments where applicable
Ratings
Complaints
Profile
Notifications
```

Pharmacy and Laboratory navigation may include additional role-specific operations.

Admin navigation is separate and permission-controlled.

---

# 65. Overall Frontend User Journey

```text
                    TechCare
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
        Login                    Register
          │                         │
          │                    Select Role
          │                         │
          │          ┌──────────────┼──────────────┐
          │          ↓              ↓              ↓
          │       Patient        Provider       Donor
          │                         │
          │             ┌───────────┼───────────────┐
          │             ↓           ↓               ↓
          │           Doctor      Nurse         Pharmacy
          │             │           │
          │             ↓           ↓
          │         Verification / Activation
          │
          └──────────────────────┐
                                 ↓
                         Role-Based Experience
```

---

# 66. Frontend Module Map

```text
TechCare Frontend
│
├── Public
│
├── Authentication
│
├── Patient
│   ├── Registration
│   └── Dashboard
│
├── Doctor
│   ├── Registration
│   └── Dashboard
│
├── Nurse
│   ├── Registration
│   └── Dashboard
│
├── Pharmacy
│   ├── Registration
│   └── Dashboard
│
├── Laboratory
│   ├── Registration
│   └── Dashboard
│
├── Blood Donor
│   ├── Registration
│   └── Dashboard / Donation Features
│
└── Admin
    └── Dashboard
```

---

# 67. Future Frontend Expansion

The current frontend architecture should allow future features without restructuring the entire project.

Potential future modules include:

- AI symptom assistant
- Emergency button workflow
- Live location sharing
- Medical history transmission to authorized emergency destinations
- Laboratory home sample collection
- Rich appointment management
- Additional provider services
- Mobile application API consumers
- Advanced communication/messaging
- Advanced analytics
- Third-party healthcare integrations

These should be added as modular features rather than placed directly inside unrelated registration pages.

---

# 68. Important Architecture Rule

Do not turn registration into a dashboard.

Registration should answer:

```text
Who are you?
What information identifies you?
What professional/service information is required?
Where do you operate?
What documents are required?
Can we submit the registration?
```

Dashboard should answer:

```text
What can you do after registration?
What requests do you have?
What services are active?
What are your bookings?
What are your finances?
What notifications do you have?
What can you manage?
```

Keeping these responsibilities separate makes the frontend easier to understand and maintain.

---

# 69. Recommended Development Order

Frontend development can proceed in this dependency order:

```text
1. Design System
      ↓
2. Shared Components
      ↓
3. Authentication
      ↓
4. Registration Forms
      ↓
5. Role Dashboards
      ↓
6. Shared Dashboard Features
      ↓
7. Cross-Module Integrations
      ↓
8. QA / Accessibility / Responsive Review
```

This order reduces duplicate implementation.

---

# 70. Final Frontend Standard

Every TechCare frontend feature should satisfy this model:

```text
                    TECHCARE FRONTEND
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           UX/UI         Logic          API
             │             │             │
             ↓             ↓             ↓
        Accessibility   Validation   Integration
             │             │             │
             └─────────────┼─────────────┘
                           ↓
                     Production UI
```

The final frontend should feel like one product, not six unrelated websites.

Consistency must remain visible across:

- Colors
- Typography
- Spacing
- Buttons
- Inputs
- Navigation
- Statuses
- Forms
- Error messages
- Responsive behavior
- Accessibility

---

# 71. Frontend Success Criteria

The TechCare frontend is successful when:

```text
A new user can understand the platform quickly.

A patient can register without confusion.

A healthcare professional can complete a structured registration.

A provider can understand verification status.

A user can use the interface comfortably on mobile.

The design feels medically trustworthy without becoming visually outdated.

The five developers can work independently without creating duplicated architecture.

The frontend can integrate cleanly with the ASP.NET Core backend.

The system remains maintainable as TechCare grows.
```

---

# 72. Ownership Summary

| Team Member | Frontend Ownership |
|---|---|
| Mostafa | Authentication & Authorization frontend integration + Laboratory Registration |
| Ahmed | Nurse Registration |
| Ahd | Doctor Registration |
| Alaa | Pharmacy Registration |
| Eman | Patient Registration + Blood Donor Registration |

All team members follow the same:

```text
Design System
+
Shared Components
+
Frontend Standards
+
Git Workflow
+
API Integration Rules
+
Accessibility Rules
```

---

# 73. Related Documentation

Core frontend documentation:

```text
documentation/01-authentication/
documentation/02-registration-forms/
documentation/03-dashboards/
documentation/04-dashboard-features/
documentation/05-shared-components/
documentation/06-ui-ux/
documentation/07-frontend-standards/
```

The frontend README is the high-level entry point.

Detailed behavior belongs in the relevant documentation file.

---

# 74. Final Rule

Before adding a new frontend page, component, or pattern, ask:

```text
Does this already exist?
Can it be reused?
Does it belong to a shared module?
Does it belong to one role?
Is it documented?
Is the API contract clear?
Does it work on mobile?
Is it accessible?
Does it follow TechCare's design system?
```

Build TechCare as one cohesive healthcare product, not as a collection of isolated pages.

---

## End of TechCare Frontend README
