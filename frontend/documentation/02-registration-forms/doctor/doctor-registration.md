Doctor Registration – Frontend Specification

File: frontend/documentation/02-registration-forms/doctor/doctor-registration.md
Responsible: Ahd
Module: Doctor Registration
Application: TechCare

1. Overview

The Doctor Registration module allows a doctor to create a professional TechCare account and submit the information required to become a healthcare provider on the platform.

The registration process collects:

Account information
Personal information
Professional information
Medical specialization
Services
Pricing
Location
Availability
Professional documents
Final confirmation

The registration process is multi-step to keep the interface organized and easy to use.

The doctor is not considered an active provider immediately after completing the registration form.

The complete process is:

Doctor Registration
        ↓
Registration Submitted
        ↓
Pending Verification
        ↓
Admin Review
        ↓
Approved / Rejected / Additional Information Required
2. Ownership

Ahd is responsible for the frontend Doctor Registration module.

Ahd
 │
 ├── Doctor Registration Pages
 ├── Doctor Registration Forms
 ├── Doctor Form Validation
 ├── Doctor File Upload UI
 ├── Doctor Review Page
 ├── Doctor Registration Success Page
 ├── Doctor Registration JavaScript
 ├── Doctor Registration CSS
 └── Doctor Registration API Integration

Shared Authentication functionality is not duplicated here.

3. Frontend Structure

Recommended structure:

frontend/
│
├── pages/
│   └── doctor/
│       └── registration/
│           ├── doctor-register.html
│           ├── doctor-personal-info.html
│           ├── doctor-professional-info.html
│           ├── doctor-specialization-info.html
│           ├── doctor-service-info.html
│           ├── doctor-location-availability.html
│           ├── doctor-documents.html
│           ├── doctor-review.html
│           └── doctor-registration-success.html
│
├── css/
│   └── doctor/
│       └── doctor-registration.css
│
├── js/
│   ├── pages/
│   │   └── doctor/
│   │       └── doctor-registration.js
│   │
│   └── services/
│       └── doctor.service.js
│
└── assets/
    └── doctor/
4. Complete Registration Flow
┌────────────────────────────┐
│         Register            │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│       Select Doctor         │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│    Doctor Account Info      │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│    Shared OTP Verification  │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│    Personal Information     │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│   Professional Information  │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│      Specialization        │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│     Services & Pricing     │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Location & Availability     │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│        Documents            │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│      Review & Submit        │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Registration Submitted      │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│   Pending Verification      │
└────────────────────────────┘
5. Registration Steps
Step	Page	File
1	Doctor Account	doctor-register.html
2	Personal Information	doctor-personal-info.html
3	Professional Information	doctor-professional-info.html
4	Specialization	doctor-specialization-info.html
5	Services & Pricing	doctor-service-info.html
6	Location & Availability	doctor-location-availability.html
7	Documents	doctor-documents.html
8	Review	doctor-review.html
9	Success	doctor-registration-success.html

OTP verification is provided by the shared Authentication module.

6. Registration Progress Indicator

Every step should display the user's progress.

Example:

┌───────────────────────────────────────────────────────────────┐
│ Doctor Registration                                           │
│                                                               │
│ ● Account → ● Personal → ● Professional → ◉ Specialization  │
│                                                               │
│ ○ Services → ○ Location → ○ Documents → ○ Review             │
│                                                               │
│ Step 4 of 8                                                   │
└───────────────────────────────────────────────────────────────┘

Status meanings:

● Completed
◉ Current
○ Upcoming
7. Step 1 – Doctor Account Registration
Page
frontend/pages/doctor/registration/doctor-register.html
Purpose

Collects the basic credentials required to create the doctor's TechCare account.

7.1 UI Sketch
┌──────────────────────────────────────────────────┐
│               Create Doctor Account              │
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
│ [ ] I agree to the Terms & Privacy Policy        │
│                                                  │
│                    [ Continue ]                  │
│                                                  │
└──────────────────────────────────────────────────┘
8. Account Fields
Field	Type	Required
Full Name	Text	Yes
Email Address	Email	Yes
Phone Number	Tel	Yes
Password	Password	Yes
Confirm Password	Password	Yes
Terms & Privacy Agreement	Checkbox	Yes
9. Account Validation
Full Name

The frontend should verify:

Required
Valid characters
Acceptable length
No meaningless whitespace-only value
Email
Required
Valid email format
Normalized before submission

Backend remains responsible for checking whether the email already exists.

Phone
Required
Valid format
Accepted country format

Phone verification is performed through the shared Authentication module.

Password

The password must follow the centralized TechCare password policy.

Example UI:

Password Requirements

✓ Minimum length
✓ Uppercase letter
✓ Lowercase letter
✓ Number
✓ Special character
Confirm Password
Password
**************

Confirm Password
**************

✓ Passwords match
10. Shared OTP Verification

After account creation, the doctor is redirected into the shared verification process.

Doctor Registration
        ↓
Account Created
        ↓
OTP Verification
        ↓
Verification Successful
        ↓
Continue Registration

The Doctor module must not implement another OTP system.

Shared documentation:

frontend/documentation/01-authentication/otp-verification.md

After verification:

OTP Verified
    ↓
Doctor Registration
    ↓
Personal Information
11. Step 2 – Personal Information
Page
frontend/pages/doctor/registration/doctor-personal-info.html
Purpose

Collects the doctor's personal identity and contact details.

11.1 UI Sketch
┌─────────────────────────────────────────────────────┐
│             Personal Information                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│ First Name *                                        │
│ [_______________________________________________]   │
│                                                     │
│ Middle Name                                         │
│ [_______________________________________________]   │
│                                                     │
│ Last Name *                                         │
│ [_______________________________________________]   │
│                                                     │
│ Date of Birth *                                     │
│ [____ / ____ / ______]                              │
│                                                     │
│ Gender *                                            │
│ [ Select Gender ▼ ]                                 │
│                                                     │
│ National ID Number *                                │
│ [_______________________________________________]   │
│                                                     │
│ Profile Picture                                     │
│ [ Choose File ]                                     │
│                                                     │
│ Governorate *                                       │
│ [ Select Governorate ▼ ]                            │
│                                                     │
│ City *                                              │
│ [ Select City ▼ ]                                   │
│                                                     │
│ Detailed Address *                                  │
│ ┌─────────────────────────────────────────────────┐ │
│ │                                                 │ │
│ │                                                 │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ [ Back ]                           [ Continue ]     │
└─────────────────────────────────────────────────────┘
12. Personal Information Fields
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
13. Personal Information Validation

The frontend should validate:

First Name → Required
Last Name → Required
Date of Birth → Valid date
Gender → Valid selection
National ID → Valid format
Governorate → Required
City → Required
Address → Required

The backend must perform authoritative identity validation.

14. Step 3 – Professional Information
Page
frontend/pages/doctor/registration/doctor-professional-info.html
Purpose

Collects the doctor's professional and academic information.

14.1 UI Sketch
┌─────────────────────────────────────────────────────┐
│             Professional Information                │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Professional Title *                                │
│ [_______________________________________________]   │
│                                                     │
│ Medical Qualification *                             │
│ [ Select Qualification ▼ ]                           │
│                                                     │
│ University / Institution *                          │
│ [_______________________________________________]   │
│                                                     │
│ Graduation Year *                                   │
│ [____________]                                      │
│                                                     │
│ Professional License Number *                      │
│ [_______________________________________________]   │
│                                                     │
│ Years of Experience *                               │
│ [____________]                                      │
│                                                     │
│ Professional Bio *                                  │
│ ┌─────────────────────────────────────────────────┐ │
│ │                                                 │ │
│ │                                                 │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ [ Back ]                           [ Continue ]     │
└─────────────────────────────────────────────────────┘
15. Professional Fields
Field	Type	Required
Professional Title	Text/Select	Yes
Medical Qualification	Select	Yes
University / Institution	Text	Yes
Graduation Year	Number	Yes
Professional License Number	Text	Yes
Years of Experience	Number	Yes
Professional Bio	Textarea	Yes
16. Professional Validation
Graduation Year

The frontend should ensure:

Valid year
Not a future year
Reasonable range
Experience
Must be a non-negative number
License Number
Required
Valid format
Backend uniqueness verification
17. Step 4 – Specialization
Page
frontend/pages/doctor/registration/doctor-specialization-info.html
Purpose

Allows the doctor to identify their medical specialty and areas of practice.

