Pharmacy Registration – Frontend Specification

File: frontend/documentation/02-registration-forms/pharmacy/pharmacy-registration.md
Responsible: Alaa
Module: Pharmacy Registration
Application: TechCare

1. Overview

The Pharmacy Registration module allows a pharmacist or pharmacy representative to register a pharmacy on the TechCare platform.

Unlike individual healthcare-provider registration, pharmacy registration contains both:

Pharmacist Information
        +
Pharmacy Information
        +
Branch Information
        +
Medicine Availability Configuration
        +
Location
        +
Operating Hours
        +
Professional Documents

The purpose of this module is to allow verified pharmacies to appear in TechCare search results and provide patients with information about nearby pharmacies and medicine availability.

The registration process is multi-step so that pharmacists do not have to complete one large form.

The overall lifecycle is:

Pharmacy Registration
        ↓
Registration Submitted
        ↓
Pending Verification
        ↓
Admin Review
        ↓
Approved / Rejected / Additional Information Required
        ↓
Pharmacy Activated

The pharmacy must not become an active marketplace/service provider before the required verification process is completed.

2. Ownership

Alaa is responsible for the Pharmacy Registration frontend module.

Alaa
 │
 ├── Pharmacy Registration Pages
 ├── Pharmacist Information Forms
 ├── Pharmacy Information Forms
 ├── Branch Information UI
 ├── Location UI
 ├── Operating Hours UI
 ├── Medicine Availability UI
 ├── Documents Upload UI
 ├── Form Validation
 ├── Review & Submit UI
 ├── Registration Success UI
 ├── Pharmacy Registration JavaScript
 ├── Pharmacy Registration CSS
 └── Pharmacy API Integration

Alaa does not own shared Authentication functionality.

3. Frontend Structure

Recommended structure:

frontend/
│
├── pages/
│   └── pharmacy/
│       └── registration/
│           ├── pharmacy-register.html
│           ├── pharmacist-info.html
│           ├── pharmacy-info.html
│           ├── branch-info.html
│           ├── medicine-services.html
│           ├── location-hours.html
│           ├── pharmacy-documents.html
│           ├── pharmacy-review.html
│           └── pharmacy-registration-success.html
│
├── css/
│   └── pharmacy/
│       └── pharmacy-registration.css
│
├── js/
│   ├── pages/
│   │   └── pharmacy/
│   │       └── pharmacy-registration.js
│   │
│   └── services/
│       └── pharmacy.service.js
│
└── assets/
    └── pharmacy/
4. Complete Registration Flow
┌─────────────────────────────┐
│          Register            │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│      Select Pharmacy        │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│    Pharmacist Information    │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│    Shared OTP Verification   │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│      Pharmacy Information    │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│       Branch Information     │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│ Medicine & Services Setup    │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│ Location & Operating Hours   │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│      Verification Documents  │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│       Review & Submit        │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│    Registration Submitted    │
└─────────────┬───────────────┘
              ↓
┌─────────────────────────────┐
│     Pending Verification     │
└─────────────────────────────┘
5. Registration Steps
Step	Page	File
1	Pharmacist Account	pharmacy-register.html
2	Pharmacist Information	pharmacist-info.html
3	Pharmacy Information	pharmacy-info.html
4	Branch Information	branch-info.html
5	Medicines & Services	medicine-services.html
6	Location & Hours	location-hours.html
7	Documents	pharmacy-documents.html
8	Review	pharmacy-review.html
9	Success	pharmacy-registration-success.html

OTP verification is handled by the centralized Authentication module.

6. Important Pharmacy Concept

The frontend must distinguish between:

Pharmacist

and

Pharmacy

The pharmacist is the professional/account holder.

The pharmacy is the business/service entity.

A pharmacy may later contain:

Pharmacy
   │
   ├── Branch 1
   ├── Branch 2
   └── Branch 3

The initial registration should establish the primary pharmacy profile and its initial branch.

Additional branches can later be managed from the Pharmacy Dashboard if that feature is enabled.

7. Registration Progress Indicator

The page should show the current progress.

Example:

