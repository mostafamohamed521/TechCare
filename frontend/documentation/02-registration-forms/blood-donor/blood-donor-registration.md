Part 2 — Blood Donor Registration

File: frontend/documentation/02-registration-forms/blood-donor/blood-donor-registration.md
Responsible: Eman
Module: Blood Donor Registration
Application: TechCare

40. Overview

The Blood Donor Registration module allows a user to register as a blood donor and provide the basic information required to participate in TechCare blood donation matching.

The donor profile focuses on:

Personal Information
+
Blood Type
+
Donation Eligibility Information
+
Location
+
Availability
+
Contact Preferences

The goal is to allow the platform to later identify suitable donors based on blood compatibility, location, and availability.

The registration process does not itself create a blood donation request or donation appointment.

41. Blood Donor Registration Flow
Register
   ↓
Select Blood Donor
   ↓
Account Information
   ↓
Shared OTP Verification
   ↓
Personal Information
   ↓
Blood & Donation Information
   ↓
Location
   ↓
Availability & Contact Preferences
   ↓
Review
   ↓
Submit
   ↓
Donor Profile Created
42. Frontend Structure
frontend/
│
├── pages/
│   └── blood-donor/
│       └── registration/
│           ├── blood-donor-register.html
│           ├── donor-personal-info.html
│           ├── donor-blood-info.html
│           ├── donor-location.html
│           ├── donor-availability.html
│           ├── donor-review.html
│           └── donor-registration-success.html
│
├── css/
│   └── blood-donor/
│       └── blood-donor-registration.css
│
├── js/
│   ├── pages/
│   │   └── blood-donor/
│   │       └── blood-donor-registration.js
│   │
│   └── services/
│       └── blood-donor.service.js
│
└── assets/
    └── blood-donor/
43. Registration Steps
Step	Page	File
1	Donor Account	blood-donor-register.html
2	Personal Information	donor-personal-info.html
3	Blood & Donation Information	donor-blood-info.html
4	Location	donor-location.html
5	Availability	donor-availability.html
6	Review	donor-review.html
7	Success	donor-registration-success.html
44. Step 1 – Donor Account
Page
frontend/pages/blood-donor/registration/blood-donor-register.html

The account section follows the same shared Authentication design.

┌──────────────────────────────────────────────┐
│        Create Blood Donor Account             │
├──────────────────────────────────────────────┤
│                                              │
│ Full Name *                                  │
│ [______________________________________]     │
│                                              │
│ Email Address *                              │
│ [______________________________________]     │
│                                              │
│ Phone Number *                               │
│ [______________________________________]     │
│                                              │
│ Password *                                   │
│ [________________________________ 👁 ]      │
│                                              │
│ Confirm Password *                           │
│ [________________________________ 👁 ]      │
│                                              │
│ [ ] Terms & Privacy Policy                   │
│                                              │
│                  [ Continue ]                │
└──────────────────────────────────────────────┘
45. Shared OTP
Donor Registration
       ↓
Account Created
       ↓
Shared OTP Verification
       ↓
Verified
       ↓
Continue

Documentation:

frontend/documentation/01-authentication/otp-verification.md
46. Step 2 – Personal Information
Page
frontend/pages/blood-donor/registration/donor-personal-info.html
UI Sketch
┌──────────────────────────────────────────────────┐
│              Donor Personal Information          │
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
47. Personal Fields
Field	Type	Required
First Name	Text	Yes
Middle Name	Text	No
Last Name	Text	Yes
Date of Birth	Date	Yes
Gender	Select/Radio	Yes
National ID	Text	Business Rule
Profile Picture	File	No
48. Step 3 – Blood & Donation Information
Page
frontend/pages/blood-donor/registration/donor-blood-info.html
UI Sketch
┌─────────────────────────────────────────────────────┐
│            Blood & Donation Information             │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Blood Type *                                        │
│ [ Select Blood Type ▼ ]                             │
│                                                     │
│ Rh Factor *                                         │
│ [ Select Rh Factor ▼ ]                              │
│                                                     │
│ Previous Donation Experience                        │
│ ( ) Yes     ( ) No                                  │
│                                                     │
│ Last Donation Date                                  │
│ [____ / ____ / ______]                              │
│                                                     │
│ Donation Availability                               │
│ (●) Available                                       │
│ ( ) Temporarily Unavailable                         │
│                                                     │
│ Additional Information                              │
│ ┌─────────────────────────────────────────────────┐ │
│ │                                                 │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ [ Back ]                           [ Continue ]     │
└─────────────────────────────────────────────────────┘
49. Blood Information Fields
Field	Type	Required
Blood Type	Select	Yes
Rh Factor	Select	Yes
Previous Donation Experience	Radio	No
Last Donation Date	Date	Conditional
Donation Availability	Radio/Select	Yes
Additional Information	Textarea	No
50. Blood Types

The UI should support standard blood groups:

A
B
AB
O

with Rh factor:

Positive (+)
Negative (-)

The final backend representation may use values such as:

A+
A-
B+
B-
AB+
AB-
O+
O-

The frontend must use the backend's final enum/value contract.

51. Previous Donation

When:

Previous Donation Experience = Yes

display:

Last Donation Date

When:

Previous Donation Experience = No

the field is not required.

Flow:

Previous Donation?
      │
      ├── No → Continue
      │
      └── Yes
             ↓
       Last Donation Date
52. Donation Availability

The donor can indicate whether they are currently available.

Example:

Donation Availability

(●) Available
( ) Temporarily Unavailable

This information will later help the blood donation matching workflow.

53. Eligibility Disclaimer

The frontend should clearly communicate that providing information during registration does not guarantee eligibility for donation.

Example:

Important

Your registration information helps TechCare identify potential
blood donors. Actual donation eligibility must be determined
according to the applicable medical and blood donation requirements.

The registration UI should not present itself as a medical eligibility diagnosis.

54. Step 4 – Donor Location
Page
frontend/pages/blood-donor/registration/donor-location.html
UI Sketch
┌────────────────────────────────────────────────────────┐
│                 Donor Location                         │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Governorate *                                          │
│ [ Select Governorate ▼ ]                               │
│                                                        │
│ City *                                                 │
│ [ Select City ▼ ]                                      │
│                                                        │
│ Area / Address *                                       │
│ [_______________________________________________]      │
│                                                        │
│ [ Use My Current Location ]                            │
│                                                        │
│ Latitude                                               │
│ [_______________________________________________]      │
│                                                        │
│ Longitude                                              │
│ [_______________________________________________]      │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
55. Location Purpose

Location is important because donor matching may consider:

Blood Compatibility
+
Geographic Proximity
+
Availability

Conceptually:

Blood Request
      ↓
Find Compatible Donors
      ↓
Filter by Location
      ↓
Check Availability
      ↓
Notify Suitable Donors
56. Location Privacy

The donor's exact location should not automatically be exposed to every patient.

The frontend should distinguish between:

Private Location Data

and

Public/Matching Location Information

For example, the patient might only see:

Approx. distance: 4.2 km
Area: Mansoura

rather than the donor's exact address.

The final privacy behavior must follow the backend authorization rules.

57. Step 5 – Availability
Page
frontend/pages/blood-donor/registration/donor-availability.html
UI Sketch
┌────────────────────────────────────────────────────────┐
│              Donation Availability                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Currently Available?                                   │
│ (●) Yes        ( ) No                                  │
│                                                        │
│ Preferred Donation Days                                │
│                                                        │
│ [ ] Sat   [ ] Sun   [ ] Mon   [ ] Tue                 │
│ [ ] Wed   [ ] Thu   [ ] Fri                           │
│                                                        │
│ Preferred Time                                         │
│ [ Morning ▼ ]                                          │
│                                                        │
│ Notification Preference                                │
│                                                        │
│ ☑ SMS                                                  │
│ ☑ Email                                                │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
58. Availability Fields
Field	Type	Required
Current Availability	Radio	Yes
Preferred Donation Days	Multi-select	No
Preferred Time	Select	No
Notification Preference	Checkbox	Business Rule

