Nurse Registration – Frontend Specification

File: frontend/documentation/02-registration-forms/nurse/nurse-registration.md
Responsible: Ahmed
Module: Nurse Registration
Application: TechCare

1. Overview

The Nurse Registration module allows a nurse to create a professional TechCare account and submit the required personal, professional, service, location, availability, and verification information.

The registration process is divided into multiple frontend steps to make the form easier to complete, especially for users who may be less comfortable with large forms.

The frontend must collect the required information, validate user input, display clear errors, preserve entered data between steps, and finally submit the complete registration request to the backend.

Registration does not immediately activate the nurse as a service provider.

After successful submission:

Nurse Registration
        ↓
Registration Submitted
        ↓
Pending Verification
        ↓
Admin Review
        ↓
Approved / Rejected / Additional Information Required

The Nurse Dashboard must not be accessible as an active provider until the account and professional verification requirements are satisfied.

2. Registration Responsibility

Ahmed is responsible for the frontend implementation and documentation of the Nurse Registration module.

Ahmed's frontend responsibilities
Nurse Registration UI
        │
        ├── Registration Pages
        ├── Form Components
        ├── Client-side Validation
        ├── Registration Navigation
        ├── File Upload UI
        ├── Review & Submit UI
        ├── Registration Success UI
        ├── Error Handling
        ├── Loading States
        └── API Integration
Shared functionality

The following functionality belongs to the shared Authentication module and must not be duplicated inside Nurse Registration:

Login
Register Entry
Role Selection
OTP Verification
Email/Phone Verification
Password Recovery
Logout
Authentication Token Handling

Therefore the Nurse Registration flow integrates with Authentication but does not recreate it.

3. Frontend File Structure

Recommended frontend implementation:

frontend/
│
├── pages/
│   └── nurse/
│       └── registration/
│           ├── nurse-register.html
│           ├── nurse-personal-info.html
│           ├── nurse-professional-info.html
│           ├── nurse-service-info.html
│           ├── nurse-location-availability.html
│           ├── nurse-documents.html
│           ├── nurse-review.html
│           └── nurse-registration-success.html
│
├── css/
│   └── nurse/
│       └── nurse-registration.css
│
├── js/
│   └── pages/
│       └── nurse/
│           └── nurse-registration.js
│
├── js/
│   └── services/
│       └── nurse.service.js
│
└── assets/
    └── nurse/

The exact technical organization may evolve, but all Nurse Registration frontend code must remain clearly separated from unrelated role modules.

4. Registration Flow

The complete frontend flow is:

                    ┌─────────────────────┐
                    │      Register       │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │    Select Nurse     │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Nurse Account Info  │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Shared Verification │
                    │   OTP / Email /     │
                    │       Phone         │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Personal Information│
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │Professional Profile │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Service & Pricing   │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │Location & Availability│
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │      Documents      │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Review & Confirmation│
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │    Submit Request   │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Registration Success│
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Pending Verification│
                    └─────────────────────┘
5. Registration Page Sequence

The frontend contains the following pages:

Step	Page	File
1	Nurse Account Registration	nurse-register.html
2	Personal Information	nurse-personal-info.html
3	Professional Information	nurse-professional-info.html
4	Service & Pricing	nurse-service-info.html
5	Location & Availability	nurse-location-availability.html
6	Documents	nurse-documents.html
7	Review	nurse-review.html
8	Success	nurse-registration-success.html

OTP verification is handled by the shared Authentication module.

6. Registration Progress Indicator

Because the registration contains several steps, the frontend should display the current progress.

Example:

┌─────────────────────────────────────────────────────────────┐
│ Nurse Registration                                          │
│                                                             │
│ ● Account ─── ● Personal ─── ○ Professional ─── ○ Service │
│                                                             │
│ ○ Location ─── ○ Documents ─── ○ Review                    │
│                                                             │
│ Step 2 of 7                                                 │
└─────────────────────────────────────────────────────────────┘

The indicator should clearly distinguish:

Completed Step
●

Current Step
◉

Upcoming Step
○

The user must know exactly where they are in the registration process.

7. Step 1 – Nurse Account Registration
Page
frontend/pages/nurse/registration/nurse-register.html
Purpose

This page collects the nurse's basic account credentials.