17.1 UI Sketch
┌──────────────────────────────────────────────────┐
│             Medical Specialization               │
├──────────────────────────────────────────────────┤
│                                                  │
│ Main Specialty *                                 │
│ [ Select Specialty ▼ ]                            │
│                                                  │
│ Sub-Specialty                                    │
│ [ Select Sub-Specialty ▼ ]                       │
│                                                  │
│ Areas of Expertise                               │
│                                                  │
│ [ ] Pediatric Care                               │
│ [ ] Chronic Diseases                             │
│ [ ] Emergency Care                               │
│ [ ] Post-operative Care                          │
│ [ ] Preventive Care                              │
│ [ ] Other                                        │
│                                                  │
│ Specialty Description                            │
│ ┌──────────────────────────────────────────────┐ │
│ │                                              │ │
│ │                                              │ │
│ └──────────────────────────────────────────────┘ │
│                                                  │
│ [ Back ]                         [ Continue ]    │
└──────────────────────────────────────────────────┘
18. Specialization Fields
Field	Type	Required
Main Specialty	Select	Yes
Sub-Specialty	Select	No
Areas of Expertise	Multi-select	Yes
Specialty Description	Textarea	No
19. Specialty Examples

Depending on the application's approved medical taxonomy, specialties may include:

Cardiology
Dermatology
Neurology
Orthopedics
Pediatrics
Ophthalmology
Dentistry
General Practice
Internal Medicine
Gynecology
ENT
Psychiatry

The exact production list should come from the backend/business configuration rather than being duplicated as uncontrolled frontend values.

20. Step 5 – Services & Pricing
Page
frontend/pages/doctor/registration/doctor-service-info.html
Purpose

Defines the healthcare services the doctor provides and the associated pricing.

20.1 UI Sketch
┌────────────────────────────────────────────────────┐
│               Services & Pricing                   │
├────────────────────────────────────────────────────┤
│                                                    │
│ Service Type *                                     │
│ [ Medical Consultation ▼ ]                         │
│                                                    │
│ Consultation / Service Price *                    │
│ [____________________] EGP                         │
│                                                    │
│ Price Per Kilometer                               │
│ [____________________] EGP / KM                   │
│                                                    │
│ Service Radius                                    │
│ [____________________] KM                         │
│                                                    │
│ Consultation Duration                             │
│ [____________________] Minutes                    │
│                                                    │
│ Additional Service Fee                            │
│ [____________________] EGP                        │
│                                                    │
│ Currency *                                        │
│ [ EGP ▼ ]                                         │
│                                                    │
│ Services Offered                                  │
│                                                    │
│ [ ] General Consultation                          │
│ [ ] Follow-up Consultation                        │
│ [ ] Home Visit                                    │
│ [ ] Medical Examination                           │
│ [ ] Chronic Disease Follow-up                     │
│                                                    │
│ [ Back ]                          [ Continue ]     │
└────────────────────────────────────────────────────┘
21. Services & Pricing Fields
Field	Type	Required
Service Type	Select	Yes
Service Price	Number	Yes
Price Per Kilometer	Number	Conditional
Service Radius	Number	Conditional
Consultation Duration	Number	Yes
Additional Fee	Number	No
Currency	Select	Yes
Services Offered	Multi-select	Yes
22. Pricing Preview

The frontend may provide an estimated preview:

Base Service Price
        +
Travel Fee
        +
Additional Fee
        =
Estimated Service Cost

Example:

┌──────────────────────────────────┐
│ Pricing Preview                  │
├──────────────────────────────────┤
│ Base Price:       500 EGP        │
│ Travel Rate:        5 EGP / KM   │
│ Additional Fee:     0 EGP        │
│                                  │
│ Estimated Total                  │
│                    500 EGP       │
└──────────────────────────────────┘

These values are examples only.

The backend remains responsible for calculating and validating the final amount.

23. Home Visit Option

The doctor may offer home consultations.

Example:

Home Visit Availability

(●) Yes
( ) No

When enabled, the UI can expose:

Price Per Kilometer
Service Radius
Location Requirements

When disabled, unnecessary home-visit fields may be hidden or disabled.

This should be handled clearly in the UI.

24. Step 6 – Location & Availability
Page
frontend/pages/doctor/registration/doctor-location-availability.html
Purpose

Collects the doctor's practice/service location and available working schedule.