┌─────────────────────────────────────────────────────────────────┐
│ Pharmacy Registration                                           │
│                                                                 │
│ ● Account → ● Pharmacist → ◉ Pharmacy → ○ Branch               │
│                                                                 │
│ ○ Medicines → ○ Location → ○ Documents → ○ Review              │
│                                                                 │
│ Step 3 of 8                                                     │
└─────────────────────────────────────────────────────────────────┘

State meanings:

● Completed
◉ Current
○ Upcoming
8. Step 1 – Pharmacy Account Registration
Page
frontend/pages/pharmacy/registration/pharmacy-register.html
Purpose

This page creates the account used by the pharmacist/pharmacy representative.

8.1 UI Sketch
┌──────────────────────────────────────────────────┐
│            Create Pharmacy Account               │
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
└──────────────────────────────────────────────────┘
9. Account Fields
Field	Type	Required
Full Name	Text	Yes
Email Address	Email	Yes
Phone Number	Tel	Yes
Password	Password	Yes
Confirm Password	Password	Yes
Terms & Privacy Agreement	Checkbox	Yes
10. Account Validation

The frontend must validate:

Full Name
Email
Phone
Password
Confirm Password
Terms Agreement

Example:

⚠ Email address is required.

⚠ Please enter a valid phone number.

⚠ Passwords do not match.

The backend remains responsible for uniqueness and authoritative validation.

11. Shared OTP Verification

The pharmacy account follows the shared Authentication flow.

Pharmacy Registration
       ↓
Account Created
       ↓
OTP Verification
       ↓
Verification Successful
       ↓
Continue Pharmacy Registration

The Pharmacy module does not implement a separate OTP system.

Shared documentation:

frontend/documentation/01-authentication/otp-verification.md
12. Step 2 – Pharmacist Information
Page
frontend/pages/pharmacy/registration/pharmacist-info.html
Purpose

Collect the professional identity of the pharmacist responsible for the pharmacy.

12.1 UI Sketch
┌────────────────────────────────────────────────────┐
│              Pharmacist Information               │
├────────────────────────────────────────────────────┤
│                                                    │
│ First Name *                                       │
│ [____________________________________________]     │
│                                                    │
│ Middle Name                                        │
│ [____________________________________________]     │
│                                                    │
│ Last Name *                                        │
│ [____________________________________________]     │
│                                                    │
│ Date of Birth *                                    │
│ [____ / ____ / ______]                             │
│                                                    │
│ Gender *                                           │
│ [ Select Gender ▼ ]                                │
│                                                    │
│ National ID Number *                               │
│ [____________________________________________]     │
│                                                    │
│ Pharmacist License Number *                        │
│ [____________________________________________]     │
│                                                    │
│ Qualification *                                    │
│ [ Select Qualification ▼ ]                         │
│                                                    │
│ Years of Experience                                │
│ [________________]                                 │
│                                                    │
│ [ Back ]                           [ Continue ]    │
└────────────────────────────────────────────────────┘
13. Pharmacist Information Fields
Field	Type	Required
First Name	Text	Yes
Middle Name	Text	No
Last Name	Text	Yes
Date of Birth	Date	Yes
Gender	Select/Radio	Yes
National ID Number	Text	Yes
Pharmacist License Number	Text	Yes
Qualification	Select	Yes
Years of Experience	Number	No
14. Pharmacist Validation

Examples:

First Name → Required
Last Name → Required
Date of Birth → Valid date
National ID → Valid format
License Number → Required
Qualification → Required
Experience → Non-negative

Backend must check whether the license or national ID is already associated with an existing account where applicable.

15. Step 3 – Pharmacy Information
Page
frontend/pages/pharmacy/registration/pharmacy-info.html
Purpose

Collect the pharmacy's business information.