UI Sketch
┌───────────────────────────────────────────────┐
│             Create Nurse Account              │
├───────────────────────────────────────────────┤
│                                               │
│ Full Name                                     │
│ [_________________________________________]   │
│                                               │
│ Email Address                                 │
│ [_________________________________________]   │
│                                               │
│ Phone Number                                  │
│ [_________________________________________]   │
│                                               │
│ Password                                      │
│ [____________________________________ 👁 ]    │
│                                               │
│ Confirm Password                              │
│ [____________________________________ 👁 ]    │
│                                               │
│ [ ] I agree to the Terms & Privacy Policy     │
│                                               │
│                    [ Continue ]               │
│                                               │
└───────────────────────────────────────────────┘
7.1 Fields
Field	Type	Required
Full Name	Text	Yes
Email Address	Email	Yes
Phone Number	Tel	Yes
Password	Password	Yes
Confirm Password	Password	Yes
Terms & Privacy Agreement	Checkbox	Yes
7.2 Validation
Full Name

Must:

Be required.
Contain a valid name.
Not contain meaningless characters.
Respect configured minimum and maximum length.
Email

Must:

Be required.
Have a valid email structure.
Be normalized before submission.
Be checked by backend for uniqueness.
Phone

Must:

Be required.
Follow the application's accepted phone format.
Be validated on frontend.
Be verified through the shared Authentication process.
Password

The password must satisfy the project's password policy.

The UI should visually communicate password requirements.

Example:

Password requirements:

✓ At least 8 characters
✓ Uppercase letter
✓ Lowercase letter
✓ Number
✓ Special character

The exact password policy must remain consistent with the centralized Authentication module.

Confirm Password

Must match the password exactly.

Example:

Password:        ************
Confirm Password:************

Passwords match ✓
8. Shared OTP Verification

After creating the account, the user follows the shared Authentication verification process.

Nurse Registration
       ↓
Account Created
       ↓
OTP Verification
       ↓
Verified
       ↓
Continue Nurse Registration

The Nurse module must not implement a separate OTP system.

The frontend should redirect to the existing Authentication OTP page.

Example:

frontend/documentation/01-authentication/otp-verification.md

After successful verification:

OTP Verified
     ↓
Return to Nurse Registration
     ↓
Personal Information
9. Step 2 – Personal Information
Page
frontend/pages/nurse/registration/nurse-personal-info.html
Purpose

Collect the nurse's personal identity and basic profile information.

9.1 UI Sketch
┌──────────────────────────────────────────────────┐
│ Personal Information                             │
├──────────────────────────────────────────────────┤
│                                                  │
│ First Name                                       │
│ [__________________________________________]     │
│                                                  │
│ Middle Name (Optional)                           │
│ [__________________________________________]     │
│                                                  │
│ Last Name                                        │
│ [__________________________________________]     │
│                                                  │
│ Date of Birth                                    │
│ [____ / ____ / ______]                           │
│                                                  │
│ Gender                                           │
│ [ Select Gender ▼ ]                              │
│                                                  │
│ National ID Number                               │
│ [__________________________________________]     │
│                                                  │
│ Profile Picture                                  │
│ [ Choose File ]                                  │
│                                                  │
│ Governorate                                      │
│ [ Select Governorate ▼ ]                         │
│                                                  │
│ City                                             │
│ [ Select City ▼ ]                                │
│                                                  │
│ Detailed Address                                 │
│ [__________________________________________]     │
│ [__________________________________________]     │
│                                                  │
│ [ Back ]                         [ Continue ]    │
└──────────────────────────────────────────────────┘
9.2 Fields
Field	Type	Required
First Name	Text	Yes
Middle Name	Text	No
Last Name	Text	Yes
Date of Birth	Date	Yes
Gender	Select/Radio	Yes
National ID Number	Text	Yes
Profile Picture	File	No
Governorate	Select	Yes
City	Select	Yes
Detailed Address	Textarea	Yes
10. Step 3 – Professional Information
Page
frontend/pages/nurse/registration/nurse-professional-info.html
Purpose

This page collects the nurse's professional information.