24.1 UI Sketch
┌────────────────────────────────────────────────────────┐
│ Location & Availability                                │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Governorate *                                          │
│ [ Select Governorate ▼ ]                               │
│                                                        │
│ City *                                                 │
│ [ Select City ▼ ]                                      │
│                                                        │
│ Detailed Address *                                     │
│ [_______________________________________________]      │
│                                                        │
│ Service Location                                       │
│                                                        │
│ [ Use Current Location ]                               │
│                                                        │
│ Latitude                                               │
│ [_______________________________________________]      │
│                                                        │
│ Longitude                                              │
│ [_______________________________________________]      │
│                                                        │
│ Home Visits                                            │
│ (●) Yes       ( ) No                                   │
│                                                        │
│ Working Days                                           │
│                                                        │
│ [ ] Sat  [ ] Sun  [ ] Mon                             │
│ [ ] Tue  [ ] Wed  [ ] Thu  [ ] Fri                    │
│                                                        │
│ Start Time                                             │
│ [__________]                                           │
│                                                        │
│ End Time                                               │
│ [__________]                                           │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
25. Location Fields
Field	Type	Required
Governorate	Select	Yes
City	Select	Yes
Detailed Address	Textarea	Yes
Latitude	Number	Conditional
Longitude	Number	Conditional
Use Current Location	Button	No
Home Visits	Radio	Yes
26. Location Interaction

The frontend can use browser geolocation:

[ Use Current Location ]
             ↓
      Browser Permission
             ↓
       Geolocation API
             ↓
     Latitude + Longitude
             ↓
        Form State
             ↓
        Backend API

The frontend must handle:

Permission Granted
Permission Denied
Location Unavailable
Request Timeout
Unsupported Browser

Example:

⚠ We could not access your current location.
Please enter your location manually.
27. Availability Schedule

Example:

Working Days:

☑ Saturday
☑ Sunday
☑ Monday
☑ Tuesday
☑ Wednesday
☐ Thursday
☐ Friday

Start Time:
09:00 AM

End Time:
05:00 PM

Validation rules:

At least one working day
        +
Valid start time
        +
Valid end time
        +
Start Time < End Time
28. Step 7 – Professional Documents
Page
frontend/pages/doctor/registration/doctor-documents.html
Purpose

Collects documents used to verify the doctor's professional identity and qualifications.

28.1 UI Sketch
┌────────────────────────────────────────────────────────┐
│              Professional Documents                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│ National ID - Front *                                  │
│ [ Choose File ]                                        │
│                                                        │
│ National ID - Back *                                   │
│ [ Choose File ]                                        │
│                                                        │
│ Medical Degree / Graduation Certificate *             │
│ [ Choose File ]                                        │
│                                                        │
│ Professional Practice License *                        │
│ [ Choose File ]                                        │
│                                                        │
│ Internship / Training Certificate                      │
│ [ Choose File ]                                        │
│                                                        │
│ Additional Supporting Document                         │
│ [ Choose File ]                                        │
│                                                        │
│ Supported Formats: PDF, JPG, JPEG, PNG                │
│                                                        │
│ [ Back ]                              [ Continue ]     │
└────────────────────────────────────────────────────────┘
29. Required Documents
Document	Required
National ID Front	Yes
National ID Back	Yes
Medical Degree / Graduation Certificate	Yes
Professional Practice License	Yes
Internship / Training Certificate	Optional / Business Rule
Additional Supporting Document	No

Required/optional status must follow the final backend verification rules.

30. File Upload Component

Each document upload should provide:

Choose File
       ↓
Validate
       ↓
Preview / Filename
       ↓
Upload
       ↓
Success / Error

Example:

┌────────────────────────────────────────────┐
│ medical_degree.pdf                         │
│ 3.2 MB                                     │
│                                            │
│ ✓ Upload completed                         │
│                              [ Remove ]    │
└────────────────────────────────────────────┘

During upload:

Uploading...
██████████████░░░░
31. File Validation

The frontend should validate:

File Selected
File Type
File Extension
File Size

Example:

Supported:
PDF
JPG
JPEG
PNG

Example error:

⚠ Unsupported file type.

Please upload a PDF, JPG, JPEG, or PNG file.

The backend must perform the final validation.

32. Step 8 – Review & Submit
Page
frontend/pages/doctor/registration/doctor-review.html
Purpose

Provides a complete summary before final submission.