15.1 UI Sketch
┌────────────────────────────────────────────────────┐
│                Pharmacy Information                │
├────────────────────────────────────────────────────┤
│                                                    │
│ Pharmacy Name *                                    │
│ [____________________________________________]     │
│                                                    │
│ Pharmacy Type *                                    │
│ [ Select Pharmacy Type ▼ ]                         │
│                                                    │
│ Commercial Registration Number                    │
│ [____________________________________________]     │
│                                                    │
│ Pharmacy License Number *                          │
│ [____________________________________________]     │
│                                                    │
│ Pharmacy Phone *                                   │
│ [____________________________________________]     │
│                                                    │
│ Pharmacy Email                                     │
│ [____________________________________________]     │
│                                                    │
│ Pharmacy Description                               │
│ ┌────────────────────────────────────────────────┐ │
│ │                                                │ │
│ │                                                │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ Pharmacy Logo                                      │
│ [ Choose File ]                                    │
│                                                    │
│ [ Back ]                           [ Continue ]    │
└────────────────────────────────────────────────────┘
16. Pharmacy Information Fields
Field	Type	Required
Pharmacy Name	Text	Yes
Pharmacy Type	Select	Yes
Commercial Registration Number	Text	Business Rule
Pharmacy License Number	Text	Yes
Pharmacy Phone	Tel	Yes
Pharmacy Email	Email	No
Pharmacy Description	Textarea	No
Pharmacy Logo	File	No
17. Pharmacy Type

The interface may support categories such as:

Community Pharmacy
Hospital Pharmacy
Specialized Pharmacy
Other

The final taxonomy should be controlled by the backend/business configuration.

18. Pharmacy Name Validation

The frontend should ensure:

Required
Acceptable length
Not only whitespace
Supported characters

Example:

Pharmacy Name *

[____________________________]

⚠ Pharmacy name is required.
19. Step 4 – Branch Information
Page
frontend/pages/pharmacy/registration/branch-info.html
Purpose

Defines the initial pharmacy branch.

This is important because patients will later search for pharmacies based on branch location.

19.1 UI Sketch
┌──────────────────────────────────────────────────────┐
│                 Primary Branch                       │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Branch Name *                                        │
│ [_______________________________________________]    │
│                                                      │
│ Branch Phone *                                       │
│ [_______________________________________________]    │
│                                                      │
│ Governorate *                                        │
│ [ Select Governorate ▼ ]                             │
│                                                      │
│ City *                                               │
│ [ Select City ▼ ]                                    │
│                                                      │
│ Detailed Address *                                   │
│ ┌──────────────────────────────────────────────────┐ │
│ │                                                  │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ [ Use Current Location ]                             │
│                                                      │
│ Latitude                                             │
│ [____________________________]                      │
│                                                      │
│ Longitude                                            │
│ [____________________________]                      │
│                                                      │
│ [ Back ]                           [ Continue ]      │
└──────────────────────────────────────────────────────┘
20. Branch Fields
Field	Type	Required
Branch Name	Text	Yes
Branch Phone	Tel	Yes
Governorate	Select	Yes
City	Select	Yes
Detailed Address	Textarea	Yes
Latitude	Number	Conditional
Longitude	Number	Conditional
21. Branch Location

The branch location is especially important for:

Nearby Pharmacy Search
Distance Calculation
Medicine Discovery
Patient Navigation
Branch Availability

The frontend may allow:

[ Use Current Location ]

which uses the browser's geolocation capability.

22. Step 5 – Medicines & Services
Page
frontend/pages/pharmacy/registration/medicine-services.html
Purpose

Defines the pharmacy's medicine-related capabilities and service categories.

The registration form should not require the pharmacist to manually enter the entire pharmacy inventory during account registration.

The registration should establish the pharmacy's supported categories and enable inventory management later.

22.1 UI Sketch
┌──────────────────────────────────────────────────────┐
│             Medicines & Services                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Services Provided *                                  │
│                                                      │
│ [ ] Prescription Medicines                           │
│ [ ] OTC Medicines                                    │
│ [ ] Baby Care Products                               │
│ [ ] Medical Supplies                                 │
│ [ ] Vitamins & Healthcare Products                   │
│                                                      │
│ Medicine Availability                                │
│                                                      │
│ (●) We maintain medicine availability information    │
│ ( ) Availability managed later                       │
│                                                      │
│ Home Delivery                                        │
│ (●) Available       ( ) Not Available               │
│                                                      │
│ Delivery Radius                                      │
│ [________________] KM                                │
│                                                      │
│ Pharmacy Services Description                       │
│ ┌──────────────────────────────────────────────────┐ │
│ │                                                  │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ [ Back ]                           [ Continue ]      │
└──────────────────────────────────────────────────────┘
23. Services Fields
Field	Type	Required
Services Provided	Multi-select	Yes
Medicine Availability Mode	Radio	Yes
Home Delivery	Radio	Yes
Delivery Radius	Number	Conditional
Service Description	Textarea	No
24. Medicine Availability Concept