10.1 UI Sketch
┌──────────────────────────────────────────────────┐
│ Professional Information                         │
├──────────────────────────────────────────────────┤
│                                                  │
│ Professional Title                               │
│ [__________________________________________]     │
│                                                  │
│ Nursing Qualification                            │
│ [ Select Qualification ▼ ]                       │
│                                                  │
│ Professional License Number                      │
│ [__________________________________________]     │
│                                                  │
│ Years of Experience                              │
│ [_______________]                                │
│                                                  │
│ Professional Skills                              │
│                                                  │
│ [ ] Patient Monitoring                           │
│ [ ] Wound Care                                   │
│ [ ] Elderly Care                                 │
│ [ ] Post-operative Care                          │
│ [ ] Personal Care Assistance                     │
│ [ ] Medication Assistance                        │
│                                                  │
│ Services Offered                                 │
│                                                  │
│ [ ] Home Nursing                                 │
│ [ ] Elderly Care                                 │
│ [ ] Wound Care                                   │
│ [ ] Patient Monitoring                           │
│                                                  │
│ Professional Bio                                 │
│ ┌──────────────────────────────────────────────┐ │
│ │                                              │ │
│ │                                              │ │
│ └──────────────────────────────────────────────┘ │
│                                                  │
│ [ Back ]                         [ Continue ]    │
└──────────────────────────────────────────────────┘
10.2 Fields
Field	Type	Required
Professional Title	Text/Select	Yes
Nursing Qualification	Select	Yes
Professional License Number	Text	Yes
Years of Experience	Number	Yes
Skills	Multi-select/Checkbox	Yes
Services Offered	Multi-select/Checkbox	Yes
Professional Bio	Textarea	Yes
10.3 Example Nursing Qualifications

The frontend may display predefined options such as:

Nursing Diploma
Bachelor of Nursing
Technical Nursing Qualification
Other

The backend remains responsible for final validation.

11. Step 4 – Service & Pricing
Page
frontend/pages/nurse/registration/nurse-service-info.html
Purpose

Allows the nurse to specify the type of service provided and the pricing model.

11.1 UI Sketch
┌──────────────────────────────────────────────────┐
│ Service & Pricing                                │
├──────────────────────────────────────────────────┤
│                                                  │
│ Service Type                                     │
│ [ Home Nursing ▼ ]                               │
│                                                  │
│ Base Service Price                               │
│ [_________________] EGP                          │
│                                                  │
│ Price Per Kilometer                              │
│ [_________________] EGP / KM                    │
│                                                  │
│ Service Radius                                   │
│ [_________________] KM                           │
│                                                  │
│ Visit Duration                                   │
│ [__________] Minutes                             │
│                                                  │
│ Additional Service Fees (Optional)               │
│ [_________________] EGP                          │
│                                                  │
│ Currency                                         │
│ [ EGP ▼ ]                                        │
│                                                  │
│ Pricing Preview                                  │
│ ┌──────────────────────────────────────────────┐ │
│ │ Base Price:              500 EGP             │ │
│ │ Travel Rate:               5 EGP / KM        │ │
│ │ Service Radius:           20 KM              │ │
│ │ Visit Duration:           60 Minutes         │ │
│ └──────────────────────────────────────────────┘ │
│                                                  │
│ [ Back ]                         [ Continue ]    │
└──────────────────────────────────────────────────┘

The displayed values above are examples only and must not be hardcoded as default business values unless approved by the backend/business rules.

11.2 Fields
Field	Type	Required
Service Type	Select	Yes
Base Service Price	Number	Yes
Price Per Kilometer	Number	Yes
Service Radius	Number	Yes
Visit Duration	Number	Yes
Additional Service Fees	Number	No
Currency	Select	Yes
12. Pricing Preview

The frontend may provide a preview to help the nurse understand the pricing model.

Example:

Base Price
500 EGP

+

Distance Fee
5 EGP × Distance

+

Additional Fee
Optional

=

Estimated Total

The displayed calculation is only a UI preview.

The final price must always be calculated and validated by the backend.

The frontend must never be trusted as the source of truth for financial calculations.

13. Step 5 – Location & Availability
Page
frontend/pages/nurse/registration/nurse-location-availability.html
Purpose

Collect the nurse's service location, home visit coverage, and working availability.