32.1 UI Sketch
┌──────────────────────────────────────────────────────────┐
│               Review Doctor Registration                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ACCOUNT                                                  │
│ Full Name: Ahmed Example                                 │
│ Email: doctor@example.com                                │
│ Phone: 01XXXXXXXXX                                       │
│                                             [ Edit ]     │
│                                                          │
│ PERSONAL INFORMATION                                     │
│ Date of Birth: 01/01/1990                               │
│ Gender: Male                                             │
│ National ID: ***************                             │
│                                             [ Edit ]     │
│                                                          │
│ PROFESSIONAL INFORMATION                                 │
│ Qualification: Bachelor of Medicine                      │
│ University: Example University                            │
│ Experience: 7 Years                                      │
│ License: *************                                   │
│                                             [ Edit ]     │
│                                                          │
│ SPECIALIZATION                                           │
│ Specialty: Orthopedics                                  │
│ Expertise: Trauma, Joint Care                            │
│                                             [ Edit ]     │
│                                                          │
│ SERVICES                                                 │
│ Medical Consultation                                    │
│ Home Visit                                               │
│                                             [ Edit ]     │
│                                                          │
│ LOCATION & AVAILABILITY                                  │
│ Governorate: Dakahlia                                   │
│ City: Mansoura                                          │
│ Home Visits: Yes                                        │
│                                             [ Edit ]     │
│                                                          │
│ DOCUMENTS                                                │
│ ✓ National ID Front                                     │
│ ✓ National ID Back                                      │
│ ✓ Medical Degree                                        │
│ ✓ Practice License                                      │
│                                             [ Edit ]     │
│                                                          │
│ [ ] I confirm that all information is correct.           │
│                                                          │
│ [ Back ]                     [ Submit Registration ]     │
└──────────────────────────────────────────────────────────┘
33. Edit Navigation

Every section should have an Edit button.

Example:

Professional Information                    [ Edit ]

Clicking Edit should return the user to the corresponding step:

Review
  ↓
Edit Professional Information
  ↓
Professional Page
  ↓
Save
  ↓
Review

The already entered information must remain available.

34. Sensitive Information

Sensitive information must be masked where practical.

Example:

National ID:
298***********

Professional License:
DOC-****-9381

Documents should be represented by filename/status rather than unnecessarily exposing their contents.

35. Final Confirmation

Before submission:

[ ] I confirm that all information provided is accurate.

The Submit button remains disabled until the confirmation is selected.

Example:

☐ I confirm that all information is correct.

[ Submit Registration ]

After checking:

☑ I confirm that all information is correct.

[ Submit Registration ]
36. Final Submission State

Normal:

[ Submit Registration ]

Loading:

[ ⏳ Submitting... ]

During submission:

Submit Button → Disabled

This prevents duplicate requests.

37. Registration Success Page
Page
frontend/pages/doctor/registration/doctor-registration-success.html
37.1 UI Sketch
┌──────────────────────────────────────────────────┐
│                                                  │
│                     ✓                            │
│                                                  │
│      Registration Submitted Successfully        │
│                                                  │
│ Your doctor registration has been submitted     │
│ and is currently under professional review.     │
│                                                  │
│ Registration ID                                  │
│ DOC-2026-000123                                  │
│                                                  │
│ Verification Status                              │
│ PENDING_VERIFICATION                             │
│                                                  │
│ You will be notified when the review is complete.│
│                                                  │
│                [ Go to Login ]                  │
│                                                  │
└──────────────────────────────────────────────────┘

The displayed Registration ID is illustrative.

38. Post-Registration Behavior

After successful registration:

Registration Submitted
        ↓
Pending Verification
        ↓
Go to Login

The doctor must not automatically receive active provider access.

The backend determines whether the doctor is:

Approved
Rejected
Pending
Additional Information Required
Suspended
39. Verification Status UI

Example:

┌──────────────────────────────────────┐
│ Professional Verification            │
│                                      │
│              ⏳ Pending              │
│                                      │
│ Your documents are currently        │
│ being reviewed by TechCare Admin.   │
│                                      │
│ Status: PENDING_VERIFICATION         │
└──────────────────────────────────────┘
40. Form State

The entire registration process should maintain a consistent state.

Conceptual structure:

const doctorRegistration = {
    account: {},
    personal: {},
    professional: {},
    specialization: {},
    services: {},
    location: {},
    availability: {},
    documents: {}
};

Each step updates its own section.

Example:

Personal Page
      ↓
personal{}

Professional Page
      ↓
professional{}

Specialization Page
      ↓
specialization{}

Services Page
      ↓
services{}

Final submission:

All Sections
      ↓
Complete Registration Payload
      ↓
Backend API
41. Validation Architecture

Validation should exist at two levels:

Frontend Validation
        +
Backend Validation

Frontend validation is for user experience.

Backend validation is authoritative.

The frontend must never assume that hiding a field or disabling a button provides security.

42. Validation Examples
Missing Specialty
⚠ Please select your medical specialty.
Missing License
⚠ Professional license number is required.
Invalid Experience
⚠ Years of experience cannot be negative.
Invalid Schedule
⚠ End time must be later than start time.
Missing Document
⚠ Please upload your professional practice license.
43. API Validation Error Handling

When the backend returns a field-specific error, the frontend should display it next to that field.

Example backend response:

professionalLicenseNumber:
"This license number is already registered."

Frontend:

Professional Practice License Number *

[ DOC-123456________________ ]

⚠ This license number is already registered.
44. Global Error Handling

The frontend should also handle:

Network Error
Server Error
Authentication Error
Validation Error
Upload Error
Duplicate Data
Expired Session
Unexpected Error

Example:

┌────────────────────────────────────────────┐
│ Unable to submit registration.             │
│                                            │
│ Please check your connection and try again.│
│                                            │
│               [ Try Again ]                │
└────────────────────────────────────────────┘
45. Step Navigation

Every step should provide:

[ Back ]                     [ Continue ]

Review:

[ Back ]                [ Submit Registration ]

Navigation rules:

Continue
   ↓
Validate Current Step
   ↓
Valid → Next Step
Invalid → Stay + Show Errors
46. Preserving Form Data

Moving between steps must not erase previous information.

Example:

Personal
   ↓
Professional
   ↓
Specialization
   ↓
Back
   ↓
Professional

The previously entered professional information should still appear.

47. Responsive Design

The Doctor Registration UI must work on:

Desktop
Laptop
Tablet
Mobile

Desktop:

┌─────────────────────────────────────────────────┐
│             Doctor Registration                 │
│                                                 │
│ First Name                Last Name             │
│ [_____________]           [_____________]       │
│                                                 │
└─────────────────────────────────────────────────┘

Mobile:

┌────────────────────────────┐
│ Doctor Registration        │
│                            │
│ First Name                 │
│ [______________________]   │
│                            │
│ Last Name                  │
│ [______________________]   │
│                            │
│ [ Continue ]               │
└────────────────────────────┘

No normal mobile screen should require horizontal scrolling.

48. Accessibility

The registration frontend must support:

Semantic HTML
Associated Labels
Keyboard Navigation
Visible Focus
Accessible Buttons
Readable Error Messages
Accessible File Inputs
Sufficient Contrast

Example:

<label for="specialty">
    Medical Specialty
</label>

<select id="specialty" name="specialty">
    <option value="">Select Specialty</option>
</select>
49. Input Structure

A consistent input pattern should be used:

Label
  ↓
Input
  ↓
Helper Text
  ↓
Error Message

Example:

Professional License Number *

[____________________________]

Enter your official professional license number.

⚠ This license number is already registered.
50. Loading States

Examples:

Loading Specialties
Medical Specialty

[ Loading specialties... ]
Loading Cities
City

[ Loading cities... ]
Uploading Document
Uploading medical_degree.pdf...

████████████░░░░░░
Submission
Submitting your registration...
51. Security Rules

The frontend must avoid exposing sensitive information unnecessarily.

Sensitive data includes:

National ID
Professional License
Uploaded Documents
Personal Address
Location Coordinates

Do not:

Put sensitive values in URLs
Log sensitive values to console
Expose them in unnecessary frontend messages
Store them insecurely

The backend must enforce authorization and verification.

52. Duplicate Submission Protection

The following should prevent duplicate submissions:

Submit
  ↓
Disable Button
  ↓
Show Loading
  ↓
Send Request
  ↓
Wait for Response

Server-side duplicate protection should also exist.

53. Unsaved Changes Warning

For long forms, the frontend may warn users before leaving an incomplete registration.