The platform's pharmacy search functionality may later allow:

Patient
   ↓
Search Medicine
   ↓
Nearby Pharmacies
   ↓
Branch
   ↓
Availability

Example UI concept:

┌──────────────────────────────────────────────┐
│ Medicine Availability                        │
├──────────────────────────────────────────────┤
│ The pharmacy can maintain medicine           │
│ availability through the Pharmacy Dashboard. │
│                                              │
│ Inventory Management                         │
│ [ Enabled ]                                  │
└──────────────────────────────────────────────┘

The registration form should not become an inventory-management system.

Inventory operations belong to the Pharmacy Dashboard.

25. Home Delivery

When Home Delivery is enabled:

Home Delivery
(●) Yes

show:

Delivery Radius
[_____________] KM

When disabled:

Home Delivery
( ) No

the radius field may be hidden or disabled.

Validation:

Home Delivery = Yes
        +
Delivery Radius > 0
26. Step 6 – Location & Operating Hours
Page
frontend/pages/pharmacy/registration/location-hours.html
Purpose

Defines the branch location and operating schedule.

26.1 UI Sketch
┌─────────────────────────────────────────────────────────┐
│           Location & Operating Hours                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Governorate *                                           │
│ [ Select Governorate ▼ ]                                │
│                                                         │
│ City *                                                  │
│ [ Select City ▼ ]                                       │
│                                                         │
│ Detailed Address *                                      │
│ [_______________________________________________]       │
│                                                         │
│ [ Use Current Location ]                                │
│                                                         │
│ Latitude                                                │
│ [_______________________________________________]       │
│                                                         │
│ Longitude                                               │
│ [_______________________________________________]       │
│                                                         │
│ Operating Days *                                        │
│                                                         │
│ [ ] Sat  [ ] Sun  [ ] Mon  [ ] Tue                     │
│ [ ] Wed  [ ] Thu  [ ] Fri                              │
│                                                         │
│ Opening Time *                                          │
│ [__________]                                            │
│                                                         │
│ Closing Time *                                          │
│ [__________]                                            │
│                                                         │
│ Open 24 Hours                                           │
│ [ ] Yes                                                 │
│                                                         │
│ [ Back ]                              [ Continue ]      │
└─────────────────────────────────────────────────────────┘
27. Operating Hours

The pharmacy must specify its operating schedule.

Example:

Saturday   ✓
Sunday     ✓
Monday     ✓
Tuesday    ✓
Wednesday  ✓
Thursday   ✓
Friday     ✓

Opening: 09:00 AM
Closing: 11:00 PM
28. 24-Hour Pharmacy

The UI may support:

Open 24 Hours
[✓]

When enabled, regular opening and closing time fields may be disabled.

Example:

Open 24 Hours
☑ Yes

Opening Time
[ Disabled ]

Closing Time
[ Disabled ]

Validation:

24 Hours = Yes
        ↓
No standard opening/closing validation required
29. Operating Hours Validation

When not operating 24 hours:

At least one day selected
        +
Opening time valid
        +
Closing time valid
        +
Opening < Closing

Invalid:

Opening: 10:00 PM
Closing: 08:00 AM

The UI must provide a clear message.

30. Location Interaction

Conceptual flow:

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

Possible states:

Location Found
Permission Denied
Location Unavailable
Timeout
Unsupported Browser

Example:

⚠ Unable to access your current location.
Please enter the branch address manually.
31. Step 7 – Pharmacy Documents
Page
frontend/pages/pharmacy/registration/pharmacy-documents.html
Purpose

Collect documents required for pharmacy and pharmacist verification.