13.1 UI Sketch
┌────────────────────────────────────────────────────────┐
│ Location & Availability                                │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Governorate                                            │
│ [ Select Governorate ▼ ]                               │
│                                                        │
│ City                                                   │
│ [ Select City ▼ ]                                      │
│                                                        │
│ Detailed Address                                       │
│ [_______________________________________________]      │
│                                                        │
│ Location                                               │
│                                                        │
│ [ Use Current Location ]                               │
│                                                        │
│ Latitude                                               │
│ [________________________]                             │
│                                                        │
│ Longitude                                              │
│ [________________________]                             │
│                                                        │
│ Available for Home Visits                              │
│ (●) Yes    ( ) No                                      │
│                                                        │
│ Working Days                                           │
│                                                        │
│ [ ] Sat  [ ] Sun  [ ] Mon  [ ] Tue                    │
│ [ ] Wed  [ ] Thu  [ ] Fri                             │
│                                                        │
│ Start Time                                             │
│ [________]                                             │
│                                                        │
│ End Time                                               │
│ [________]                                             │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
14. Location Fields
Field	Type	Required
Governorate	Select	Yes
City	Select	Yes
Detailed Address	Textarea	Yes
Latitude	Number	Conditional
Longitude	Number	Conditional
Use Current Location	Button	No
Available for Home Visits	Radio	Yes

Latitude and longitude are required when location-based service functionality depends on precise coordinates.

15. Location Interaction

The frontend may provide:

[ Use Current Location ]

After permission is granted:

Browser
   ↓
Geolocation API
   ↓
Latitude + Longitude
   ↓
Frontend Form State
   ↓
Backend

The UI must handle:

Permission Granted
Permission Denied
Location Unavailable
Location Timeout
Browser Does Not Support Geolocation

Example error:

Unable to access your current location.
Please enter your address manually.

The user must always be able to continue with manual location information where the business rules allow it.

16. Availability

The nurse must specify available working days and hours.

Example:

Working Days:

☑ Saturday
☑ Sunday
☐ Monday
☑ Tuesday
☑ Wednesday
☐ Thursday
☐ Friday

Start Time:
09:00 AM

End Time:
05:00 PM

Validation:

At least one working day
        +
Valid start time
        +
Valid end time
        +
Start Time < End Time

Invalid example:

Start: 06:00 PM
End:   09:00 AM

The frontend must display a clear validation message.

17. Step 6 – Documents
Page
frontend/pages/nurse/registration/nurse-documents.html
Purpose

Collect documents required for professional verification.

17.1 UI Sketch
┌───────────────────────────────────────────────────────┐
│ Professional Documents                                │
├───────────────────────────────────────────────────────┤
│                                                       │
│ National ID - Front                                   │
│ [ Choose File ]                                       │
│                                                       │
│ National ID - Back                                    │
│ [ Choose File ]                                       │
│                                                       │
│ Nursing Certificate                                   │
│ [ Choose File ]                                       │
│                                                       │
│ Professional / Practice License                       │
│ [ Choose File ]                                       │
│                                                       │
│ Additional Supporting Document                        │
│ [ Choose File ]                                       │
│                                                       │
│ Supported: PDF, JPG, JPEG, PNG                        │
│                                                       │
│ ┌───────────────────────────────────────────────────┐ │
│ │ ✓ document.pdf                                    │ │
│ │   Upload completed                                │ │
│ └───────────────────────────────────────────────────┘ │
│                                                       │
│ [ Back ]                              [ Continue ]    │
└───────────────────────────────────────────────────────┘
18. Required Documents
Document	Required
National ID Front	Yes
National ID Back	Yes
Nursing Certificate	Yes
Professional / Practice License	Yes
Additional Supporting Document	No

The exact accepted document types and maximum sizes must match the backend/API rules.

19. File Upload UX

For every uploaded file, the frontend should display:

Filename
File Size
File Type
Upload Status
Remove Button

Example:

┌────────────────────────────────────────┐
│ nursing_certificate.pdf                │
│ 2.4 MB                                 │
│                                        │
│ Uploading... 65%                       │
│ █████████████░░░░░░░                   │
└────────────────────────────────────────┘

After success:

✓ Uploaded successfully

After failure:

✕ Upload failed

Invalid file type.

The user should be able to remove and replace a selected file before final submission.

