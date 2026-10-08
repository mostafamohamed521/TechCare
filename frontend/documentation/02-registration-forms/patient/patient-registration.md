# — Patient Registration

**File:** `frontend/documentation/02-registration-forms/patient/patient-registration.md`  
**Responsible:** Eman  
**Module:** Patient Registration  
**Application:** TechCare

## 1. Overview

The Patient Registration module allows users to create a TechCare patient account and provide the personal, contact, location, and basic medical information required to use the healthcare platform.

The patient registration process is designed to be simple and accessible because TechCare targets users who may include:

- Elderly Patients
- Less-Mobile Patients
- Patients With Disabilities
- General Users

The registration form should therefore avoid unnecessary complexity and should provide clear guidance at every step.

The overall process is:

```text
Patient Registration
        ↓
Account Creation
        ↓
OTP Verification
        ↓
Personal Information
        ↓
Medical Information
        ↓
Location Information
        ↓
Emergency / Contact Information
        ↓
Review
        ↓
Submit
        ↓
Patient Account Ready
```

Unlike professional providers, the patient does not require professional verification before using the normal patient features.

## 2. Ownership

Eman owns:

- Patient Registration

Specifically:

- Patient Registration Pages
- Patient Forms
- Patient Validation
- Patient Medical Information UI
- Patient Location UI
- Patient Review UI
- Patient Success UI
- Patient Registration JavaScript
- Patient Registration CSS
- Patient API Integration

Eman does not own shared Authentication.

## 3. Frontend Structure

Recommended structure:

```text
frontend/
│
├── pages/
│   └── patient/
│       └── registration/
│           ├── patient-register.html
│           ├── patient-personal-info.html
│           ├── patient-medical-info.html
│           ├── patient-location.html
│           ├── patient-emergency-contact.html
│           ├── patient-review.html
│           └── patient-registration-success.html
│
├── css/
│   └── patient/
│       └── patient-registration.css
│
├── js/
│   ├── pages/
│   │   └── patient/
│   │       └── patient-registration.js
│   │
│   └── services/
│       └── patient.service.js
│
└── assets/
    └── patient/
```

## 4. Complete Registration Flow

```text
┌──────────────────────────────┐
│          Register             │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│        Select Patient         │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       Account Information     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│     Shared OTP Verification   │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│      Personal Information     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       Medical Information     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│       Location Information    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│      Emergency Contact        │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│          Review               │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│          Submit               │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│     Registration Success      │
└──────────────────────────────┘
```

## 5. Registration Steps

| Step | Page | File |
|---|---|---|
| 1 | Patient Account | `patient-register.html` |
| 2 | Personal Information | `patient-personal-info.html` |
| 3 | Medical Information | `patient-medical-info.html` |
| 4 | Location | `patient-location.html` |
| 5 | Emergency Contact | `patient-emergency-contact.html` |
| 6 | Review | `patient-review.html` |
| 7 | Success | `patient-registration-success.html` |

OTP verification is shared with Authentication.

## 6. Registration Progress Indicator

Example:

```text
┌───────────────────────────────────────────────────────────────┐
│ Patient Registration                                          │
│                                                               │
│ ● Account → ● Personal → ◉ Medical → ○ Location              │
│                                                               │
│ ○ Emergency Contact → ○ Review                               │
│                                                               │
│ Step 3 of 6                                                   │
└───────────────────────────────────────────────────────────────┘
```

## 7. Step 1 – Patient Account

### Page

`frontend/pages/patient/registration/patient-register.html`

### UI Sketch

```text
┌──────────────────────────────────────────────────┐
│              Create Patient Account              │
├──────────────────────────────────────────────────┤
│                                                  │
│ Full Name *                                      │
│ [__________________________________________]     │
│                                                  │
│ Email Address *                                  │
│ [__________________________________________]     │
│                                                  │
│ Phone Number *                                   │
│ [__________________________________________]     │
│                                                  │
│ Password *                                       │
│ [_______________________________________ 👁 ]    │
│                                                  │
│ Confirm Password *                               │
│ [_______________________________________ 👁 ]    │
│                                                  │
│ [ ] I agree to Terms & Privacy Policy            │
│                                                  │
│                    [ Continue ]                  │
└──────────────────────────────────────────────────┘
```

## 8. Account Fields

| Field | Type | Required |
|---|---|---|
| Full Name | Text | Yes |
| Email | Email | Yes |
| Phone | Tel | Yes |
| Password | Password | Yes |
| Confirm Password | Password | Yes |
| Terms Agreement | Checkbox | Yes |