31.1 UI Sketch
┌──────────────────────────────────────────────────────┐
│             Pharmacy Verification Documents          │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Pharmacist National ID - Front *                     │
│ [ Choose File ]                                      │
│                                                      │
│ Pharmacist National ID - Back *                      │
│ [ Choose File ]                                      │
│                                                      │
│ Pharmacist Qualification Certificate *               │
│ [ Choose File ]                                      │
│                                                      │
│ Pharmacist Professional License *                    │
│ [ Choose File ]                                      │
│                                                      │
│ Pharmacy License *                                   │
│ [ Choose File ]                                      │
│                                                      │
│ Commercial Registration                              │
│ [ Choose File ]                                      │
│                                                      │
│ Pharmacy Ownership / Authorization Document         │
│ [ Choose File ]                                      │
│                                                      │
│ Additional Supporting Document                       │
│ [ Choose File ]                                      │
│                                                      │
│ Supported: PDF, JPG, JPEG, PNG                       │
│                                                      │
│ [ Back ]                           [ Continue ]      │
└──────────────────────────────────────────────────────┘
32. Document Requirements

Suggested document model:

Document	Required
Pharmacist National ID Front	Yes
Pharmacist National ID Back	Yes
Pharmacist Qualification Certificate	Yes
Pharmacist Professional License	Yes
Pharmacy License	Yes
Commercial Registration	Business Rule
Ownership/Authorization Document	Business Rule
Additional Supporting Document	No

The final required/optional status must follow the backend verification rules.

33. Document Upload Component

Each file should show:

Document Name
Filename
File Size
Upload Status
Remove / Replace

Example:

┌────────────────────────────────────────────┐
│ pharmacy_license.pdf                       │
│ 2.1 MB                                     │
│                                            │
│ ✓ Uploaded successfully                   │
│                               [ Remove ]   │
└────────────────────────────────────────────┘

During upload:

Uploading...
██████████████░░░░
34. File Validation

Before upload:

File Selected
     ↓
File Type
     ↓
File Extension
     ↓
File Size
     ↓
Upload

Example accepted formats:

PDF
JPG
JPEG
PNG

Example:

⚠ Unsupported file type.
Please upload a PDF, JPG, JPEG, or PNG file.

The backend must perform final validation.

35. Step 8 – Review & Submit
Page
frontend/pages/pharmacy/registration/pharmacy-review.html
Purpose

Displays a complete summary of the registration before submission.

35.1 UI Sketch
┌─────────────────────────────────────────────────────────────┐
│              Review Pharmacy Registration                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ PHARMACIST                                                  │
│ Name: Ahmed Example                                         │
│ License: ************                                       │
│ Qualification: Bachelor of Pharmacy                         │
│                                              [ Edit ]       │
│                                                             │
│ PHARMACY                                                    │
│ Name: Example Pharmacy                                      │
│ License: ************                                       │
│ Phone: 01XXXXXXXXX                                          │
│                                              [ Edit ]       │
│                                                             │
│ PRIMARY BRANCH                                              │
│ Branch: Main Branch                                         │
│ City: Mansoura                                              │
│ Address: ***************                                    │
│                                              [ Edit ]       │
│                                                             │
│ SERVICES                                                    │
│ ✓ Prescription Medicines                                    │
│ ✓ OTC Medicines                                             │
│ ✓ Medical Supplies                                          │
│ Home Delivery: Yes                                          │
│                                              [ Edit ]       │
│                                                             │
│ OPERATING HOURS                                             │
│ Saturday – Thursday                                         │
│ 09:00 AM – 11:00 PM                                         │
│                                              [ Edit ]       │
│                                                             │
│ DOCUMENTS                                                   │
│ ✓ National ID Front                                         │
│ ✓ National ID Back                                          │
│ ✓ Professional License                                      │
│ ✓ Pharmacy License                                          │
│                                              [ Edit ]       │
│                                                             │
│ [ ] I confirm that all information is correct.              │
│                                                             │
│ [ Back ]                 [ Submit Registration ]            │
└─────────────────────────────────────────────────────────────┘
36. Review Page Rules

The user must be able to edit each major section.

Example:

Review
  ↓
[ Edit Pharmacy ]
  ↓
Pharmacy Information
  ↓
Update
  ↓
Review

Previously entered data must remain available.

37. Sensitive Information

Sensitive data should be masked where practical.

Example:

National ID:
298***********

Professional License:
PHA-****-3812

Pharmacy License:
LIC-****-8221

Uploaded documents should be represented by filename and status instead of unnecessary full document exposure.

38. Final Confirmation

Before final submission:

[ ] I confirm that all provided information is accurate and that
    I have permission to register this pharmacy on TechCare.