20. Step 7 – Review & Submit
Page
frontend/pages/nurse/registration/nurse-review.html
Purpose

Displays all collected information before final submission.

The user must be able to review the information and return to individual steps to edit mistakes.

20.1 UI Sketch
┌────────────────────────────────────────────────────────┐
│ Review Nurse Registration                              │
├────────────────────────────────────────────────────────┤
│                                                        │
│ ACCOUNT                                                │
│ Full Name: Mostafa Example                             │
│ Email: nurse@example.com                               │
│ Phone: 01XXXXXXXXX                                     │
│                                      [ Edit ]          │
│                                                        │
│ PERSONAL INFORMATION                                   │
│ Date of Birth: 01/01/1995                             │
│ Gender: Female                                         │
│ National ID: ***************                           │
│                                      [ Edit ]          │
│                                                        │
│ PROFESSIONAL INFORMATION                               │
│ Qualification: Bachelor of Nursing                     │
│ Experience: 5 Years                                    │
│ License: ***********                                   │
│                                      [ Edit ]          │
│                                                        │
│ SERVICES                                               │
│ Home Nursing                                           │
│ Elderly Care                                           │
│ Wound Care                                             │
│                                      [ Edit ]          │
│                                                        │
│ LOCATION & AVAILABILITY                                │
│ Governorate: Dakahlia                                  │
│ City: Mansoura                                         │
│ Home Visits: Yes                                       │
│                                      [ Edit ]          │
│                                                        │
│ DOCUMENTS                                              │
│ ✓ National ID Front                                    │
│ ✓ National ID Back                                     │
│ ✓ Nursing Certificate                                  │
│ ✓ Professional License                                 │
│                                      [ Edit ]          │
│                                                        │
│ [ ] I confirm that all information is correct.         │
│                                                        │
│ [ Back ]                    [ Submit Registration ]    │
└────────────────────────────────────────────────────────┘
21. Sensitive Information on Review Page

Sensitive information such as:

National ID
Professional License
Uploaded Documents

should not unnecessarily be displayed in full.

Example:

National ID:
298***********

License:
LIC-****-2381

The frontend should show only the minimum information necessary for confirmation.

22. Final Submission

The final action is:

Submit Registration

Before submission, the frontend must verify that:

All required steps completed
        +
Required fields valid
        +
Required documents uploaded
        +
Confirmation accepted
        ↓
Allow Submission

After clicking Submit:

Button disabled
       ↓
Loading indicator
       ↓
API Request
       ↓
Success / Error
23. Submit Button States

The submit button must have multiple states.

Normal
[ Submit Registration ]
Loading
[ ⏳ Submitting... ]
Disabled
[ Submit Registration ]

Button becomes disabled during submission to prevent duplicate requests.

24. Step Navigation

Each page should contain:

[ Back ]                    [ Continue ]

For the first step:

[ Cancel ]                  [ Continue ]

For the review step:

[ Back ]              [ Submit Registration ]

The frontend should not lose previously entered data when navigating between registration steps.

25. Form State Management

The registration data can be maintained as one registration state object.

Example conceptual structure:

const nurseRegistration = {
    account: {},
    personal: {},
    professional: {},
    services: {},
    location: {},
    availability: {},
    documents: {}
};

Each page updates only its corresponding section.

Example:

personal page
      ↓
personal state

professional page
      ↓
professional state

service page
      ↓
service state

At final submission:

All sections
     ↓
Registration Payload
     ↓
API
26. Validation Strategy

Frontend validation improves user experience, but it is not the final security boundary.

The system must use:

Frontend Validation
        +
Backend Validation

Never rely only on JavaScript validation.

27. Common Validation Messages

The frontend should use clear messages.

Bad:

Invalid input.

Better:

Please enter your professional license number.

Bad:

Error 400.

Better:

We could not submit your registration.
Please review the highlighted fields and try again.
28. Required Field Validation

Required fields should clearly indicate their status.

Example:

Full Name *
[____________________________]

When empty:

Full Name *
[____________________________]

⚠ Full Name is required.

Validation should preferably happen:

On blur

and again:

Before Continue

and finally:

Before Submit
29. Professional Validation

Examples:

Years of Experience
Must be a non-negative number.

Invalid:

-5
Service Price
Must not be negative.
Price Per Kilometer
Must not be negative.
Service Radius
Must be greater than zero.
Visit Duration
Must be a valid positive duration.
30. Document Validation

Before upload, the frontend should check:

File Selected
      ↓
File Extension
      ↓
File Type
      ↓
File Size
      ↓
Upload

Example accepted types:

PDF
JPG
JPEG
PNG

The exact accepted extensions and limits must be synchronized with backend configuration.

31. Backend Validation Errors

Backend validation errors should be mapped to their related fields when possible.

Example API response:

professionalLicenseNumber:
"This license number is already registered."

Frontend:

Professional License Number
[LIC-123456________________]

⚠ This license number is already registered.

This is better than displaying a generic page-level error only.

32. Registration Error States

The frontend must handle:

Validation Error
Authentication Error
OTP Verification Error
Network Error
Server Error
File Upload Error
Duplicate Data Error
Session Expiration
Unexpected Error

Example network error:

┌───────────────────────────────────────────┐
│ Unable to connect to TechCare.             │
│ Please check your internet connection     │
│ and try again.                            │
│                                           │
│             [ Try Again ]                 │
└───────────────────────────────────────────┘
33. Registration Success Page
Page
frontend/pages/nurse/registration/nurse-registration-success.html
33.1 UI Sketch
┌──────────────────────────────────────────────────┐
│                                                  │
│                    ✓                             │
│                                                  │
│        Registration Submitted Successfully       │
│                                                  │
│ Your nurse registration has been submitted      │
│ successfully and is currently under review.      │
│                                                  │
│ Registration ID                                  │
│ NC-2026-000123                                  │
│                                                  │
│ Status                                           │
│ PENDING_VERIFICATION                            │
│                                                  │
│ We will notify you once the verification         │
│ process is completed.                            │
│                                                  │
│                [ Go to Login ]                  │
│                                                  │
└──────────────────────────────────────────────────┘

The Registration ID is illustrative.

34. Important Post-Registration Rule

After registration submission:

Success
  ↓
Pending Verification
  ↓
Login

The nurse must not automatically enter the active Nurse Dashboard unless the backend confirms that the account has the appropriate verified/provider status.

This prevents unverified providers from accessing protected provider capabilities.

35. Pending Verification State

The frontend should represent the pending status clearly.

Example:

┌─────────────────────────────────────────────┐
│ Verification Status                         │
│                                             │
│             ⏳ Pending                      │
│                                             │
│ Your professional profile is being reviewed.│
│                                             │
│ Status: PENDING_VERIFICATION                │
└─────────────────────────────────────────────┘

Possible statuses:

PENDING_VERIFICATION
APPROVED
REJECTED
ADDITIONAL_INFORMATION_REQUIRED
SUSPENDED

The final status values must match the backend contract.

36. Responsive Design

The Nurse Registration interface must work on:

Desktop
Laptop
Tablet
Mobile

Desktop:

┌─────────────────────────────────────────────┐
│              Registration Form              │
│                                             │
│   Field                    Field            │
│   [__________]             [__________]    │
│                                             │
└─────────────────────────────────────────────┘

Mobile:

┌───────────────────────┐
│ Registration          │
│                       │
│ Full Name             │
│ [_________________]   │
│                       │
│ Email                 │
│ [_________________]   │
│                       │
│ Phone                 │
│ [_________________]   │
│                       │
│ [ Continue ]          │
└───────────────────────┘

Forms should never require horizontal scrolling on normal mobile screen sizes.

37. Accessibility

The registration forms must support accessible interaction.

Every input should have an associated label.

Example:

<label for="licenseNumber">
    Professional License Number
</label>

<input
    id="licenseNumber"
    name="licenseNumber"
    type="text"
>

The frontend should also support:

Keyboard Navigation
Visible Focus States
Readable Error Messages
Accessible Labels
Adequate Input Contrast
Button Accessibility
File Input Accessibility
38. Input Design Standards

Inputs should consistently use:

Label
Input
Helper Text
Error Message

Example:

Professional License Number *

[____________________________]

Enter your valid professional license number.

⚠ This license number is already registered.
39. Loading States

Every asynchronous operation should have an appropriate loading state.

Examples:

Loading cities
City
[ Loading cities... ]
Uploading document
Uploading...
████████████░░░░
Submitting
Submitting registration...

The interface should prevent confusing duplicate actions while requests are in progress.

40. API Integration

The Nurse Registration frontend should have a dedicated service layer:

frontend/js/services/nurse.service.js

The service layer should be responsible for communicating with backend endpoints.

Conceptually:

const nurseService = {
    register: async (payload) => {},
    uploadDocument: async (file) => {},
    submitRegistration: async (payload) => {}
};

The UI page should not contain all API implementation logic directly.

Recommended flow:

HTML
 ↓
Page JavaScript
 ↓
Nurse Service
 ↓
HTTP Request
 ↓
ASP.NET Core API
41. API Security Rules

The frontend must not assume that hidden fields are secure.

For example:

<input type="hidden" value="approved">

does not provide security.

The backend must determine:

Role
Verification Status
Permissions
Provider Status
Financial Values

The frontend only displays the current backend-authoritative state.

42. Duplicate Submission Protection

The frontend must prevent:

Double Click
      ↓
Two Registration Requests

Use:

Submit button disabled
      +
Loading state
      +
Server-side duplicate protection
43. Unsaved Changes

When appropriate, the frontend should warn the user before leaving a partially completed registration.

Example:

┌──────────────────────────────────────────────┐
│ Leave registration?                          │
│                                              │
│ You have unsaved information.                │
│                                              │
│ [ Stay ]                    [ Leave ]        │
└──────────────────────────────────────────────┘

This is especially useful for multi-step registration.

44. Registration Data Security

The frontend must avoid unnecessarily exposing sensitive information.

Sensitive values include:

National ID
Professional License
Uploaded Verification Documents
Personal Address
Location Coordinates

The frontend must not:

Expose sensitive data in URL parameters
Log sensitive values to console
Display full document contents unnecessarily
Store sensitive information insecurely
45. User Experience Rules

The registration should follow these principles:

Simple
Clear
Consistent
Accessible
Mobile Friendly
Error Resistant
Easy to Navigate

Users should always know:

Where am I?
What information is required?
What is invalid?
What happens next?
Can I go back?
Is my data saved?
46. Complete Frontend Flow

The final complete Nurse Registration experience is:

┌───────────────────────┐
│ Register              │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Select Nurse          │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Account Information   │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ OTP Verification      │
│ (Shared Auth Module)  │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Personal Information  │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Professional Info     │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Service & Pricing     │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Location & Availability│
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Documents             │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Review                │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Submit Registration   │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Success               │
│ Pending Verification  │
└───────────────────────┘
47. Final Nurse Registration Checklist

Ahmed's Nurse Registration frontend is considered complete when the following are implemented:

[ ] Nurse account form
[ ] Personal information form
[ ] Professional information form
[ ] Service information form
[ ] Pricing form
[ ] Location form
[ ] Availability form
[ ] Document upload form
[ ] Review page
[ ] Success page
[ ] Registration progress indicator
[ ] Step navigation
[ ] Client-side validation
[ ] Error states
[ ] Loading states
[ ] File validation
[ ] File upload UI
[ ] Responsive layout
[ ] Accessibility support
[ ] API service integration
[ ] Duplicate submission protection
[ ] Pending verification state
[ ] Secure handling of sensitive data
48. Ownership Boundary

Ahmed owns:

Nurse Registration

Specifically:

frontend/pages/nurse/registration/
frontend/css/nurse/nurse-registration.css
frontend/js/pages/nurse/nurse-registration.js
frontend/js/services/nurse.service.js
frontend/documentation/02-registration-forms/nurse/

Ahmed does not own the implementation of:

Shared Login
Shared OTP
Shared Password Recovery
Authentication Token System
Global Authorization System

Those remain under Mostafa's Authentication & Authorization responsibility.

49. Integration With Other Modules

Nurse Registration interacts with other modules but does not own them.

                    Nurse Registration
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
 Authentication     Provider Verification   Location
          │                │                │
          ↓                ↓                ↓
    OTP / Identity     Admin Review       Coordinates

After approval:

Nurse Registration
       ↓
Provider Verification
       ↓
Approved
       ↓
Nurse Dashboard

This keeps the frontend architecture modular and prevents responsibilities from being mixed across team members.