Validation follows the shared Authentication rules.

## 9. Shared OTP Verification

After account creation:

```text
Patient Registration
       ↓
Account Created
       ↓
OTP Verification
       ↓
Verified
       ↓
Continue Patient Registration
```

The Patient module does not implement its own OTP mechanism.

Shared documentation:

`frontend/documentation/01-authentication/otp-verification.md`

## 10. Step 2 – Personal Information

### Page

`frontend/pages/patient/registration/patient-personal-info.html`

### UI Sketch

```text
┌──────────────────────────────────────────────────┐
│             Personal Information                 │
├──────────────────────────────────────────────────┤
│                                                  │
│ First Name *                                     │
│ [__________________________________________]     │
│                                                  │
│ Middle Name                                      │
│ [__________________________________________]     │
│                                                  │
│ Last Name *                                      │
│ [__________________________________________]     │
│                                                  │
│ Date of Birth *                                  │
│ [____ / ____ / ______]                           │
│                                                  │
│ Gender *                                         │
│ [ Select Gender ▼ ]                              │
│                                                  │
│ National ID Number                               │
│ [__________________________________________]     │
│                                                  │
│ Profile Picture                                  │
│ [ Choose File ]                                  │
│                                                  │
│ [ Back ]                         [ Continue ]    │
└──────────────────────────────────────────────────┘
```

## 11. Personal Fields

| Field | Type | Required |
|---|---|---|
| First Name | Text | Yes |
| Middle Name | Text | No |
| Last Name | Text | Yes |
| Date of Birth | Date | Yes |
| Gender | Select/Radio | Yes |
| National ID Number | Text | Business Rule |
| Profile Picture | File | No |

## 12. Personal Validation

The frontend checks:

- First Name → Required
- Last Name → Required
- Date of Birth → Valid date
- Gender → Required
- National ID → Valid format when provided

The backend remains responsible for authoritative validation.

## 13. Step 3 – Medical Information

### Page

`frontend/pages/patient/registration/patient-medical-info.html`

### Purpose

Collect basic medical information that can help provide relevant healthcare services.

This section must be handled carefully because medical information is sensitive.

### 13.1 UI Sketch

```text
┌────────────────────────────────────────────────────────────┐
│                 Medical Information                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│ Blood Type                                                 │
│ [ Select Blood Type ▼ ]                                    │
│                                                            │
│ Previous Medical Conditions                                │
│ ┌────────────────────────────────────────────────────────┐ │
│ │                                                        │ │
│ │                                                        │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ Current Medications                                        │
│ ┌────────────────────────────────────────────────────────┐ │
│ │                                                        │ │
│ │                                                        │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ Allergies                                                  │
│ ┌────────────────────────────────────────────────────────┐ │
│ │                                                        │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ Additional Medical Notes                                  │
│ ┌────────────────────────────────────────────────────────┐ │
│ │                                                        │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ [ Back ]                                  [ Continue ]    │
└────────────────────────────────────────────────────────────┘
```

## 14. Medical Fields

| Field | Type | Required |
|---|---|---|
| Blood Type | Select | No |
| Previous Medical Conditions | Textarea | No |
| Current Medications | Textarea | No |
| Allergies | Textarea | No |
| Additional Medical Notes | Textarea | No |

Medical information may remain optional because a user may not have complete information during initial registration.

## 15. Medical Information UX

The frontend should explain why the information is being collected.

Example:

> **Medical Information**
>
> Providing your medical information can help healthcare providers understand your needs.
>
> You can update this information later from your profile.

Do not force users to enter medical information that is not necessary for the registration.

## 16. Sensitive Medical Data

The frontend must not:

- Log medical information to console
- Expose medical information in URLs
- Place medical information inside query strings
- Display medical information unnecessarily

The frontend should use controlled UI components for editing medical information.

## 17. Step 4 – Location

### Page

`frontend/pages/patient/registration/patient-location.html`

### Purpose

Collect the patient's address and location for nearby healthcare discovery and home services.

### 17.1 UI Sketch

```text
┌────────────────────────────────────────────────────────┐
│                 Location Information                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Governorate *                                          │
│ [ Select Governorate ▼ ]                               │
│                                                        │
│ City *                                                 │
│ [ Select City ▼ ]                                      │
│                                                        │
│ Detailed Address *                                     │
│ ┌────────────────────────────────────────────────────┐ │
│ │                                                    │ │
│ └────────────────────────────────────────────────────┘ │
│                                                        │
│ Current Location                                      │
│                                                        │
│ [ Use My Current Location ]                            │
│                                                        │
│ Latitude                                              │
│ [_____________________________________________]       │
│                                                        │
│ Longitude                                             │
│ [_____________________________________________]       │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
```