The Submit button should not be enabled until confirmation is completed.

39. Final Submission

Flow:

Submit
  ↓
Validate All Data
  ↓
Disable Button
  ↓
Show Loading
  ↓
Send Registration
  ↓
Success / Error
40. Submission States
Normal
[ Submit Registration ]
Loading
[ ⏳ Submitting... ]
Disabled
[ Submit Registration ]

The button must be disabled while submission is in progress.

41. Pharmacy Registration Success
Page
frontend/pages/pharmacy/registration/pharmacy-registration-success.html
41.1 UI Sketch
┌───────────────────────────────────────────────────┐
│                                                   │
│                    ✓                              │
│                                                   │
│       Pharmacy Registration Submitted             │
│                                                   │
│ Your pharmacy registration has been submitted     │
│ successfully and is currently under review.      │
│                                                   │
│ Registration ID                                   │
│ PH-2026-000123                                    │
│                                                   │
│ Verification Status                               │
│ PENDING_VERIFICATION                              │
│                                                   │
│ You will be notified when the verification        │
│ process is completed.                             │
│                                                   │
│                  [ Go to Login ]                  │
│                                                   │
└───────────────────────────────────────────────────┘

The Registration ID shown is illustrative.

42. Post-Registration Behavior

After successful submission:

Submitted
    ↓
Pending Verification
    ↓
Go to Login

The pharmacy must not immediately become searchable as a verified pharmacy unless the backend indicates that it is approved and active.

43. Verification Status

Possible statuses:

PENDING_VERIFICATION
APPROVED
REJECTED
ADDITIONAL_INFORMATION_REQUIRED
SUSPENDED

Example:

┌──────────────────────────────────────────────┐
│ Pharmacy Verification                        │
│                                              │
│                ⏳ Pending                    │
│                                              │
│ Your pharmacy documents are being reviewed.  │
│                                              │
│ Status: PENDING_VERIFICATION                 │
└──────────────────────────────────────────────┘
44. Registration State

The frontend may maintain the form as:

const pharmacyRegistration = {
    account: {},
    pharmacist: {},
    pharmacy: {},
    branch: {},
    services: {},
    location: {},
    operatingHours: {},
    documents: {}
};

Each step updates only its own section.

Example:

Account
   ↓
account{}

Pharmacist
   ↓
pharmacist{}

Pharmacy
   ↓
pharmacy{}

Branch
   ↓
branch{}

Services
   ↓
services{}

At submission:

All Sections
      ↓
Complete Registration Payload
      ↓
ASP.NET Core API
45. Validation Strategy

Two validation layers must exist:

Frontend Validation
        +
Backend Validation

Frontend validation improves usability.

Backend validation is authoritative.

The frontend must never be treated as a security boundary.

46. Important Pharmacy Validation
Pharmacy Name
Required
Valid length
Not whitespace only
Pharmacy License
Required
Valid format
Backend uniqueness check
Pharmacist License
Required
Valid format
Backend uniqueness check
Delivery Radius

When delivery is enabled:

Required
Greater than zero
Operating Hours
At least one day
+
Valid opening time
+
Valid closing time
47. Dynamic Fields

The pharmacy registration form contains conditional sections.

Example:

Home Delivery
      │
      ├── No → Hide Delivery Radius
      │
      └── Yes
            ↓
      Show Delivery Radius

Another example:

Open 24 Hours
      │
      ├── Yes → Disable Opening / Closing Time
      │
      └── No → Require Opening / Closing Time

This creates a cleaner user experience.

48. Dependent Dropdowns

Governorate and City should be dependent.

Flow:

Select Governorate
        ↓
Load Cities
        ↓
Select City

Example:

Governorate
[ Dakahlia ▼ ]

City
[ Loading... ]

        ↓

City
[ Mansoura ▼ ]

The frontend should not display invalid cities for the selected governorate.

49. Error Handling

The frontend must handle:

Validation Error
Network Error
Server Error
Duplicate Pharmacy
Duplicate License
File Upload Error
Authentication Error
Session Expired
Unexpected Error

Example:

┌───────────────────────────────────────────┐
│ Unable to submit registration.             │
│                                           │
│ Please review the highlighted fields      │
│ and try again.                             │
│                                           │
│               [ Try Again ]                │
└───────────────────────────────────────────┘
50. Field-Level API Errors