Example:

┌────────────────────────────────────────────┐
│ Leave registration?                        │
│                                            │
│ Your entered information may be lost.      │
│                                            │
│ [ Stay ]                    [ Leave ]      │
└────────────────────────────────────────────┘
54. API Service

Doctor API communication should be separated from UI logic.

File:

frontend/js/services/doctor.service.js

Conceptual structure:

const doctorService = {

    createRegistration: async (payload) => {
        // API request
    },

    uploadDocument: async (file) => {
        // Document upload
    },

    submitRegistration: async (payload) => {
        // Final submission
    }

};

Recommended architecture:

Doctor HTML
     ↓
Doctor Registration JS
     ↓
Doctor Service
     ↓
ASP.NET Core API
55. Registration Lifecycle
DRAFT
  ↓
ACCOUNT_CREATED
  ↓
OTP_VERIFIED
  ↓
PROFILE_COMPLETED
  ↓
DOCUMENTS_UPLOADED
  ↓
SUBMITTED
  ↓
PENDING_VERIFICATION
  ↓
┌───────────────┬─────────────────────┐
↓               ↓                     ↓
APPROVED      REJECTED      ADDITIONAL_INFO_REQUIRED

The actual state values must follow the backend API contract.

56. Integration With Authentication

Doctor Registration depends on the shared Authentication module:

            Authentication
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
Account Creation       OTP Verification
        │                   │
        └─────────┬─────────┘
                  ↓
          Doctor Registration

Doctor Registration does not own:

Login
OTP System
Password Recovery
Token Management
Role Selection
Global Authorization
57. Integration With Provider Verification

Doctor Registration connects directly to Provider Verification after submission.

Doctor Registration
        ↓
Documents Submitted
        ↓
Provider Verification
        ↓
Admin Review
        ↓
Verification Result

A doctor account can therefore exist while the professional provider status is still pending.

58. Integration With Location

Location information will later support:

Nearby Doctor Search
Distance Calculation
Home Visit Availability
Travel Fee Calculation
Patient Discovery

Therefore the frontend should collect location data in a consistent and structured way.

59. Integration With Doctor Dashboard

After verification:

Registration
      ↓
Verification
      ↓
Approved
      ↓
Doctor Dashboard

The Doctor Dashboard is outside the scope of this registration file.

The registration module is responsible only for collecting and submitting the data required to enter that lifecycle.

60. Complete UI Flow
                    ┌───────────────┐
                    │ Register      │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Doctor Role   │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Account       │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ OTP           │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Personal      │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Professional  │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Specialization│
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Services      │
                    │ & Pricing     │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Location      │
                    │ & Availability│
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Documents     │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Review        │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Submit        │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Success       │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Pending       │
                    │ Verification  │
                    └───────────────┘
61. Final Implementation Checklist
[ ] Doctor Account Page
[ ] Personal Information Page
[ ] Professional Information Page
[ ] Specialization Page
[ ] Services Page
[ ] Pricing Section
[ ] Location Page
[ ] Availability Section
[ ] Documents Page
[ ] Review Page
[ ] Success Page

[ ] Registration Progress Indicator
[ ] Step Navigation
[ ] Form State Management
[ ] Client-side Validation
[ ] Server-side Error Mapping
[ ] Loading States
[ ] Empty States
[ ] File Upload Validation
[ ] File Upload Status
[ ] Duplicate Submission Protection

[ ] Responsive Design
[ ] Accessibility
[ ] Sensitive Data Protection
[ ] API Service Integration
[ ] Pending Verification UI
[ ] Authentication Integration
[ ] Provider Verification Integration
62. Ownership Boundary

Ahd owns:

frontend/documentation/02-registration-forms/doctor/

and the Doctor Registration implementation:

frontend/pages/doctor/registration/
frontend/css/doctor/doctor-registration.css
frontend/js/pages/doctor/doctor-registration.js
frontend/js/services/doctor.service.js

Ahd does not own the shared Authentication implementation.

Shared Authentication remains under Mostafa:

frontend/documentation/01-authentication/
63. Module Relationship
                    ┌──────────────────────┐
                    │ Authentication       │
                    │ Mostafa              │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Doctor Registration  │
                    │ Ahd                  │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Provider Verification│
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ Doctor Dashboard     │
                    └──────────────────────┘