## 18. Location Fields

| Field | Type | Required |
|---|---|---|
| Governorate | Select | Yes |
| City | Select | Yes |
| Detailed Address | Textarea | Yes |
| Latitude | Number | Conditional |
| Longitude | Number | Conditional |
| Use Current Location | Button | No |

## 19. Location Workflow

```text
[ Use My Current Location ]
            ↓
      Browser Permission
            ↓
       Geolocation API
            ↓
      Latitude/Longitude
            ↓
         Form State
            ↓
        Backend API
```

The user must be informed why location is useful.

Example:

> Your location helps TechCare find healthcare providers near you.

## 20. Location Error States

Possible cases:

- Permission Granted
- Permission Denied
- Location Unavailable
- Timeout
- Browser Unsupported

Example:

```text
⚠ We could not access your current location.

You can enter your address manually.
```

## 21. Governorate / City Dependency

The city dropdown should depend on the selected governorate.

```text
Governorate
    ↓
Load Cities
    ↓
City
```

Example:

```text
Governorate
[ Dakahlia ▼ ]

City
[ Loading cities... ]

        ↓

City
[ Mansoura ▼ ]
```

## 22. Step 5 – Emergency Contact

### Page

`frontend/pages/patient/registration/patient-emergency-contact.html`

### Purpose

Allows the patient to provide a trusted contact who can be associated with emergency-related workflows.

This information is not intended to automatically trigger an emergency service during registration.

### 22.1 UI Sketch

```text
┌──────────────────────────────────────────────────────┐
│               Emergency Contact                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Emergency Contact Name                               │
│ [_______________________________________________]    │
│                                                      │
│ Relationship                                         │
│ [ Select Relationship ▼ ]                             │
│                                                      │
│ Phone Number                                         │
│ [_______________________________________________]    │
│                                                      │
│ Alternative Phone                                    │
│ [_______________________________________________]    │
│                                                      │
│ [ Back ]                           [ Continue ]      │
└──────────────────────────────────────────────────────┘
```

## 23. Emergency Contact Fields

| Field | Type | Required |
|---|---|---|
| Contact Name | Text | Business Rule |
| Relationship | Select | Business Rule |
| Phone Number | Tel | Business Rule |
| Alternative Phone | Tel | No |

The exact required status should follow the final product requirements.

## 24. Relationship Examples

- Parent
- Spouse
- Sibling
- Child
- Relative
- Friend
- Caregiver
- Other

## 25. Emergency Contact Validation

Examples:

- Contact Name → Valid name
- Phone → Valid phone format
- Alternative Phone → Valid phone when provided
- Relationship → Valid selection

## 26. Step 6 – Review

### Page

`frontend/pages/patient/registration/patient-review.html`

### UI Sketch

```text
┌─────────────────────────────────────────────────────────┐
│               Review Patient Registration               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ ACCOUNT                                                 │
│ Name: Mostafa Example                                   │
│ Email: patient@example.com                              │
│ Phone: 01XXXXXXXXX                                      │
│                                          [ Edit ]       │
│                                                         │
│ PERSONAL                                                │
│ Date of Birth: 01/01/2000                              │
│ Gender: Male                                            │
│                                          [ Edit ]       │
│                                                         │
│ MEDICAL                                                 │
│ Blood Type: O+                                          │
│ Medical Conditions: ***********                         │
│ Medications: ***********                                │
│                                          [ Edit ]       │
│                                                         │
│ LOCATION                                                │
│ Governorate: Dakahlia                                   │
│ City: Mansoura                                          │
│ Address: ************                                   │
│                                          [ Edit ]       │
│                                                         │
│ EMERGENCY CONTACT                                       │
│ Name: ************                                      │
│ Phone: ************                                     │
│                                          [ Edit ]       │
│                                                         │
│ [ ] I confirm that all information is correct.          │
│                                                         │
│ [ Back ]                      [ Create Patient Account ]│
└─────────────────────────────────────────────────────────┘
```

## 27. Review Page Rules

The user can edit each section:

```text
Review
  ↓
[ Edit Medical Information ]
  ↓
Medical Information
  ↓
Save
  ↓
Review
```

Information must remain populated when moving between steps.

## 28. Patient Registration Submission

```text
Review
   ↓
Confirm Information
   ↓
Validate
   ↓
Submit
   ↓
Create Patient Account
```