Example backend response:

pharmacyLicenseNumber:
"This pharmacy license is already registered."

Frontend:

Pharmacy License Number *

[ LIC-123456________________ ]

⚠ This pharmacy license is already registered.

Errors should be attached to the correct field whenever possible.

51. Step Navigation

Each step:

[ Back ]                     [ Continue ]

Review:

[ Back ]                [ Submit Registration ]

Logic:

Continue
   ↓
Validate Current Step
   ↓
Valid?
 ┌───────┴───────┐
Yes              No
 ↓                ↓
Next Step       Show Errors
52. Form Data Preservation

Moving backward must not erase data.

Example:

Pharmacy Information
        ↓
Branch Information
        ↓
Medicines
        ↓
Back
        ↓
Branch Information

Previously entered values should remain populated.

53. Responsive Design

The Pharmacy Registration frontend must support:

Desktop
Laptop
Tablet
Mobile

Desktop:

┌──────────────────────────────────────────────┐
│ Pharmacy Information                         │
│                                              │
│ Pharmacy Name       Pharmacy Phone           │
│ [____________]      [____________]            │
│                                              │
└──────────────────────────────────────────────┘

Mobile:

┌────────────────────────────┐
│ Pharmacy Information       │
│                            │
│ Pharmacy Name              │
│ [______________________]   │
│                            │
│ Pharmacy Phone             │
│ [______________________]   │
│                            │
│ [ Continue ]               │
└────────────────────────────┘

No horizontal scrolling should be required on normal mobile screens.

54. Accessibility

The pharmacy registration interface must support:

Semantic HTML
Accessible Labels
Keyboard Navigation
Visible Focus
Accessible Buttons
Accessible Selects
Accessible File Inputs
Readable Errors
Sufficient Contrast

Example:

<label for="pharmacyName">
    Pharmacy Name
</label>

<input
    id="pharmacyName"
    name="pharmacyName"
    type="text"
>
55. Loading States

Examples:

Loading Cities
City

[ Loading cities... ]
Uploading Document
Uploading pharmacy_license.pdf...

██████████████░░░░
Loading Services
Loading pharmacy services...
Submission
Submitting pharmacy registration...
56. Sensitive Data Protection

Sensitive information includes:

National ID
Pharmacist License
Pharmacy License
Commercial Registration
Personal Address
Branch Coordinates
Verification Documents

The frontend must not:

Expose sensitive data in URLs
Log sensitive information to console
Display unnecessary sensitive values
Store documents insecurely
57. Duplicate Submission Protection

The frontend should prevent:

Double Click
      ↓
Two Registration Requests

Recommended:

Click Submit
      ↓
Disable Button
      ↓
Show Loading
      ↓
Send Request
      ↓
Wait for Result

The backend must also provide duplicate protection.

58. Unsaved Changes

Because pharmacy registration contains many fields, leaving the page may result in data loss.

The frontend may provide:

┌─────────────────────────────────────────────┐
│ Leave registration?                         │
│                                             │
│ You have unsaved registration information.  │
│                                             │
│ [ Stay ]                    [ Leave ]       │
└─────────────────────────────────────────────┘
59. API Integration

Pharmacy communication should be separated from the page UI.

Service:

frontend/js/services/pharmacy.service.js

Conceptual example:

const pharmacyService = {

    createRegistration: async (payload) => {
        // Create pharmacy registration
    },

    uploadDocument: async (file) => {
        // Upload verification document
    },

    submitRegistration: async (payload) => {
        // Final registration submission
    },

    getCities: async (governorateId) => {
        // Load cities
    }

};

Architecture:

Pharmacy HTML
      ↓
Pharmacy Registration JS
      ↓
Pharmacy Service
      ↓
ASP.NET Core API
60. Pharmacy Inventory Boundary

The registration module should not contain the complete inventory management functionality.

Registration:

Configure Pharmacy
       ↓
Enable Medicine Availability
       ↓
Complete Registration

Later:

Pharmacy Dashboard
       ↓
Inventory Management
       ↓
Add Medicine
       ↓
Update Quantity
       ↓
Update Availability
       ↓
Remove / Disable Medicine