The availability model should remain simple.

59. Notification Preference

Possible options:

SMS
Email
Platform Notification

The final options depend on the available notification channels.

The donor should only receive notifications through channels they have agreed to where consent is required.

60. Step 6 – Review
Page
frontend/pages/blood-donor/registration/donor-review.html
UI Sketch
┌──────────────────────────────────────────────────────────┐
│             Review Blood Donor Registration               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ACCOUNT                                                  │
│ Name: **************                                     │
│ Email: **************                                    │
│ Phone: **************                                    │
│                                          [ Edit ]        │
│                                                          │
│ PERSONAL                                                 │
│ Date of Birth: ********                                 │
│ Gender: Male                                             │
│                                          [ Edit ]        │
│                                                          │
│ BLOOD INFORMATION                                        │
│ Blood Type: O+                                           │
│ Previous Donation: Yes                                   │
│                                          [ Edit ]        │
│                                                          │
│ LOCATION                                                 │
│ Governorate: Dakahlia                                    │
│ City: Mansoura                                           │
│                                          [ Edit ]        │
│                                                          │
│ AVAILABILITY                                             │
│ Status: Available                                        │
│ Days: Sat, Sun, Tue                                      │
│                                          [ Edit ]        │
│                                                          │
│ [ ] I confirm that the provided information is correct.  │
│                                                          │
│ [ Back ]                [ Submit Donor Registration ]    │
└──────────────────────────────────────────────────────────┘
61. Sensitive Information on Review

Do not unnecessarily expose:

National ID
Exact Address
Precise Coordinates

Example:

National ID:
298***********

Location:
Mansoura, Dakahlia
62. Final Submission
Review
  ↓
Confirm
  ↓
Validate
  ↓
Submit
  ↓
Create Donor Profile

Button states:

[ Submit Donor Registration ]

        ↓

[ ⏳ Submitting... ]

The button must be disabled during submission.

63. Donor Registration Success
Page
frontend/pages/blood-donor/registration/donor-registration-success.html
UI Sketch
┌───────────────────────────────────────────────────┐
│                                                   │
│                    ✓                              │
│                                                   │
│       Blood Donor Registration Complete           │
│                                                   │
│ Your donor profile has been created successfully. │
│                                                   │
│ Donor Status                                      │
│ AVAILABLE                                         │
│                                                   │
│ You may receive notifications when a compatible   │
│ blood donation need is available.                 │
│                                                   │
│                  [ Go to Login ]                  │
│                                                   │
└───────────────────────────────────────────────────┘

The actual availability/status must come from backend state.

64. Donor State

Conceptual registration object:

const bloodDonorRegistration = {
    account: {},
    personal: {},
    bloodInformation: {},
    location: {},
    availability: {}
};

Final:

Account
+
Personal
+
Blood Information
+
Location
+
Availability
       ↓
Blood Donor API
65. Validation Strategy

Frontend:

Required Fields
Field Format
Date Validation
Blood Type Selection
Availability Validation
Location Validation

Backend:

Authoritative Validation
Authorization
Duplicate Detection
Donor Status
Matching Eligibility Rules

The frontend must not decide medical eligibility by itself.

66. Important Date Validation

When the user indicates a previous donation:

Last Donation Date

must not be a future date.

Example:

⚠ Last donation date cannot be in the future.

Any medical donation interval/eligibility rule should be enforced by the backend/business logic rather than hardcoded as a frontend medical decision.

67. Error Handling

The module must handle:

Validation Error
Network Error
Server Error
Location Error
Duplicate Account
Session Expired
Unexpected Error

Example:

┌──────────────────────────────────────────┐
│ Unable to complete donor registration.   │
│                                          │
│ Please review the form and try again.    │
│                                          │
│              [ Try Again ]               │
└──────────────────────────────────────────┘
68. Location Integration
[ Use My Current Location ]
            ↓
      Browser Permission
            ↓
       Geolocation API
            ↓
      Latitude/Longitude
            ↓
        Donor Profile

Manual location entry should remain available where permitted.

69. Responsive Design

Blood Donor Registration must support:

Desktop
Laptop
Tablet
Mobile

Example mobile:

┌────────────────────────────┐
│ Donor Registration         │
│                            │
│ Blood Type                 │
│ [ O+ ▼ ]                   │
│                            │
│ Availability              │
│ (●) Available              │
│                            │
│ [ Continue ]               │
└────────────────────────────┘
70. Accessibility

The module must support:

Semantic HTML
Accessible Labels
Keyboard Navigation
Visible Focus
Readable Validation
Accessible Radio Buttons
Accessible Checkboxes
Accessible Selects
71. API Service

File:

frontend/js/services/blood-donor.service.js

Conceptual structure:

const bloodDonorService = {

    createRegistration: async (payload) => {
        // Create donor registration
    },

    submitRegistration: async (payload) => {
        // Submit donor profile
    },

    getCities: async (governorateId) => {
        // Load cities
    }

};

Architecture:

Blood Donor HTML
        ↓
Registration JavaScript
        ↓
Blood Donor Service
        ↓
ASP.NET Core API
72. Blood Donor Integration With Matching

After registration:

Donor Profile
      ↓
Blood Type
      ↓
Location
      ↓
Availability
      ↓
Blood Donation Matching

Later, when a blood request exists:

Blood Request
      ↓
Compatible Blood Type
      ↓
Nearby Donors
      ↓
Available Donors
      ↓
Notifications

The registration module only collects the required data.

73. Donor Privacy

The donor should have control over the information presented to other users.

The frontend should not expose:

Exact Home Address
Precise Coordinates
National ID
Private Contact Information

unless the relevant authorized workflow explicitly requires it.

74. Complete Blood Donor Flow
                    ┌───────────────┐
                    │ Register      │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Blood Donor   │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Account       │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ OTP           │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Personal      │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Blood Info    │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Location      │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Availability  │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Review        │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Submit        │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Success       │
                    └───────────────┘
75. Blood Donor Checklist
[ ] Account Page
[ ] Personal Information
[ ] Blood Information
[ ] Previous Donation Section
[ ] Location
[ ] Availability
[ ] Notification Preferences
[ ] Review
[ ] Success

[ ] Progress Indicator
[ ] Step Navigation
[ ] Form State
[ ] Validation
[ ] Error Handling
[ ] Loading States
[ ] Location Integration
[ ] Responsive Design
[ ] Accessibility
[ ] Privacy Protection
[ ] Duplicate Submission Protection
[ ] API Integration
76. Eman's Ownership

Eman owns the following documentation:

frontend/documentation/02-registration-forms/patient/patient-registration.md

frontend/documentation/02-registration-forms/blood-donor/blood-donor-registration.md

And the corresponding implementation areas:

frontend/pages/patient/registration/
frontend/css/patient/
frontend/js/pages/patient/
frontend/js/services/patient.service.js

and:

frontend/pages/blood-donor/registration/
frontend/css/blood-donor/
frontend/js/pages/blood-donor/
frontend/js/services/blood-donor.service.js
77. Integration With Shared Authentication

Both modules use:

frontend/documentation/01-authentication/

for:

Account Creation
Role Selection
OTP Verification
Login
Password Recovery
Token Management

There should be one centralized Authentication implementation, not separate login/OTP implementations for Patient and Blood Donor.

78. Eman Module Relationship
                         Authentication
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
               Patient             Blood Donor
                    │                   │
                    ↓                   ↓
             Patient Profile      Donor Profile
                    │                   │
          ┌─────────┼─────────┐         │
          ↓         ↓         ↓         ↓
       Booking   Search     Records   Blood Matching