The submit button must be disabled while the request is processing.

## 29. Registration Success

### Page

`frontend/pages/patient/registration/patient-registration-success.html`

### UI Sketch

```text
┌───────────────────────────────────────────────────┐
│                                                   │
│                    ✓                              │
│                                                   │
│        Patient Registration Successful            │
│                                                   │
│ Your TechCare patient account has been created    │
│ successfully.                                     │
│                                                   │
│ Account Status                                    │
│ ACTIVE                                            │
│                                                   │
│                  [ Go to Login ]                  │
│                                                   │
└───────────────────────────────────────────────────┘
```

The exact account status comes from the backend.

## 30. Registration State

Conceptual structure:

```javascript
const patientRegistration = {
    account: {},
    personal: {},
    medical: {},
    location: {},
    emergencyContact: {}
};
```

Final payload:

```text
Account
+
Personal
+
Medical
+
Location
+
Emergency Contact
       ↓
Patient Registration API
```

## 31. Patient Validation Strategy

Use:

```text
Frontend Validation
        +
Backend Validation
```

Examples:

- Required field validation
- Email validation
- Phone validation
- Date validation
- Location validation
- Medical data format validation

## 32. Patient Error Handling

The frontend must handle:

- Validation Error
- Network Error
- Server Error
- Authentication Error
- Duplicate Account
- Location Error
- Session Expired
- Unexpected Error

Example:

```text
┌──────────────────────────────────────────┐
│ Unable to create your account.           │
│                                          │
│ Please review the highlighted fields    │
│ and try again.                           │
│                                          │
│             [ Try Again ]                │
└──────────────────────────────────────────┘
```

## 33. Medical Data Privacy

Patient medical information must be treated as highly sensitive data.

The frontend should avoid:

```javascript
console.log(patientMedicalData);
```

or URLs such as:

```text
/patient?condition=...
```

Medical information should be transferred through the appropriate authenticated API mechanisms.

## 34. Responsive Design

Patient registration must be optimized for:

- Desktop
- Laptop
- Tablet
- Mobile

Mobile example:

```text
┌────────────────────────────┐
│ Patient Registration       │
│                            │
│ Full Name                  │
│ [______________________]   │
│                            │
│ Email                      │
│ [______________________]   │
│                            │
│ Phone                      │
│ [______________________]   │
│                            │
│ [ Continue ]               │
└────────────────────────────┘
```

## 35. Accessibility

Patient registration should support:

- Semantic HTML
- Accessible Labels
- Keyboard Navigation
- Visible Focus
- Readable Error Messages
- Accessible Controls
- Mobile-Friendly Inputs

Because some TechCare users may have accessibility challenges, accessibility is especially important for this module.

## 36. API Service

### File

`frontend/js/services/patient.service.js`

### Conceptual structure

```javascript
const patientService = {

    createRegistration: async (payload) => {
        // Create patient account
    },

    submitRegistration: async (payload) => {
        // Submit patient registration
    },

    getCities: async (governorateId) => {
        // Load cities
    }

};
```

### Architecture

```text
Patient HTML
      ↓
Patient Registration JS
      ↓
Patient Service
      ↓
ASP.NET Core API
```

## 37. Patient Integration With Other Modules

After registration, the patient can later use:

- Search & Discovery
- Booking
- Doctor Services
- Nurse Services
- Pharmacy Search
- Laboratory Services
- Blood Donation
- Notifications
- Payments
- Ratings
- Complaints
- Medical Records

Registration only creates and initializes the patient profile.

## 38. Patient Registration Checklist

### Registration Pages

- [ ] Account Page
- [ ] Personal Information
- [ ] Medical Information
- [ ] Location
- [ ] Emergency Contact
- [ ] Review
- [ ] Success

### Functionality

- [ ] Progress Indicator
- [ ] Step Navigation
- [ ] Data Preservation
- [ ] Client-side Validation
- [ ] API Integration
- [ ] Error Handling
- [ ] Loading States
- [ ] Location Integration
- [ ] Responsive Design
- [ ] Accessibility
- [ ] Sensitive Data Protection
- [ ] Duplicate Submission Protection

## 39. Ownership Boundary

### Eman owns

`frontend/documentation/02-registration-forms/patient/`

### Implementation

```text
frontend/pages/patient/registration/
frontend/css/patient/patient-registration.css
frontend/js/pages/patient/patient-registration.js
frontend/js/services/patient.service.js
```

Shared Authentication remains with Mostafa.