This keeps registration manageable and prevents a massive onboarding form.

61. Additional Branch Boundary

The first branch is created during registration.

Additional branches belong to the Pharmacy Dashboard.

Registration
     ↓
Primary Branch
     ↓
Verification
     ↓
Pharmacy Dashboard
     ↓
Manage Branches
     ├── Add Branch
     ├── Edit Branch
     ├── Disable Branch
     └── Update Hours
62. Integration With Search & Discovery

After approval, pharmacy information supports:

Patient
   ↓
Search Pharmacy
   ↓
Filter by Location
   ↓
View Pharmacy
   ↓
View Branch
   ↓
View Available Medicines

The registration frontend must therefore collect structured pharmacy and branch information.

63. Integration With Location

Pharmacy branch location supports:

Nearby Pharmacy Search
Distance Calculation
Delivery Radius
Medicine Discovery
Patient Navigation
64. Integration With Provider Verification

The overall relationship:

Pharmacy Registration
        ↓
Documents Submitted
        ↓
Provider Verification
        ↓
Admin Review
        ↓
Approved
        ↓
Pharmacy Activated

The registration module collects the information.

The verification module determines approval.

65. Integration With Authentication
                 Authentication
                       │
             ┌─────────┴──────────┐
             ↓                    ↓
      Account Creation       OTP Verification
             │                    │
             └──────────┬─────────┘
                        ↓
                Pharmacy Registration

Pharmacy registration does not own:

Login
OTP
Password Recovery
Token Management
Global Authorization

These remain part of the shared Authentication/Authorization module.

66. Complete Frontend Flow
                    ┌───────────────┐
                    │ Register      │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Pharmacy Role │
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
                    │ Pharmacist    │
                    │ Information   │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Pharmacy      │
                    │ Information   │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Primary       │
                    │ Branch        │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Medicines &   │
                    │ Services      │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Location &    │
                    │ Hours         │
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Documents     │
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
                    └──────┬────────┘
                           ↓
                    ┌───────────────┐
                    │ Pending       │
                    │ Verification  │
                    └───────────────┘
67. Final Implementation Checklist
[ ] Pharmacy Account Page
[ ] Pharmacist Information Page
[ ] Pharmacy Information Page
[ ] Primary Branch Page
[ ] Medicines & Services Page
[ ] Location & Hours Page
[ ] Documents Page
[ ] Review Page
[ ] Success Page

[ ] Registration Progress Indicator
[ ] Step Navigation
[ ] Form State Management
[ ] Client-side Validation
[ ] Backend Error Mapping
[ ] Loading States
[ ] File Upload UI
[ ] File Validation
[ ] Location Integration
[ ] Governorate/City Dependency
[ ] Operating Hours Validation
[ ] Home Delivery Conditional Fields
[ ] 24-Hour Conditional Fields

[ ] Responsive Design
[ ] Accessibility
[ ] Sensitive Data Protection
[ ] Duplicate Submission Protection
[ ] API Service Integration
[ ] Pending Verification State
68. Ownership Boundary

Alaa owns:

frontend/documentation/02-registration-forms/pharmacy/

and the Pharmacy Registration implementation:

frontend/pages/pharmacy/registration/
frontend/css/pharmacy/pharmacy-registration.css
frontend/js/pages/pharmacy/pharmacy-registration.js
frontend/js/services/pharmacy.service.js

Alaa does not own:

Shared Login
Shared OTP
Password Recovery
Global Authentication
Global Authorization

These belong to Mostafa's shared Authentication & Authorization module.

69. Module Relationship
                    ┌───────────────────────┐
                    │ Authentication        │
                    │ Mostafa               │
                    └───────────┬───────────┘
                                ↓
                    ┌───────────────────────┐
                    │ Pharmacy Registration │
                    │ Alaa                  │
                    └───────────┬───────────┘
                                ↓
                    ┌───────────────────────┐
                    │ Provider Verification │
                    └───────────┬───────────┘
                                ↓
                    ┌───────────────────────┐
                    │ Pharmacy Dashboard    │
                    └───────────┬───────────┘
                                ↓
                    ┌───────────────────────┐
                    │ Medicine Inventory    │
                    │ & Branch Management   │
                    └───────────────────────┘