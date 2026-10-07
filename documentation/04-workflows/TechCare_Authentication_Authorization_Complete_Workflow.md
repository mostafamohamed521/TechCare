# TechCare — Authentication & Authorization Complete Workflow

> **Document Type:** System-wide Authentication & Authorization Specification  
> **Scope:** Shared foundation used by Patient, Doctor, Nurse, Pharmacy, Laboratory, Blood Donor, and Admin modules  
> **Platform:** ASP.NET Core Web API + Frontend + Relational Database  
> **Primary Goal:** One secure identity system, one token/session strategy, centralized authorization, auditable access, and role-aware routing across the entire TechCare platform.

---

## 1. Purpose

Authentication answers:

> **Who is this user?**

Authorization answers:

> **What is this authenticated user allowed to do?**

TechCare must not create a separate login implementation for every module. There must be **one shared Identity/Auth subsystem** that supports multiple role types and module-specific permissions.

The same account can be associated with one or more TechCare role profiles depending on the final business rules. The authentication layer owns identity, credentials, sessions/tokens, OTP, password lifecycle, email/phone verification, account state, and security events. Domain modules own their business data.

Example:

```text
User Account
   |
   +--> Patient Profile
   +--> Doctor Profile
   +--> Nurse Profile
   +--> Pharmacist/Staff Profile
   +--> Laboratory Staff Profile
   +--> Blood Donor Profile
   +--> Admin Profile
```

The system must keep these concerns separate:

```text
Authentication
    ↓
Identity established
    ↓
Authorization evaluated
    ↓
Domain object access checked
    ↓
Business action executed
    ↓
Audit/security event recorded
```

---

# 2. Authentication & Authorization Philosophy

## 2.1 Core Rules

1. Never trust a role sent by the client.
2. Never trust a user ID supplied in a request body when the authenticated identity is available from the token/session.
3. Never authorize a request solely because the user has a valid JWT.
4. Never expose medical or financial data through generic user lookup endpoints.
5. Authentication must be centralized.
6. Authorization must be enforced at API/service level, not only in the frontend.
7. Object-level authorization is mandatory.
8. Sensitive actions should require stronger controls than ordinary reads.
9. Tokens must have controlled lifetime and revocation strategy.
10. Security-sensitive actions must be auditable.
11. Account state must be checked at authorization time or through a reliable revocation mechanism.
12. Failed authentication and suspicious activity must be rate-limited and monitored.
13. Passwords must never be stored in plaintext.
14. OTPs must not be stored in plaintext when persistence is required.
15. Refresh tokens must be rotated/revoked according to the chosen session model.

---

# 3. Supported Actors

## 3.1 Patient

Main capabilities:

- Create patient account
- Verify contact information
- Manage own account
- Manage own patient profile
- Search providers
- Book providers
- View own appointments
- View own prescriptions
- View own lab results
- Order medicines
- Create blood requests
- View own notifications

## 3.2 Doctor

Main capabilities:

- Create doctor account
- Verify professional identity
- Manage doctor profile
- Manage services
- Handle patient requests
- Access authorized patient medical context
- Create diagnosis/treatment/prescription within scope
- Manage appointments
- View earnings/withdrawal information

## 3.3 Nurse

Main capabilities:

- Create nurse account
- Verify professional information
- Manage nursing services
- Handle requests
- Perform nursing visits
- Record nursing notes within allowed scope
- Manage appointments and earnings

## 3.4 Pharmacy Owner / Manager

Main capabilities:

- Manage pharmacy organization
- Manage pharmacy profile
- Manage inventory
- Manage medicine availability/pricing
- Manage orders
- Manage pharmacy staff
- Review prescriptions according to business rules
- Manage earnings

## 3.5 Pharmacist / Pharmacy Staff

Main capabilities depend on assigned permissions.

Examples:

- View orders
- Verify prescription information
- Prepare orders
- Update order states
- Manage permitted stock operations

## 3.6 Laboratory Owner / Manager / Staff

Main capabilities depend on permission level.

Examples:

- Manage laboratory profile
- Configure tests
- Handle bookings
- Assign collectors
- Track samples
- Enter results
- Validate/publish results according to role

## 3.7 Blood Donor

Main capabilities:

- Maintain donor profile
- Manage donation-related preferences
- Receive eligible blood request notifications
- Accept/decline donation opportunities
- View donation history
- View rewards/points

## 3.8 Admin

Administrative permissions are deliberately stronger and must be explicitly granted.

Examples:

- Verify providers
- Manage users
- Suspend/restrict accounts
- Handle complaints
- Review system operations
- Monitor financial workflows
- Manage catalogs
- Review security events
- Manage other admins according to hierarchy

---

# 4. Recommended Identity Architecture

## 4.1 Identity vs Domain Profiles

Recommended separation:

```text
AspNetUsers / Users
      |
      +--> Authentication data
      |      - User ID
      |      - Login credentials
      |      - Contact verification
      |      - Security stamp/version
      |      - Account state
      |
      +--> Role / Permission mapping
      |
      +--> Domain Profile(s)
             - Patient
             - Doctor
             - Nurse
             - Pharmacy Staff
             - Laboratory Staff
             - Donor
             - Admin
```

The **User** record is the identity anchor. Domain records should reference the user ID.

## 4.2 Why this architecture

It avoids duplicated password logic and makes it possible to:

- change password once for the entire account
- disable one account centrally
- audit login activity centrally
- apply common MFA policies
- issue tokens consistently
- use a single refresh-token mechanism
- implement shared authorization policies

---

# 5. User Account Lifecycle

```text
REGISTERED
   ↓
PENDING_VERIFICATION
   ↓
VERIFIED
   ↓
ACTIVE
   ├── RESTRICTED
   ├── SUSPENDED
   └── DEACTIVATED
```

Possible transitions:

```text
PENDING_VERIFICATION → VERIFIED
VERIFIED → ACTIVE
ACTIVE → RESTRICTED
ACTIVE → SUSPENDED
ACTIVE → DEACTIVATED
RESTRICTED → ACTIVE
SUSPENDED → ACTIVE
```

Permanent deletion should not be treated as the default because medical, financial, legal, and audit relationships may need controlled retention. Instead, use deactivation/anonymization policies appropriate to the product and applicable requirements.

---

# 6. Registration Workflow

## 6.1 Generic Registration Flow

```text
Client
  ↓
POST /api/v1/auth/register
  ↓
Validate request
  ↓
Normalize email/phone
  ↓
Check duplicate identity
  ↓
Create user
  ↓
Create security state
  ↓
Create OTP challenge
  ↓
Send OTP
  ↓
Return verification-required response
```

The system should not assume a successful registration means the account is fully active.

## 6.2 Registration Input

Possible common fields:

```json
{
  "firstName": "Mostafa",
  "lastName": "Mohamed",
  "email": "user@example.com",
  "phoneNumber": "01000000000",
  "password": "StrongPasswordHere",
  "confirmPassword": "StrongPasswordHere",
  "requestedRole": "Patient"
}
```

The backend must validate `requestedRole` against allowed self-registration roles.

A client should **not** be allowed to register itself as `Admin` by simply passing:

```json
{"requestedRole":"Admin"}
```

Administrative roles must be provisioned through trusted administrative workflows.

## 6.3 Role-specific Registration

Shared endpoint can be used with role-specific fields, or separate onboarding endpoints can be used while still sharing the same Identity subsystem.

Example:

```text
POST /auth/register/patient
POST /auth/register/doctor
POST /auth/register/nurse
POST /auth/register/donor
```

For pharmacy/laboratory organization onboarding, the identity system may create a user first and then create an organization profile through the relevant module.

---

# 7. Password Policy

Recommended rules:

- Minimum length suitable for modern security policy
- Block commonly compromised passwords
- Confirm-password match on client and server
- Normalize identity values
- Never log passwords
- Never return passwords
- Hash using a strong adaptive password hashing implementation through the Identity framework

Password validation belongs to the backend even if the frontend already validates it.

---

# 8. Email / Phone Verification

## 8.1 Goal

Prove control over the registered contact channel before enabling sensitive account capabilities.

## 8.2 Verification Flow

```text
Register
  ↓
Create OTP challenge
  ↓
Deliver OTP
  ↓
User enters OTP
  ↓
POST /auth/verify-contact
  ↓
Hash/compare OTP
  ↓
Check expiry
  ↓
Check attempt limit
  ↓
Mark contact verified
  ↓
Activate eligible account
```

## 8.3 OTP Security

Recommended fields:

```text
OtpChallenge
- Id
- UserId
- Purpose
- Destination
- CodeHash
- CreatedAt
- ExpiresAt
- Attempts
- MaxAttempts
- ConsumedAt
- RequestedFromIp
- Channel
```

Never store a plaintext OTP in the database.

Possible purposes:

```text
ACCOUNT_VERIFICATION
PASSWORD_RESET
LOGIN_VERIFICATION
MFA_CHALLENGE
CHANGE_PHONE
CHANGE_EMAIL
HIGH_RISK_ACTION
```

OTP must be bound to its purpose. A password-reset OTP must not automatically authorize a high-risk financial action.

---

# 9. OTP Request Workflow

```text
User requests OTP
      ↓
Check account state
      ↓
Check rate limit
      ↓
Invalidate/expire competing challenge when appropriate
      ↓
Create new challenge
      ↓
Store only protected representation
      ↓
Send OTP
      ↓
Return generic success response
```

Response should avoid leaking whether an arbitrary account exists where that would enable user enumeration.

---

# 10. OTP Verification Workflow

```text
Input OTP
   ↓
Find active challenge by challenge ID/purpose
   ↓
Check expiration
   ↓
Check attempts
   ↓
Compare protected value
   ↓
Increment/consume challenge
   ↓
Apply requested action
```

Failure reasons internally may include:

- expired
- invalid
- already consumed
- attempt limit exceeded
- wrong purpose

The public response should avoid exposing unnecessary internal details.

---

# 11. Login Workflow

## 11.1 Standard Login

```text
Client
  ↓
POST /api/v1/auth/login
  ↓
Normalize identifier
  ↓
Locate user
  ↓
Check account state
  ↓
Verify password
  ↓
Check verification requirement
  ↓
Check MFA requirement
  ↓
Create authenticated session/tokens
  ↓
Record login event
  ↓
Return auth result
```

## 11.2 Login Response

Possible response structure:

```json
{
  "accessToken": "...",
  "expiresAt": "...",
  "refreshToken": "...",
  "user": {
    "id": "...",
    "displayName": "..."
  },
  "roles": ["Patient"],
  "permissions": [
    "Appointments.ReadOwn",
    "Appointments.Create",
    "Profile.UpdateOwn"
  ],
  "nextStep": null
}
```

Depending on the architecture, permissions need not all be returned to the client. The server must remain the source of truth.

---

# 12. Account States During Login

| State | Login Result |
|---|---|
| PENDING_VERIFICATION | Require verification |
| ACTIVE | Continue |
| RESTRICTED | Continue with limited permissions if policy allows |
| SUSPENDED | Deny login / session according to policy |
| DEACTIVATED | Deny login |

A provider can also have:

```text
Account = ACTIVE
ProfessionalVerification = PENDING
```

This means the user may access basic account functions but cannot necessarily accept professional jobs.

That distinction is critical.

---

# 13. JWT Access Token

If JWT bearer authentication is selected for the API, the access token should contain only information required for authorization and request processing.

Typical claims:

```text
sub        → User ID
jti        → Token ID
iss        → Issuer
aud        → Audience
exp        → Expiration
iat        → Issued-at
role       → Role claim(s)
```

Optional claim:

```text
security_version
```

The API must validate issuer, audience, lifetime, signature, and other configured validation requirements.

---

# 14. Access Token Lifetime

Access tokens should be relatively short-lived compared with long-lived refresh sessions.

Example conceptual model:

```text
Access Token
   ↓
Short-lived
   ↓
Used for API requests
```

```text
Refresh Token
   ↓
Longer-lived
   ↓
Used only to obtain a new access token
```

Exact durations should be chosen by the team based on the final threat model and deployment requirements rather than hard-coding them into the workflow specification.

---

# 15. Refresh Token Workflow

## 15.1 Basic Flow

```text
Access Token expires
      ↓
Client sends refresh token
      ↓
POST /api/v1/auth/refresh
      ↓
Validate refresh token
      ↓
Check session/account state
      ↓
Rotate refresh token
      ↓
Issue new access token
      ↓
Issue replacement refresh token
```

## 15.2 Refresh Token Rotation

Recommended entity:

```text
RefreshTokenSession
- Id
- UserId
- TokenHash
- CreatedAt
- ExpiresAt
- RevokedAt
- ReplacedById
- CreatedByIp
- UserAgent
- DeviceName
- LastUsedAt
- FamilyId
```

The raw refresh token should not be stored directly if the chosen design supports hashing.

---

# 16. Refresh Token Reuse Detection

Important security scenario:

```text
Refresh Token A issued
       ↓
A used → B issued
       ↓
Attacker later tries A again
```

The system can treat this as suspicious token reuse and revoke the related token family/session depending on the chosen security policy.

This helps reduce the impact of stolen refresh tokens.

---

# 17. Logout Workflow

## 17.1 Device/session logout

```text
Client
  ↓
POST /auth/logout
  ↓
Authenticate current session
  ↓
Revoke refresh session/token
  ↓
Record logout/security event
  ↓
Client removes local auth state
```

Access tokens already issued may remain technically valid until expiry if the system is purely stateless. For high-security requirements, a server-side revocation/version strategy may also be implemented.

## 17.2 Logout all sessions

```text
POST /auth/logout-all
```

Expected behavior:

- revoke active refresh sessions
- increment security/session version where implemented
- require new login on other devices
- create security audit event

---

# 18. Forgot Password Workflow

```text
User selects Forgot Password
       ↓
POST /auth/forgot-password
       ↓
Find account if applicable
       ↓
Apply anti-enumeration response
       ↓
Create reset challenge/token
       ↓
Send OTP/link
```

Never reveal sensitive account existence information through an easily distinguishable response.

---

# 19. Reset Password Workflow

```text
User enters OTP/link + new password
       ↓
POST /auth/reset-password
       ↓
Validate reset challenge
       ↓
Validate new password
       ↓
Set new password hash
       ↓
Invalidate active password-reset challenges
       ↓
Revoke existing sessions according to policy
       ↓
Create security event
       ↓
Return success
```

Recommended behavior: changing a password should invalidate or re-evaluate existing refresh sessions to reduce account-takeover persistence.

---

# 20. Change Password Workflow

For an authenticated user:

```text
POST /auth/change-password
```

Request:

```json
{
  "currentPassword": "...",
  "newPassword": "..."
}
```

Flow:

```text
Authenticate
  ↓
Validate current password
  ↓
Validate new password
  ↓
Update password hash
  ↓
Update security state
  ↓
Revoke other sessions according to policy
  ↓
Record security event
```

A password change should not require the frontend to send the user ID. The server gets the current user from the authenticated identity.

---

# 21. Multi-Factor Authentication (MFA)

MFA should be supported as a security layer, especially for administrative and sensitive professional accounts.

Possible methods:

```text
Password
   +
Second factor
```

Second factor could be an OTP-based mechanism or another secure authenticator approach selected during implementation.

## 21.1 MFA Setup

```text
Authenticated user
   ↓
POST /auth/mfa/setup
   ↓
Create enrollment challenge
   ↓
Confirm factor
   ↓
Enable MFA
```

## 21.2 MFA Login

```text
Password accepted
   ↓
MFA required?
   ↓
Yes
   ↓
Challenge
   ↓
Verify
   ↓
Issue full session/tokens
```

Never issue a fully privileged token before required MFA is completed.

---

# 22. Step-Up Authentication

Some actions deserve additional verification even for an already authenticated user.

Examples:

- Change phone number
- Change email
- Add security factor
- Disable MFA
- Large financial withdrawal
- Change sensitive profile data
- Administrative privilege changes
- Other high-impact security operations

Flow:

```text
Authenticated
   ↓
Request sensitive action
   ↓
Risk/policy check
   ↓
Step-up challenge
   ↓
Success
   ↓
Perform action
```

---

# 23. Role Model

TechCare should distinguish between:

```text
Role
Permission
Resource
Action
Ownership
Organization scope
```

A role is a collection of permissions.

Example:

```text
Doctor
 ├── Profile.ReadOwn
 ├── Profile.UpdateOwn
 ├── Appointments.ReadOwn
 ├── Appointments.Accept
 ├── Consultation.Create
 ├── Prescription.Create
 └── Earnings.ReadOwn
```

A permission is a specific capability:

```text
Appointments.Accept
```

---

# 24. RBAC Design

Recommended tables/entities:

```text
User
Role
UserRole
Permission
RolePermission
```

Potential additional tables:

```text
Organization
OrganizationMember
OrganizationMemberRole
```

This enables pharmacy and laboratory staff to have organization-scoped permissions.

---

# 25. Permission Naming Convention

Use predictable permission names.

Examples:

```text
Users.Read
Users.ReadSensitive
Users.Suspend
Users.Reactivate
Users.AssignRole

Doctors.Read
Doctors.Verify
Doctors.Suspend

Appointments.ReadOwn
Appointments.ReadAssigned
Appointments.Create
Appointments.Accept
Appointments.Reject
Appointments.Cancel

Prescriptions.ReadOwn
Prescriptions.Create
Prescriptions.Verify

Inventory.Read
Inventory.Update
Orders.ReadAssigned
Orders.UpdateStatus

LabResults.ReadOwn
LabResults.Create
LabResults.Validate
LabResults.Publish

BloodRequests.Create
BloodRequests.ReadOwn
Donations.Accept
Donations.Record

Finance.ReadOwn
Withdrawals.Create
Withdrawals.Approve
```

The exact list should evolve with the modules, but naming should remain consistent.

---

# 26. Policy-Based Authorization

Instead of scattering role checks everywhere:

```csharp
if (userRole == "Doctor")
{
    ...
}
```

prefer policy-based authorization where practical:

```text
[Authorize(Policy = "Prescription.Create")]
```

The policy checks the permission claim/store and then object/business rules verify that the specific resource belongs to or is accessible by the current actor.

---

# 27. Object-Level Authorization

This is one of the most important parts of TechCare.

Example:

```text
GET /api/v1/appointments/123
```

Authentication tells us:

```text
User = 57
```

Role tells us:

```text
Role = Doctor
```

But neither is enough.

The server must verify:

```text
Appointment 123
   belongs to / is assigned to / is otherwise accessible by
   Doctor 57
```

Otherwise the API may suffer from IDOR/BOLA vulnerabilities.

---

# 28. Ownership Rules

## Patient-owned resources

Examples:

- own profile
- own addresses
- own appointments
- own orders
- own prescriptions
- own lab results
- own notifications
- own blood requests

## Doctor-owned resources

Examples:

- own profile
- own services
- own schedule
- own accepted appointments
- own financial records

## Nurse-owned resources

Same concept with nursing scope.

## Pharmacy-scoped resources

Access depends on:

```text
User
   ↓
Organization Membership
   ↓
Pharmacy ID
   ↓
Permission
```

## Laboratory-scoped resources

Same pattern:

```text
User
   ↓
Lab Membership
   ↓
Laboratory ID
   ↓
Permission
```

---

# 29. Admin Authorization Model

Do not make every administrator equal automatically.

Recommended hierarchy concept:

```text
SuperAdmin
   ↓
PlatformAdmin
   ↓
OperationsAdmin
   ↓
SupportAdmin
   ↓
VerificationAdmin
```

Exact roles are project-dependent.

Important rule:

> An admin must not gain permissions beyond what their assigned admin role grants.

Sensitive operations should also be protected against self-escalation.

---

# 30. Least Privilege

Every user and service should receive only the minimum permissions needed.

Examples:

```text
Pharmacy Staff
✓ Read assigned orders
✓ Update preparation state
✗ Manage pharmacy owner
✗ Withdraw money
✗ Suspend users
```

```text
Verification Admin
✓ Review professional documents
✓ Approve/reject verification
✗ Modify patient medical history
```

```text
Nurse
✓ Read authorized nursing-related patient information
✗ Create doctor diagnosis
✗ Create doctor prescription
```

---

# 31. Separation of Duties

High-impact actions should be separated when appropriate.

Example:

```text
Admin A → reviews withdrawal
Admin B → approves withdrawal
```

or:

```text
Provider → submits documents
Verification Admin → approves provider
```

The system should avoid designs where the same person can silently create, approve, and settle every sensitive transaction.

---

# 32. Authorization Decision Flow

Every protected request should conceptually pass through:

```text
1. Is request authenticated?
          ↓ yes
2. Is account active/allowed?
          ↓ yes
3. Does user have required permission?
          ↓ yes
4. Does user have access to this resource?
          ↓ yes
5. Is action allowed in current business state?
          ↓ yes
6. Is extra verification required?
          ↓ no
7. Execute action
          ↓
8. Audit when required
```

---

# 33. Business-State Authorization

Having a permission does not mean an action is always allowed.

Example:

```text
Doctor has Prescription.Create
```

Still, creation may be rejected when:

- appointment is not eligible
- consultation is closed
- doctor is suspended
- patient relationship is invalid
- required fields are incomplete
- prescription is already finalized under an immutable workflow

Authorization therefore has two levels:

```text
Permission authorization
+
Business-state authorization
```

---

# 34. Role-Aware Routing on Frontend

After login:

```text
User authenticated
   ↓
Fetch identity/profile summary
   ↓
Determine effective role/context
   ↓
Route to dashboard
```

Example:

```text
Patient   → /patient/dashboard
Doctor    → /doctor/dashboard
Nurse     → /nurse/dashboard
Pharmacist→ /pharmacy/dashboard
Lab Staff → /laboratory/dashboard
Donor     → /donor/dashboard
Admin     → /admin/dashboard
```

Frontend routing is for UX only. API authorization remains mandatory.

---

# 35. Multi-Role Account

The architecture should be able to support an account associated with multiple legitimate capabilities if the final product permits it.

Example:

```text
User 100
 ├── Patient
 └── Blood Donor
```

The user can select an operational context when needed.

The backend must still evaluate each requested action against the effective role/permission context.

Do not use a client-controlled role query string as the sole security mechanism.

---

# 36. Organization Context

Pharmacy and laboratory accounts often require organization-scoped authorization.

Example:

```text
User 77
   ↓
Pharmacy Staff
   ↓
Pharmacy 15
```

Request:

```text
PATCH /pharmacies/15/orders/900
```

The server must verify both:

```text
User 77 ∈ Pharmacy 15
AND
User 77 has Orders.UpdateStatus
```

Checking only the second condition is insufficient.

---

# 37. Organization Membership Lifecycle

```text
INVITED
   ↓
ACCEPTED
   ↓
ACTIVE
   ├── SUSPENDED
   └── REMOVED
```

Invitation flow:

```text
Owner creates invitation
    ↓
Invitation token
    ↓
Invitee verifies identity
    ↓
Accept invitation
    ↓
Membership activated
```

An invite should be time-limited and single-use.

---

# 38. Permission Update Workflow

```text
Authorized Admin
   ↓
Select user/membership
   ↓
Select allowed role/permission
   ↓
Validate admin authority
   ↓
Write change
   ↓
Update security version/cache
   ↓
Invalidate affected sessions if required
   ↓
Audit event
```

Never allow users to assign themselves stronger roles.

---

# 39. Privilege Escalation Protection

The API must explicitly block patterns such as:

```text
User edits own role → Admin
User edits own permissions → All permissions
Patient edits organization ID → Trusted Pharmacy
```

Client-supplied role or permission fields must never directly override server-side authorization data.

---

# 40. Account Suspension Workflow

Suspension can be triggered by:

- Admin action
- security investigation
- repeated abuse
- policy violation
- verification failure
- system decision under predefined rules

Flow:

```text
Authorized Admin
   ↓
Select account
   ↓
Enter reason
   ↓
Confirm
   ↓
Set state = SUSPENDED
   ↓
Revoke/invalidate active sessions according to policy
   ↓
Notify user
   ↓
Audit
```

Suspension reason should be protected data and shown only to authorized parties.

---

# 41. Account Reactivation

```text
Admin
  ↓
Review suspension
  ↓
Confirm reactivation
  ↓
State = ACTIVE
  ↓
Keep/restore only valid roles and permissions
  ↓
Require new login if security policy dictates
  ↓
Audit
```

Reactivation must not silently restore a role that was separately revoked.

---

# 42. Failed Login Protection

Track security metadata such as:

```text
Failed login count
Last failed login time
Last successful login time
Lockout state
```

Use progressive defenses:

```text
Repeated failures
   ↓
Rate limiting
   ↓
Temporary lockout or challenge
   ↓
Security event
```

Do not make lockout behavior so predictable that it becomes an account-lockout attack vector against arbitrary users.

---

# 43. Rate Limiting

Rate limit sensitive endpoints such as:

```text
/login
/register
/request-otp
/verify-otp
/forgot-password
/reset-password
/refresh
```

Rate limiting can be applied by multiple dimensions where appropriate:

- IP
- account/identifier
- device/session
- endpoint

The implementation should avoid allowing an attacker to bypass limits simply by changing one dimension.

---

# 44. Anti-Enumeration Strategy

A malicious user should not easily discover:

- whether an email belongs to a user
- whether a phone is registered
- whether a specific privileged account exists

For sensitive recovery operations, use generic responses such as:

```text
If the account is eligible, recovery instructions will be sent.
```

Actual side effects depend on server-side account state.

---

# 45. Session Management

A session record may contain:

```text
SessionId
UserId
RefreshTokenId
CreatedAt
LastSeenAt
ExpiresAt
RevokedAt
IP
UserAgent
DeviceLabel
```

Features:

```text
My Devices / Active Sessions
```

User can:

- view active sessions where supported
- revoke one session
- revoke all sessions

Sensitive metadata should not expose more information than necessary.

---

# 46. Device Management

Optional but useful:

```text
Trusted Device
Recent Device
Unknown Device
```

A new device may trigger:

- email/security alert
- MFA
- step-up authentication
- session review

Do not treat browser local storage as a cryptographic proof of device identity.

---

# 47. Token Storage on Frontend

The exact storage strategy depends on frontend architecture.

For browser-based applications, a safer session design often uses secure, HTTP-only cookies for long-lived session credentials, with appropriate CSRF protections, rather than exposing refresh tokens to JavaScript.

If bearer tokens are kept client-side, the threat model must explicitly consider XSS and token theft.

The key principle:

> Never choose token storage only because it is convenient.

---

# 48. HTTPS Requirement

Authentication traffic must be protected in transit in deployed environments.

The application should not send:

- passwords
- OTPs
- reset tokens
- refresh tokens
- sensitive medical data

over plaintext production HTTP.

---

# 49. CORS

CORS must be explicitly configured for trusted frontend origins.

Avoid permissive production configuration such as:

```text
AllowAnyOrigin + credentials
```

When cookies/credentials are used, configure origins and credential handling correctly.

---

# 50. CSRF Protection

CSRF must be considered when authentication is cookie-based.

State-changing operations should use appropriate CSRF defenses.

Examples:

```text
POST
PUT
PATCH
DELETE
```

JWTs sent in authorization headers have a different CSRF profile, but they remain exposed to token theft through XSS if handled unsafely.

---

# 51. Input Validation

Every authentication endpoint requires server-side validation.

Validate:

- required fields
- length
- email format
- phone format
- OTP format
- password policy
- token structure
- allowed enum values

Client-side validation improves UX but is never the security boundary.

---

# 52. User Identity Resolution

The backend should provide a reliable helper/service such as:

```text
CurrentUserService
```

Responsibilities:

- read authenticated user ID
- read roles/claims
- expose authentication status
- expose organization memberships where necessary

Controllers should not repeatedly parse tokens manually.

---

# 53. Authorization Service

Recommended abstraction:

```text
IAuthorizationService
```

Conceptually:

```text
AuthorizeAsync(
    user,
    resource,
    permission
)
```

It can combine:

```text
Permission
Role
Ownership
Organization membership
Business state
```

---

# 54. Permission Evaluation Example

Example request:

```text
POST /api/v1/prescriptions
```

Evaluation:

```text
Authenticated?
   ↓ yes
Doctor?
   ↓ yes
Prescription.Create?
   ↓ yes
Doctor owns appointment?
   ↓ yes
Appointment in eligible state?
   ↓ yes
Patient relationship valid?
   ↓ yes
Create prescription
```

A frontend button being hidden is not authorization.

---

# 55. Medical Data Authorization

TechCare contains sensitive healthcare information.

Authorization must distinguish between:

```text
Patient's own medical record
Doctor's authorized clinical access
Nurse's limited care-related access
Laboratory result access
Pharmacy prescription access
Admin operational access
```

An admin dashboard should not automatically imply unrestricted clinical access to all medical details.

Use minimal necessary access.

---

# 56. Break-Glass / Exceptional Access

Future/high-security feature:

```text
Break-glass access
```

When emergency access is justified, the system may require:

- reason
- explicit confirmation
- additional audit trail
- restricted duration
- post-event review

This should not exist as a silent universal admin bypass.

---

# 57. Password Reset Security Rules

Reset links/codes should:

- expire
- be single-use
- be bound to a specific purpose
- have replay protections
- be rate-limited
- not reveal account existence

After successful reset:

```text
Old reset challenge → invalid
Old reset links → invalid
Existing sessions → revoke/re-evaluate
```

---

# 58. Change Email Workflow

```text
Authenticated user
   ↓
Request new email
   ↓
Step-up authentication if required
   ↓
Create verification challenge
   ↓
Verify new email
   ↓
Update email
   ↓
Audit event
   ↓
Security notification
```

Do not immediately trust an unverified new email as the primary recovery channel.

---

# 59. Change Phone Workflow

Same architecture:

```text
Authenticate
   ↓
Request change
   ↓
Strong verification
   ↓
OTP to new destination
   ↓
Confirm
   ↓
Update
   ↓
Audit
```

---

# 60. Account Deactivation

User-initiated deactivation:

```text
Authenticated
   ↓
Confirm action
   ↓
Optional step-up authentication
   ↓
State = DEACTIVATED
   ↓
Revoke sessions
   ↓
Audit
```

The application must separately handle:

- pending appointments
- orders
- financial balances
- professional verification
- domain-specific obligations

The auth module should not independently delete those records.

---

# 61. Admin Login Workflow

Admin authentication should use stronger security controls.

Conceptual flow:

```text
Admin login
   ↓
Password
   ↓
MFA/step-up when required
   ↓
Check Admin role
   ↓
Check Admin account state
   ↓
Issue session
   ↓
Record security event
```

An account being a normal user does not automatically grant admin access.

---

# 62. Admin Session Policy

For admin accounts:

- shorter idle/session rules may be appropriate
- MFA should be strongly considered or required by project policy
- suspicious login detection should be stricter
- sensitive operations should generate detailed audit events

Exact durations and controls must be finalized during deployment/security review.

---

# 63. Security Notifications

Potential events:

```text
New login
Password changed
Email changed
Phone changed
MFA enabled
MFA disabled
Session revoked
Account suspended
Role changed
Permission changed
Provider verification changed
```

The notification itself must not expose sensitive internals.

---

# 64. Audit Logging

Authentication/security events should include structured audit data.

Recommended entity:

```text
SecurityAuditLog
- Id
- UserId nullable
- ActorUserId nullable
- EventType
- Timestamp
- IPAddress
- UserAgent
- CorrelationId
- Success
- ReasonCode
- TargetType
- TargetId
- Metadata
```

Examples:

```text
LOGIN_SUCCESS
LOGIN_FAILURE
OTP_REQUESTED
OTP_FAILED
OTP_VERIFIED
PASSWORD_CHANGED
PASSWORD_RESET
REFRESH_TOKEN_ROTATED
REFRESH_TOKEN_REUSE_DETECTED
LOGOUT
LOGOUT_ALL
ROLE_ASSIGNED
ROLE_REMOVED
PERMISSION_CHANGED
ACCOUNT_SUSPENDED
ACCOUNT_REACTIVATED
```

---

# 65. Audit Integrity

Audit records should be protected from ordinary modification/deletion.

Normal application users must not be able to:

```text
UPDATE SecurityAuditLog
DELETE SecurityAuditLog
```

Administrative access to audit logs should also be explicitly permissioned and itself audited.

---

# 66. Correlation IDs

Every API request should ideally carry a correlation ID.

Example:

```text
CorrelationId = 8f1...
```

Use it across:

```text
Request
 → Authentication
 → Authorization
 → Service
 → Database
 → Notification
 → Audit
```

This greatly improves troubleshooting and incident investigation.

---

# 67. Authentication API Catalog

Suggested endpoints:

```text
POST /api/v1/auth/register
POST /api/v1/auth/verify-contact
POST /api/v1/auth/resend-otp
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/logout-all
POST /api/v1/auth/forgot-password
POST /api/v1/auth/verify-reset-code
POST /api/v1/auth/reset-password
POST /api/v1/auth/change-password
POST /api/v1/auth/change-email
POST /api/v1/auth/change-phone
GET  /api/v1/auth/me
GET  /api/v1/auth/sessions
DELETE /api/v1/auth/sessions/{sessionId}
POST /api/v1/auth/mfa/setup
POST /api/v1/auth/mfa/verify
POST /api/v1/auth/mfa/disable
```

Exact route names may be adjusted during API design.

---

# 68. Authorization API / Infrastructure

Authorization does not necessarily need dedicated public endpoints for every permission check.

Most authorization is internal middleware/policy logic.

Admin-only endpoints may include:

```text
GET /api/v1/admin/users/{id}/roles
PUT /api/v1/admin/users/{id}/roles
GET /api/v1/admin/users/{id}/permissions
PUT /api/v1/admin/users/{id}/permissions
POST /api/v1/admin/users/{id}/suspend
POST /api/v1/admin/users/{id}/reactivate
GET /api/v1/admin/security/audit-logs
GET /api/v1/admin/security/events
```

These endpoints require their own object and privilege checks.

---

# 69. `GET /auth/me`

Recommended use:

```text
GET /api/v1/auth/me
```

Purpose:

- validate that frontend session is usable
- retrieve current identity summary
- retrieve effective roles
- retrieve basic account state
- retrieve enabled profile contexts

Example:

```json
{
  "id": "user-id",
  "displayName": "User Name",
  "emailVerified": true,
  "roles": ["Patient", "BloodDonor"],
  "accountStatus": "ACTIVE",
  "profiles": {
    "patient": true,
    "doctor": false,
    "donor": true
  }
}
```

Avoid returning unnecessary sensitive data.

---

# 70. DTO Separation

Never expose Identity entities directly from APIs.

Use dedicated DTOs:

```text
LoginRequest
LoginResponse
RegisterRequest
VerifyOtpRequest
RefreshTokenRequest
ChangePasswordRequest
ForgotPasswordRequest
ResetPasswordRequest
CurrentUserResponse
SessionResponse
```

This limits accidental data exposure and makes API contracts clearer.

---

# 71. Database Design

Recommended core entities:

```text
User
Role
Permission
UserRole
RolePermission
OtpChallenge
RefreshTokenSession
SecurityAuditLog
LoginAttempt
OrganizationMembership
Invitation
SecurityVersion / equivalent
```

Optional:

```text
MfaMethod
TrustedDevice
AccountRecoveryChallenge
SecurityNotification
```

---

# 72. User Entity — Conceptual Fields

```text
User
----
Id
FirstName
LastName
Email
NormalizedEmail
PhoneNumber
NormalizedPhoneNumber
PasswordHash
EmailVerifiedAt
PhoneVerifiedAt
AccountStatus
SecurityStamp / SecurityVersion
LastLoginAt
CreatedAt
UpdatedAt
DeactivatedAt
```

Identity-specific implementation can use ASP.NET Core Identity's built-in fields rather than duplicating them manually.

---

# 73. Role Entity

```text
Role
----
Id
Name
NormalizedName
Description
IsSystemRole
CreatedAt
```

Examples:

```text
Patient
Doctor
Nurse
PharmacyOwner
Pharmacist
PharmacyStaff
LaboratoryOwner
LaboratoryStaff
BloodDonor
Admin
SuperAdmin
```

---

# 74. Permission Entity

```text
Permission
----------
Id
Code
Name
Description
Module
IsSensitive
CreatedAt
```

Examples:

```text
Users.Read
Users.Suspend
Appointments.Accept
Prescriptions.Create
LabResults.Publish
Withdrawals.Approve
```

---

# 75. Organization Membership Entity

```text
OrganizationMembership
----------------------
Id
UserId
OrganizationId
OrganizationType
RoleId / MembershipRole
Status
InvitedBy
JoinedAt
RemovedAt
CreatedAt
```

Examples:

```text
Pharmacy 12 + User 44 + Pharmacist
Laboratory 5 + User 77 + Collector
```

---

# 76. Password Security

Do:

```text
Password
   ↓
Adaptive password hashing
   ↓
Database
```

Do not:

```text
Password
   ↓
Plaintext DB
```

Do not:

```text
Password
   ↓
Custom weak hashing implementation
```

Use the framework's mature Identity password infrastructure unless there is a strong architectural reason to replace it.

---

# 77. Token Security

Recommended server behavior:

- sign JWT access tokens using strong configured keys
- rotate signing secrets/keys operationally when needed
- validate issuer and audience
- reject expired tokens
- reject malformed tokens
- protect refresh token records
- revoke sessions after high-risk credential changes

Do not place secrets in source code.

---

# 78. Secret Management

Sensitive values should come from secure configuration mechanisms.

Examples:

```text
JWT signing keys
Email provider credentials
SMS provider credentials
Database credentials
Encryption keys
```

Do not commit secrets into:

```text
appsettings.json
Git repository
Docker image
Frontend source code
```

Use environment/configuration secret management appropriate to the deployment.

---

# 79. Authorization Middleware Pipeline

Conceptual ASP.NET Core pipeline:

```text
Request
  ↓
HTTPS / Proxy handling
  ↓
Rate Limiting
  ↓
CORS
  ↓
Authentication
  ↓
Authorization
  ↓
Controller / Endpoint
  ↓
Application Service
  ↓
Domain Rules
  ↓
Persistence
```

The exact middleware order depends on the application's configuration, but authentication must occur before protected authorization decisions.

---

# 80. ASP.NET Core Conceptual Configuration

Typical architecture:

```text
ASP.NET Core Identity
      +
JWT Bearer Authentication
      +
Authorization Policies
      +
Custom Authorization Handlers
      +
Resource-based checks
```

Possible services:

```text
IIdentityService
ITokenService
IOtpService
ISessionService
IAuthorizationService
IPermissionService
ISecurityAuditService
```

---

# 81. Example Authorization Components

```text
CurrentUserService
AuthorizationPolicyProvider
PermissionAuthorizationHandler
ResourceAuthorizationHandler
OrganizationAccessService
SecurityAuditService
```

Keep authorization logic reusable rather than duplicating checks inside every controller.

---

# 82. Resource-Based Authorization Example

Example resource:

```text
Appointment
```

Possible rules:

```text
Patient → own appointment
Doctor  → assigned appointment
Nurse   → assigned nursing appointment
Admin   → operational access according to permission
```

The resource handler should compare the authenticated identity with the actual resource relationships.

---

# 83. Authorization Failure Responses

Common categories:

```text
401 Unauthorized
→ No valid authentication
```

```text
403 Forbidden
→ Authenticated but not allowed
```

```text
404 Not Found
→ Sometimes preferable when exposing the resource existence would itself be sensitive
```

The final choice must be consistent with the product's security/error-handling strategy.

---

# 84. Avoid Information Leakage

Do not return responses such as:

```text
"User 55 belongs to a pharmacy and you are not allowed."
```

when the caller should not even know the resource exists.

Generic responses can be safer:

```text
Resource not available.
```

---

# 85. Frontend Protected Routes

Conceptual route guard:

```text
ProtectedRoute
   ↓
Is authenticated?
   ├── No → /login
   └── Yes
        ↓
      Required role/permission?
        ├── No → Forbidden page
        └── Yes → Render page
```

Again: this improves user experience but is not the security boundary.

---

# 86. Frontend Auth State

Typical state:

```text
AuthState
- user
- isAuthenticated
- accountStatus
- roles
- loading
- session expiry
```

The frontend should gracefully handle:

```text
401
→ refresh/re-login logic

403
→ forbidden UI

Session expired
→ clear auth state and redirect to login
```

Avoid infinite refresh loops.

---

# 87. Refresh Loop Protection

Bad behavior:

```text
401
 ↓
refresh
 ↓
401
 ↓
refresh
 ↓
401
 ↓
∞
```

Instead:

```text
401
 ↓
Attempt refresh once
 ↓
If failed
 ↓
Clear session
 ↓
Redirect to login
```

Concurrent requests should be coordinated so multiple refresh requests do not create token-rotation races.

---

# 88. Concurrent Refresh Handling

Possible frontend approach:

```text
Request A → 401
Request B → 401
Request C → 401
        ↓
Single refresh operation
        ↓
New token
        ↓
Retry queued requests
```

This reduces refresh-token race conditions.

---

# 89. Login Concurrency

The backend must tolerate simultaneous login attempts safely.

Examples:

- duplicate OTP requests
- multiple password attempts
- simultaneous session creation
- repeated retry after a timeout

Operations should be idempotent where appropriate.

---

# 90. Registration Concurrency

Potential race:

```text
Request A checks email → available
Request B checks email → available
A creates
B creates
```

Database uniqueness constraints must be the final protection.

Required unique constraints should be applied to normalized identity fields where appropriate.

---

# 91. OTP Concurrency

Potential race:

```text
Two verify requests arrive together
```

The system must ensure the same one-time challenge cannot be consumed twice.

Use transactional/concurrency protection around the consume operation.

---

# 92. Refresh Token Concurrency

Potential race:

```text
Request A refreshes token
Request B refreshes same token at nearly same time
```

The backend must define deterministic behavior using rotation/concurrency controls so a refresh token cannot be accepted repeatedly in an unsafe way.

---

# 93. Security Event Classification

Classify events into:

```text
INFO
WARNING
SECURITY_ALERT
CRITICAL
```

Examples:

```text
INFO → successful login
WARNING → repeated login failures
SECURITY_ALERT → refresh-token reuse
CRITICAL → suspicious privilege escalation attempt
```

---

# 94. Security Monitoring Dashboard

Admin/security dashboard may show:

```text
Failed logins
OTP abuse
Token reuse detections
Suspended accounts
Recent admin privilege changes
Unusual login activity
Recent high-risk security actions
```

Raw sensitive data should be minimized in dashboard responses.

---

# 95. Account Recovery

Supported recovery path:

```text
Forgot password
   ↓
Identity/contact verification
   ↓
Recovery challenge
   ↓
New password
   ↓
Session revocation
   ↓
Security notification
```

High-risk recovery should not rely on a single weak factor.

---

# 96. Lost Phone / Lost Device

User support flow:

```text
User cannot access device
   ↓
Recover account through approved recovery channel
   ↓
Revoke lost sessions
   ↓
Re-enroll security factor
   ↓
Issue new session
```

Admin/support staff should not ask users to send passwords or OTPs through insecure channels.

---

# 97. Support/Admin Impersonation

Avoid silent impersonation.

If support needs temporary access for troubleshooting, use a controlled workflow with:

- explicit permission
- target user
- reason
- limited duration
- audit event
- restricted capabilities

Do not silently replace the user's identity in the audit trail.

---

# 98. Sensitive Action Audit

At minimum, audit:

```text
Role changes
Permission changes
Account suspension
Account reactivation
Password changes/reset
MFA changes
Admin login
Provider verification decisions
Financial approval-related auth events
```

---

# 99. Module Integration Rules

## Patient Module

Auth provides:

```text
Current User
Patient role
Permissions
Session
```

Patient module decides:

```text
Can access own medical profile?
Can create booking?
Can view own lab result?
```

## Doctor Module

Auth provides:

```text
Doctor identity + permissions
```

Doctor module decides:

```text
Is this appointment assigned to this doctor?
Is the provider verified?
Can doctor create this prescription?
```

## Nurse Module

Auth provides identity and role.

Nurse module decides resource and business scope.

## Pharmacy Module

Auth provides identity and membership.

Pharmacy module decides pharmacy/organization ownership and operational permissions.

## Laboratory Module

Auth provides identity and membership.

Laboratory module decides sample/result access based on role and relationship.

## Blood Donor Module

Auth provides authenticated donor identity.

Blood module decides donor eligibility, matching visibility, donation-state rules, and donor history access.

## Admin Module

Auth provides admin identity and permission.

Admin module decides whether a particular operational action is allowed and logs it.

---

# 100. Authentication ↔ Patient Integration Example

```text
Patient registers
   ↓
Identity created
   ↓
OTP verification
   ↓
Patient role assigned
   ↓
Patient profile created
   ↓
Login
   ↓
Access token/session
   ↓
Patient dashboard
```

---

# 101. Authentication ↔ Doctor Integration Example

```text
Doctor registers
   ↓
Identity created
   ↓
Verification
   ↓
Doctor role
   ↓
Doctor profile
   ↓
Professional verification = PENDING
   ↓
Admin approves
   ↓
Provider capabilities enabled
```

Authentication state and professional verification state should remain separate.

---

# 102. Authentication ↔ Pharmacy Integration Example

```text
Pharmacy owner account
   ↓
Identity
   ↓
Organization created
   ↓
Pharmacy verification pending
   ↓
Admin verifies
   ↓
Owner gets approved organization permissions
```

---

# 103. Authentication ↔ Laboratory Integration Example

```text
Lab owner/staff identity
   ↓
Organization membership
   ↓
Role/permission
   ↓
Lab resource access
```

---

# 104. Authentication ↔ Blood Donor Integration Example

```text
User identity
   ↓
Blood Donor role/profile
   ↓
Donor preferences
   ↓
Eligible request notifications
```

The donor module should not duplicate password or token handling.

---

# 105. Authorization Matrix — High Level

| Feature | Patient | Doctor | Nurse | Pharmacy Staff | Lab Staff | Donor | Admin |
|---|---:|---:|---:|---:|---:|---:|---:|
| Login | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Manage own auth profile | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Manage own password | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Patient medical data | Own | Authorized | Limited/Authorized | Minimal/Needed | Relevant Results | Own donation profile | Explicitly limited/permissioned |
| Provider verification | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Manage own provider profile | N/A | ✓ | ✓ | ✓ | ✓ | N/A | According to admin scope |
| Manage pharmacy inventory | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | Catalog/system scope |
| Manage lab results | ✗ | Authorized read | ✗ | ✗ | ✓ | ✗ | Operational scope |
| Blood requests | ✓ | Possible read/operational access | Possible read | ✗ | ✗ | View eligible opportunities | Admin monitoring |
| Role assignment | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | Restricted admin |
| Security audit | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | Authorized admins |

This table is intentionally high-level. The actual implementation must be permission- and resource-based.

---

# 106. Detailed Permission Groups

## Identity Permissions

```text
Profile.ReadOwn
Profile.UpdateOwn
Credentials.ChangeOwn
Sessions.ReadOwn
Sessions.RevokeOwn
```

## Administrative Identity Permissions

```text
Users.Read
Users.ReadSensitive
Users.Suspend
Users.Reactivate
Users.AssignRole
Users.RemoveRole
Permissions.Read
Permissions.Update
```

## Security Operations

```text
SecurityAudit.Read
SecurityEvents.Read
SecuritySessions.Revoke
```

---

# 107. Authorization Anti-Patterns

## Anti-pattern 1 — Frontend-only authorization

```text
Hide Admin button
```

This provides no backend security.

## Anti-pattern 2 — Role in request body

```json
{"role":"Admin"}
```

Never trust it.

## Anti-pattern 3 — Generic user-ID authorization

```text
GET /users/{id}
```

without field-level and role-aware restrictions.

## Anti-pattern 4 — Controller-only checks

Business services must still enforce important rules so another endpoint cannot bypass them.

## Anti-pattern 5 — Admin wildcard access

Avoid:

```text
Admin = automatically allowed to everything
```

Especially for medical data.

---

# 108. Common Security Vulnerabilities to Test

The team should explicitly test:

```text
Broken Access Control
BOLA / IDOR
Privilege Escalation
JWT validation mistakes
Refresh token replay
OTP replay
OTP brute-force
Credential stuffing
Rate-limit bypass
Session fixation
CSRF where applicable
XSS token theft scenarios
Account enumeration
Password reset abuse
Mass assignment
Sensitive data exposure
```

---

# 109. Mass Assignment Protection

Dangerous DTO:

```json
{
  "firstName": "User",
  "role": "Admin",
  "accountStatus": "ACTIVE",
  "isVerified": true
}
```

Never bind arbitrary identity/admin fields directly from user-controlled payloads.

Use explicit command models.

---

# 110. Claims vs Database Permissions

There are two common approaches:

```text
JWT contains roles/permissions
```

or:

```text
JWT contains identity
   ↓
Server resolves current permissions
```

A hybrid can also be used.

For highly dynamic permissions, avoid relying on long-lived tokens containing stale privileged claims without a revocation/version strategy.

---

# 111. Security Version

A user-level security version can be used conceptually:

```text
User.SecurityVersion = 8
```

When a high-risk event occurs:

```text
SecurityVersion 8 → 9
```

Tokens/sessions carrying version 8 can be rejected if the server checks the current version.

Use this carefully to balance security and scalability.

---

# 112. Session Revocation Triggers

Possible triggers:

```text
Password changed
Password reset
MFA disabled
Suspicious token reuse
Account suspended
Admin force logout
Security incident
```

The exact revocation scope can be:

```text
Current session
All sessions on device
All sessions for account
Token family
```

---

# 113. Login Audit Example

```json
{
  "event": "LOGIN_SUCCESS",
  "userId": "123",
  "timestamp": "...",
  "ipAddress": "...",
  "userAgent": "...",
  "correlationId": "...",
  "success": true
}
```

Do not store unnecessary secrets or raw authentication credentials in logs.

---

# 114. Password Reset Audit Example

```json
{
  "event": "PASSWORD_RESET",
  "userId": "123",
  "timestamp": "...",
  "correlationId": "...",
  "success": true
}
```

The audit should not contain the password, OTP, or reset secret.

---

# 115. Privilege Change Audit Example

```json
{
  "event": "ROLE_ASSIGNED",
  "actorUserId": "10",
  "targetUserId": "77",
  "role": "VerificationAdmin",
  "reason": "Operational role assignment",
  "timestamp": "..."
}
```

---

# 116. Full Registration Scenario — Patient

```text
1. Patient opens registration page
2. Enters identity data
3. Backend validates
4. Backend normalizes email/phone
5. Backend checks duplicates
6. User record created
7. Patient role assigned
8. OTP challenge created
9. OTP sent
10. Patient enters OTP
11. OTP verified
12. Contact marked verified
13. Account activated according to policy
14. Session/tokens created if product policy allows
15. Patient profile onboarding begins
16. Audit event recorded
```

---

# 117. Full Registration Scenario — Doctor

```text
1. Doctor registers
2. Identity created
3. Contact verified
4. Doctor role/profile created
5. Professional documents uploaded through doctor module
6. Verification remains pending
7. Basic account login allowed according to policy
8. Restricted provider capabilities until approval
9. Admin approves
10. Provider capabilities enabled
```

---

# 118. Full Login Scenario

```text
Open Login
   ↓
Enter email/phone + password
   ↓
Server validates credentials
   ↓
Account state checked
   ↓
Role/permissions resolved
   ↓
MFA check
   ↓
Session/tokens created
   ↓
Login audit event
   ↓
Frontend loads /me
   ↓
Route to correct dashboard
```

---

# 119. Full Forgot Password Scenario

```text
Forgot Password
   ↓
Enter email/phone
   ↓
Generic response
   ↓
Recovery challenge sent when eligible
   ↓
Enter code/link
   ↓
Validate challenge
   ↓
Enter new password
   ↓
Password hash updated
   ↓
Sessions invalidated according to policy
   ↓
Security notification
   ↓
Login again
```

---

# 120. Full Access-Control Scenario — Doctor

```text
Doctor logs in
   ↓
JWT/session valid
   ↓
Doctor permission = Prescriptions.Create
   ↓
Doctor requests appointment
   ↓
Resource authorization
   ↓
Appointment belongs to Doctor?
   ↓ yes
Appointment state allows prescription?
   ↓ yes
Create prescription
   ↓
Audit where appropriate
```

---

# 121. Full Access-Control Failure — Doctor tries another doctor's appointment

```text
Doctor A authenticated
   ↓
Permission = Prescriptions.Create
   ↓
Requests Appointment owned by Doctor B
   ↓
Resource authorization fails
   ↓
Action rejected
   ↓
Security event if configured
```

Permission alone must not bypass ownership.

---

# 122. Full Access-Control Scenario — Pharmacy Staff

```text
Staff logs in
   ↓
Pharmacy Staff role
   ↓
Membership = Pharmacy 15
   ↓
Permission = Orders.UpdateStatus
   ↓
Request order 900
   ↓
Order belongs to Pharmacy 15?
   ↓ yes
State transition valid?
   ↓ yes
Update order
```

---

# 123. Full Access-Control Scenario — Laboratory

```text
Lab Staff
   ↓
Permission = LabResults.Publish
   ↓
Request result 500
   ↓
Result belongs to their laboratory?
   ↓ yes
Staff role allowed to publish?
   ↓ yes
Result publication rules pass?
   ↓ yes
Publish
```

A collector should not automatically inherit result-publication permission merely because they belong to the same laboratory.

---

# 124. Full Access-Control Scenario — Patient Medical Data

```text
Patient A
   ↓
GET /patients/B/medical-profile
   ↓
Authenticated = yes
   ↓
Role = Patient
   ↓
Requested resource = Patient B
   ↓
Ownership check = false
   ↓
Access denied
```

---

# 125. Session Revocation Scenario

```text
User logged in on phone + laptop
       ↓
User changes password
       ↓
Server revokes existing sessions according to policy
       ↓
Laptop refresh attempt
       ↓
Refresh token rejected
       ↓
Login required
```

---

# 126. Refresh Token Reuse Scenario

```text
Refresh Token A
   ↓
Legitimate client uses A
   ↓
B issued
   ↓
Attacker uses stolen A
   ↓
Reuse detected
   ↓
Security response
   ↓
Revoke token family/session
   ↓
Security alert
```

---

# 127. OTP Abuse Scenario

```text
Attacker requests OTP repeatedly
   ↓
Rate limit reached
   ↓
Further requests blocked/throttled
   ↓
Security event may be recorded
```

---

# 128. Account Enumeration Scenario

The public recovery API should avoid meaningful differences such as:

```text
200 + "Email exists"
404 + "Email not found"
```

Instead, use a generic response where suitable while still performing the appropriate server-side behavior.

---

# 129. Security Notification Scenario

```text
Successful login from unfamiliar environment
   ↓
Create security notification
   ↓
Send configured alert
   ↓
User reviews account sessions
```

---

# 130. Emergency Account Lock

For suspected compromise:

```text
Admin/Security mechanism
   ↓
Lock account
   ↓
Revoke sessions
   ↓
Disable sensitive actions
   ↓
Notify user/support
   ↓
Audit
```

Recovery must require an explicit trusted workflow.

---

# 131. Authorization State Machine

```text
REQUEST
   ↓
AUTHENTICATED?
 ┌───────┴────────┐
 NO               YES
 ↓                 ↓
401          ACCOUNT ALLOWED?
             ┌──────┴──────┐
            NO             YES
            ↓               ↓
           403        HAS PERMISSION?
                      ┌─────┴─────┐
                     NO           YES
                     ↓             ↓
                    403      RESOURCE ACCESS?
                              ┌────┴────┐
                             NO         YES
                             ↓           ↓
                            403      BUSINESS STATE?
                                        ┌─┴─┐
                                       NO   YES
                                       ↓     ↓
                                      409   ALLOW
```

The exact HTTP status for business-rule failures should be selected consistently by the API team.

---

# 132. Registration State Machine

```text
NEW
 ↓
REGISTERED
 ↓
PENDING_VERIFICATION
 ├── VERIFY → VERIFIED
 └── EXPIRE/ABANDON → RESTART/RECOVERY
 ↓
ACTIVE
```

---

# 133. Credential State Machine

```text
PASSWORD_VALID
   ↓
CHANGE REQUEST
   ↓
VERIFIED
   ↓
PASSWORD UPDATED
   ↓
SESSION POLICY APPLIED
```

---

# 134. MFA State Machine

```text
DISABLED
   ↓ setup
PENDING_ENROLLMENT
   ↓ verify
ENABLED
   ↓ disable + verification
DISABLED
```

---

# 135. Session State Machine

```text
CREATED
   ↓
ACTIVE
   ├── REFRESHED
   ├── LOGGED_OUT
   ├── REVOKED
   └── EXPIRED
```

---

# 136. Organization Invitation State Machine

```text
CREATED
   ↓
SENT
   ↓
PENDING
 ├── ACCEPTED
 ├── EXPIRED
 └── REVOKED
```

---

# 137. Important Domain Boundary

Authentication module should **not** own business logic such as:

```text
Doctor consultation logic
Pharmacy inventory logic
Laboratory sample logic
Blood matching logic
Appointment pricing
```

It owns:

```text
Identity
Credentials
Sessions
Roles
Permissions
Security state
Authentication audit
```

Domain modules own their own business rules.

---

# 138. Shared Contracts

The auth module should publish clear internal contracts such as:

```text
IIdentityContext
IUserContext
IPermissionChecker
IOrganizationAccessChecker
ISecurityAuditWriter
```

This avoids every module depending on authentication internals.

---

# 139. Suggested Project Structure — ASP.NET Core

```text
src/
├── TechCare.Api/
│   ├── Controllers/
│   ├── Middleware/
│   └── Program.cs
│
├── TechCare.Application/
│   ├── Authentication/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── DTOs/
│   │   └── Services/
│   └── Authorization/
│       ├── Policies/
│       ├── Handlers/
│       └── Services/
│
├── TechCare.Domain/
│   ├── Identity/
│   ├── Security/
│   └── Organizations/
│
└── TechCare.Infrastructure/
    ├── Identity/
    ├── Persistence/
    ├── Tokens/
    ├── OTP/
    └── Security/
```

The exact folder architecture may differ; the separation of responsibilities is the important part.

---

# 140. Authentication Service Responsibilities

`IIdentityService` can handle:

```text
Register
Validate credentials
Create identity
Update credential state
Resolve current user
```

`ITokenService`:

```text
Create access token
Validate refresh session
Rotate refresh token
Revoke token/session
```

`IOtpService`:

```text
Create challenge
Send challenge
Verify challenge
Invalidate challenge
```

`ISecurityAuditService`:

```text
Write structured security events
```

---

# 141. Example Login Application Command

Conceptual request:

```csharp
public sealed record LoginCommand(
    string Identifier,
    string Password
);
```

The command handler:

```text
Validate input
→ Find user
→ Check state
→ Verify credentials
→ Evaluate MFA
→ Create session/tokens
→ Audit event
```

---

# 142. Example Permission Check

Conceptual:

```csharp
await authorizationService.AuthorizeAsync(
    currentUser,
    appointment,
    Permissions.Appointments.Accept);
```

The handler can combine:

```text
Role permission
+ ownership
+ provider status
+ appointment state
```

---

# 143. Error Handling Strategy

Authentication errors should not leak sensitive internals.

Example safe classes:

```text
INVALID_CREDENTIALS
ACCOUNT_NOT_ACTIVE
VERIFICATION_REQUIRED
MFA_REQUIRED
OTP_INVALID
OTP_EXPIRED
RATE_LIMITED
SESSION_EXPIRED
FORBIDDEN
```

Internal logs can preserve more diagnostic information without returning it to the user.

---

# 144. Validation Error Example

```json
{
  "code": "VALIDATION_ERROR",
  "message": "One or more fields are invalid.",
  "errors": {
    "password": [
      "Password does not meet the configured policy."
    ]
  }
}
```

Do not echo sensitive credentials.

---

# 145. Rate Limit Response

Possible response:

```json
{
  "code": "RATE_LIMITED",
  "message": "Too many requests. Please try again later."
}
```

Include standard retry metadata where supported and safe.

---

# 146. Test Strategy

Authentication is security-critical and should receive unit, integration, authorization, and end-to-end tests.

Test layers:

```text
Unit Tests
Integration Tests
API Tests
Authorization Tests
Security Tests
Concurrency Tests
End-to-End Tests
```

---

# 147. Registration Test Cases

```text
✓ Valid registration
✓ Duplicate email
✓ Duplicate phone
✓ Invalid email
✓ Weak password
✓ Password mismatch
✓ Unsupported role
✓ Self-registration attempt as Admin
✓ Registration race condition
✓ OTP sent
✓ OTP verification
✓ OTP expiration
✓ OTP max attempts
✓ OTP replay blocked
```

---

# 148. Login Test Cases

```text
✓ Valid credentials
✓ Invalid password
✓ Unknown identity
✓ Unverified account
✓ Suspended account
✓ Deactivated account
✓ MFA required
✓ MFA failure
✓ Successful login event
✓ Failed login logging
✓ Rate limiting
```

---

# 149. Refresh Token Test Cases

```text
✓ Valid refresh
✓ Expired refresh
✓ Revoked refresh
✓ Malformed refresh
✓ Rotation works
✓ Old refresh rejected
✓ Token reuse detection
✓ Concurrent refresh behavior
✓ Suspended user cannot refresh
✓ Password reset invalidates sessions as designed
```

---

# 150. Password Reset Test Cases

```text
✓ Existing account recovery
✓ Unknown account generic response
✓ Valid OTP/link
✓ Expired challenge
✓ Reused challenge
✓ Wrong purpose
✓ Weak new password
✓ Successful reset
✓ Existing session behavior verified
```

---

# 151. Authorization Test Cases

```text
✓ Patient can read own resource
✓ Patient cannot read another patient's resource
✓ Doctor can access assigned appointment
✓ Doctor cannot access unrelated appointment
✓ Nurse cannot create doctor-only prescription
✓ Pharmacy staff cannot modify another pharmacy
✓ Lab staff cannot access another lab's sample
✓ Collector cannot publish result without permission
✓ Admin with limited role cannot use restricted operation
✓ Super admin can perform configured privileged operation
```

---

# 152. IDOR/BOLA Test Cases

For every `{id}` endpoint, explicitly attempt:

```text
Authenticated User A
   ↓
Request Resource owned by User B
```

Repeat for:

- appointments
- prescriptions
- orders
- lab results
- samples
- blood requests
- financial records
- staff records
- organization data

This test category should be mandatory before release.

---

# 153. Privilege Escalation Tests

Attempt:

```text
Patient → Admin
Patient → Doctor
Pharmacy Staff → Owner
Lab Collector → Result Publisher
Normal Admin → SuperAdmin
```

Through:

- request body
- query parameters
- manipulated IDs
- altered claims
- replayed tokens
- frontend modifications

All must fail unless explicitly authorized.

---

# 154. Session Security Tests

```text
✓ Logout revokes session
✓ Logout-all revokes all sessions
✓ Password reset invalidates sessions as designed
✓ Suspended account cannot continue privileged access
✓ Expired token rejected
✓ Wrong issuer rejected
✓ Wrong audience rejected
✓ Invalid signature rejected
✓ Token reuse detected where configured
```

---

# 155. OTP Security Tests

```text
✓ OTP expiry
✓ OTP replay blocked
✓ OTP brute-force rate limited
✓ Wrong-purpose OTP rejected
✓ Consumed OTP rejected
✓ Multiple simultaneous verification requests handled safely
```

---

# 156. MFA Tests

```text
✓ MFA enrollment
✓ Invalid MFA code
✓ Successful MFA challenge
✓ MFA disable requires verification
✓ Recovery after lost device
✓ Admin MFA policy enforced
```

---

# 157. Audit Tests

Verify audit generation for:

```text
login
logout
password change
password reset
MFA changes
role changes
permission changes
suspension
reactivation
security alerts
```

Also verify that audit records themselves are protected.

---

# 158. Performance Considerations

Auth is called frequently, so optimize carefully.

Possible optimizations:

- indexed normalized email/phone
- indexed role mappings
- indexed refresh-token hashes
- indexed active session queries
- cached static permission metadata
- efficient claims extraction
- avoid repeated database calls inside a single request

Do not cache authorization decisions longer than is safe for the permission-change model.

---

# 159. Database Indexing

Potential indexes:

```text
User.NormalizedEmail
User.NormalizedPhoneNumber
RefreshTokenSession.TokenHash
RefreshTokenSession.UserId
RefreshTokenSession.ExpiresAt
RefreshTokenSession.RevokedAt
OtpChallenge.UserId
OtpChallenge.Purpose
OtpChallenge.ExpiresAt
SecurityAuditLog.UserId
SecurityAuditLog.EventType
SecurityAuditLog.Timestamp
OrganizationMembership.UserId
OrganizationMembership.OrganizationId
OrganizationMembership.Status
```

Exact indexing should be validated against actual query patterns.

---

# 160. Background Jobs

Possible background jobs:

```text
Expire OTP challenges
Expire refresh sessions
Clean temporary security records
Send security notifications
Detect suspicious patterns
Create security summaries
```

Background jobs must use service-level permissions and should not pretend to be a human admin.

---

# 161. Logging Rules

Do log:

```text
Event type
User ID
Correlation ID
Timestamp
Success/failure
Reason code
Relevant target IDs
```

Do not log:

```text
Passwords
OTP plaintext
Refresh token plaintext
Access token plaintext
Reset token plaintext
Secrets
```

Be especially careful with HTTP request-body logging.

---

# 162. Privacy-Minimal Auth Data

Authentication should store only what is necessary.

Example:

```text
Phone number → identity/recovery
Email → identity/recovery
IP → security monitoring
User Agent → session/security
```

Retention of security logs should be an explicit system policy.

---

# 163. Data Retention Boundaries

Define separate retention rules for:

```text
Active identity
Session records
OTP challenges
Security audit logs
Deleted/deactivated accounts
```

Do not allow temporary OTP data to remain indefinitely.

---

# 164. Recovery vs Verification

Important distinction:

```text
Email verification
≠
Password recovery
≠
MFA
≠
Professional verification
```

They serve different security and business purposes.

Doctor professional verification, for example, does not prove that the doctor remembers their password, and password login does not prove professional qualification.

---

# 165. Authentication vs Professional Verification

Example:

```text
Doctor Account
   ↓
Identity verified = YES
Professional docs approved = NO
```

The doctor is a legitimate authenticated user but may remain unable to perform provider operations that require professional approval.

This pattern applies to nurses, pharmacies, and laboratories too.

---

# 166. Authentication vs Donor Eligibility

Example:

```text
Donor account ACTIVE
   ↓
Blood donation opportunity
   ↓
Eligibility/business rules checked by Blood module
```

Authentication must not decide medical donor eligibility.

---

# 167. Authentication vs Financial Permissions

Being logged in does not imply withdrawal permission.

Example:

```text
Doctor authenticated
   ↓
Finance.ReadOwn = YES
   ↓
Withdrawal.Create = YES
   ↓
Withdrawal.Approve = NO
```

Approval belongs to an authorized operational role.

---

# 168. Secure API Endpoint Example

Conceptually:

```csharp
[Authorize(Policy = Policies.AppointmentsAccept)]
[HttpPost("{id}/accept")]
public async Task<IActionResult> Accept(Guid id)
{
    // Load resource
    // Verify object-level access
    // Verify business state
    // Execute transition
}
```

The important part is that the endpoint verifies the **specific resource**, not only the global role.

---

# 169. API Request Sequence

```text
Frontend
  ↓
Authorization header / secure session
  ↓
ASP.NET Core Authentication
  ↓
ClaimsPrincipal
  ↓
Authorization Policy
  ↓
Endpoint
  ↓
Application Service
  ↓
Resource Authorization
  ↓
Business Rules
  ↓
Database
```

---

# 170. API Security Checklist per Endpoint

For every protected endpoint, ask:

```text
1. Does it require authentication?
2. Which permission is required?
3. Which roles can normally have that permission?
4. What resource is being accessed?
5. Who owns that resource?
6. Does organization scope matter?
7. Is account/provider verification required?
8. Is the business state valid?
9. Is step-up authentication required?
10. Must the action be audited?
```

---

# 171. Endpoint Matrix Template

Use this table for every API:

| Endpoint | Auth | Permission | Resource Check | Org Scope | Extra Verification | Audit |
|---|---|---|---|---|---|---|
| POST /auth/login | No | — | — | — | MFA if required | Yes |
| POST /auth/change-password | Yes | Own credential | Current user | — | Current password | Yes |
| GET /appointments/{id} | Yes | Appointments.Read | Owner/assigned | Maybe | No | Optional |
| POST /prescriptions | Yes | Prescriptions.Create | Authorized appointment | No | Maybe | Yes |
| PATCH /pharmacies/{id}/orders/{orderId} | Yes | Orders.UpdateStatus | Pharmacy membership | Yes | No | Yes |
| POST /lab-results/{id}/publish | Yes | LabResults.Publish | Lab membership | Yes | Maybe | Yes |

---

# 172. Complete Authentication Journey

```text
┌──────────────────────┐
│ User Opens TechCare  │
└──────────┬───────────┘
           ↓
     Register/Login
           ↓
     Authentication
           ↓
      Verification
           ↓
      MFA (if needed)
           ↓
    Session / Tokens
           ↓
   Current User Context
           ↓
   Role + Permissions
           ↓
   Resource Authorization
           ↓
      Business Rules
           ↓
       API Action
           ↓
      Audit/Event
```

---

# 173. Complete Authorization Journey

```text
Request
  ↓
Who are you?
  ↓
Authenticated user
  ↓
Are you active?
  ↓
What can your role do?
  ↓
Do you have permission?
  ↓
Does this exact object belong to your scope?
  ↓
Does organization membership match?
  ↓
Is business state valid?
  ↓
Is extra verification required?
  ↓
Allow / Deny
```

---

# 174. Full End-to-End Scenario — Doctor

```text
Doctor registers
   ↓
OTP verification
   ↓
Doctor role assigned
   ↓
Profile created
   ↓
Professional verification pending
   ↓
Admin approves documents
   ↓
Doctor logs in
   ↓
MFA if required
   ↓
Access token/session created
   ↓
Doctor receives appointment request
   ↓
Authorization checks Doctor appointment permission
   ↓
Appointment belongs to doctor
   ↓
Accept action allowed
   ↓
Consultation starts
   ↓
Doctor creates diagnosis/treatment/prescription
   ↓
Authorization verifies relationship and scope
   ↓
Prescription saved
   ↓
Audit created where applicable
```

---

# 175. Full End-to-End Scenario — Pharmacy Staff

```text
Staff account invited
   ↓
Invitation accepted
   ↓
Identity authenticated
   ↓
Membership activated
   ↓
Role = Pharmacist/Staff
   ↓
Login
   ↓
Orders.UpdateStatus permission
   ↓
Staff opens order
   ↓
Server verifies order belongs to pharmacy
   ↓
State transition validated
   ↓
Order updated
   ↓
Audit recorded
```

---

# 176. Full End-to-End Scenario — Laboratory

```text
Lab staff login
   ↓
Membership checked
   ↓
Lab role resolved
   ↓
Sample request loaded
   ↓
Organization access verified
   ↓
Collector updates collection state
   ↓
Collector cannot publish result
   ↓
Authorized result validator can validate
   ↓
Authorized publisher publishes
```

---

# 177. Full End-to-End Scenario — Admin

```text
Admin login
   ↓
Credential verification
   ↓
MFA
   ↓
Admin permissions
   ↓
Open verification request
   ↓
Read permitted provider documents
   ↓
Approve/reject
   ↓
Professional module status updated
   ↓
Security audit created
   ↓
Provider notified
```

---

# 178. Cross-Module Security Principle

Every business module must use the shared authentication context.

Never create:

```text
Doctor JWT
Nurse JWT
Patient JWT
Pharmacy JWT
```

as completely separate identity systems for the same platform account.

Instead:

```text
Shared Identity
    ↓
Role / Permission / Context
    ↓
Module Authorization
```

---

# 179. What the Auth Team Owns

The Authentication & Authorization owner is responsible for:

### Identity

- users
- registration
- login
- verification
- credential management

### Session

- access tokens
- refresh tokens
- logout
- sessions
- revocation

### Security

- OTP
- MFA
- rate limits
- account lock/restrictions
- security events
- security notifications

### Authorization

- roles
- permissions
- policies
- resource authorization helpers
- organization membership authorization

### Integration

- shared middleware
- shared CurrentUser service
- shared authorization contracts
- security documentation
- auth integration tests

---

# 180. What the Auth Team Does NOT Own

Auth team should not own:

```text
Doctor diagnosis logic
Nurse visit notes
Pharmacy inventory rules
Laboratory sample processing
Blood donor matching logic
Appointment pricing
Patient medical-history business logic
```

The respective module owns those concerns while consuming the shared auth services.

---

# 181. Definition of Done — Authentication

Authentication is considered complete when:

```text
[ ] Register works
[ ] Contact verification works
[ ] Login works
[ ] JWT/session creation works
[ ] Refresh flow works
[ ] Rotation/revocation strategy works
[ ] Logout works
[ ] Logout-all works
[ ] Forgot password works
[ ] Reset password works
[ ] Change password works
[ ] Current user endpoint works
[ ] Account states work
[ ] Rate limiting exists
[ ] OTP protection exists
[ ] Passwords are hashed
[ ] Secrets are not committed
[ ] Security logging exists
[ ] Sessions can be revoked
```

---

# 182. Definition of Done — Authorization

```text
[ ] Roles implemented
[ ] Permissions implemented
[ ] Policies implemented
[ ] Resource authorization implemented
[ ] Organization scope implemented where required
[ ] Least privilege applied
[ ] Admin roles separated
[ ] Self-role-escalation blocked
[ ] IDOR/BOLA tests pass
[ ] Privilege escalation tests pass
[ ] Frontend route guards implemented
[ ] Backend remains authoritative
```

---

# 183. Definition of Done — Security

```text
[ ] HTTPS in deployment
[ ] Rate limiting
[ ] Anti-enumeration behavior
[ ] OTP expiry
[ ] OTP attempt limits
[ ] Token validation
[ ] Refresh token protection
[ ] Session revocation
[ ] Audit logs
[ ] No secrets in logs
[ ] No plaintext passwords
[ ] No plaintext OTPs
[ ] No plaintext refresh tokens where hashing is used
[ ] Strong authorization tests
[ ] Security incident handling path documented
```

---

# 184. Required Integration Checklist for Other Team Members

Before Patient/Doctor/Nurse/Pharmacy/Lab/Blood modules consider Auth integrated:

```text
[ ] They can obtain current user ID securely
[ ] They do not accept user ID blindly from frontend for own-resource operations
[ ] They can require permissions/policies
[ ] They can perform object-level authorization
[ ] They understand organization scope
[ ] They use standard 401/403 handling
[ ] They do not implement custom passwords
[ ] They do not implement custom JWT generation
[ ] They do not duplicate OTP logic
[ ] They emit domain audit events when needed
```

---

# 185. Recommended Auth Integration Contract

Every module should be able to answer through shared services:

```text
Who is the current user?
What are their roles?
What permissions do they have?
Which organizations do they belong to?
Is the account active?
Can they perform this operation on this resource?
```

---

# 186. Practical API Integration Example

Patient module:

```text
GET /api/v1/patients/me
```

Backend:

```text
CurrentUserService.UserId
   ↓
PatientRepository.GetByUserId()
```

Do not require:

```text
GET /patients/{anyUserId}
```

for the ordinary "my profile" flow.

---

# 187. Practical API Integration Example — Doctor

```text
POST /api/v1/doctor/appointments/{id}/accept
```

Backend sequence:

```text
Authenticate Doctor
   ↓
Permission check
   ↓
Load appointment
   ↓
Verify appointment.DoctorId == currentUser.DoctorId
   ↓
Verify appointment state
   ↓
Accept
```

---

# 188. Practical API Integration Example — Pharmacy

```text
PATCH /api/v1/pharmacies/{pharmacyId}/orders/{orderId}
```

Backend sequence:

```text
Authenticate
   ↓
Organization membership check
   ↓
Permission check
   ↓
Load order
   ↓
Verify order.PharmacyId == pharmacyId
   ↓
Verify user belongs to pharmacy
   ↓
Verify state transition
   ↓
Update
```

---

# 189. Practical API Integration Example — Laboratory

```text
POST /api/v1/laboratories/{labId}/results/{resultId}/publish
```

Backend sequence:

```text
Authenticate
   ↓
Lab membership
   ↓
LabResults.Publish permission
   ↓
Resource ownership
   ↓
Result validation state
   ↓
Publish
   ↓
Audit
```

---

# 190. Practical API Integration Example — Blood Donor

```text
POST /api/v1/donations/{id}/accept
```

Backend sequence:

```text
Authenticate
   ↓
Donor role/profile
   ↓
Donation opportunity belongs to current donor
   ↓
Opportunity still open?
   ↓
Donor eligibility/business rules
   ↓
Accept
```

The blood module owns eligibility rules; auth only establishes identity and permission.

---

# 191. Admin Security Checklist

```text
[ ] Admin authentication uses dedicated policies
[ ] MFA policy configured
[ ] Admin role assignment restricted
[ ] Sensitive admin actions audited
[ ] Admin endpoints have resource checks
[ ] No self-escalation
[ ] Session revocation supported
[ ] Security logs visible only to authorized admins
[ ] Financial approval permissions separated where appropriate
```

---

# 192. Deployment Checklist

Before production-like deployment:

```text
[ ] Production HTTPS configured
[ ] JWT signing configuration secured
[ ] Database credentials secured
[ ] Email/SMS secrets secured
[ ] CORS restricted
[ ] Rate limiting configured
[ ] Logging configured safely
[ ] Error details disabled for production
[ ] Refresh token/session storage protected
[ ] Password policy configured
[ ] Admin MFA policy configured
[ ] Backup/recovery procedures defined
[ ] Security monitoring enabled
```

---

# 193. Environment Separation

Use separate configuration/secrets for:

```text
Development
Testing
Staging
Production
```

Never copy real production credentials into local development.

---

# 194. Testing Environment Rules

Test users should use synthetic data.

Do not place real passwords, medical records, or real patient identifiers in automated test fixtures.

---

# 195. Security Review Before Merge

Every auth-related Pull Request should be checked for:

```text
Authentication bypass
Authorization bypass
Mass assignment
Sensitive logging
Token handling
Rate-limit bypass
Object ownership checks
Error leakage
Concurrency issues
Session invalidation behavior
```

---

# 196. Suggested Team Pull Requests

Break the implementation into small PRs:

```text
PR 1 → Identity/Users
PR 2 → Registration + Verification
PR 3 → Login + JWT
PR 4 → Refresh Sessions
PR 5 → Logout/Revocation
PR 6 → Password Recovery
PR 7 → Roles/Permissions
PR 8 → Policies/Handlers
PR 9 → Organization Membership
PR 10 → MFA/Step-up
PR 11 → Security Audit
PR 12 → Rate Limiting/Security Hardening
PR 13 → Integration Tests
```

---

# 197. Suggested Implementation Order

Recommended dependency order:

```text
1. Identity model
2. ASP.NET Core Identity setup
3. User registration
4. Verification/OTP
5. Login
6. JWT/session model
7. Refresh token rotation
8. Logout/revocation
9. Password reset/change
10. Roles
11. Permissions
12. Policies
13. Resource authorization
14. Organization membership
15. MFA
16. Audit logging
17. Rate limiting
18. Integration with all modules
19. Security testing
```

---

# 198. MVP vs Future Scope

## MVP

```text
✓ Registration
✓ Login
✓ Logout
✓ Password reset
✓ Password change
✓ OTP verification
✓ JWT/session authentication
✓ Refresh token
✓ Roles
✓ Permission-based authorization
✓ Resource ownership checks
✓ Organization scope
✓ Account suspension
✓ Security audit
✓ Rate limiting for auth endpoints
```

## Future / Advanced

```text
→ Passkeys/WebAuthn
→ Risk-based authentication
→ Device reputation
→ Advanced anomaly detection
→ Adaptive MFA
→ Break-glass emergency access
→ Advanced session analytics
→ Central SIEM integration
→ Fine-grained ABAC
```

---

# 199. Final Architecture Diagram

```text
                         ┌─────────────────────┐
                         │      Frontend       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Authentication API  │
                         └──────────┬──────────┘
                                    │
            ┌───────────────────────┼────────────────────────┐
            │                       │                        │
            ▼                       ▼                        ▼
      Identity Store          Token/Session Store       OTP Service
            │                       │                        │
            └───────────────────────┼────────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Authorization Layer │
                         │ Roles + Permissions │
                         │ Policies + Handlers │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼─────────────────┐
                    │               │                 │
                    ▼               ▼                 ▼
                 Patient          Doctor            Nurse
                    │               │                 │
                    ├───────────────┼─────────────────┤
                    │               │                 │
                    ▼               ▼                 ▼
               Pharmacy         Laboratory        Blood Donor
                    │               │                 │
                    └───────────────┼─────────────────┘
                                    │
                                    ▼
                                Admin
                                    │
                                    ▼
                         Security Audit / Monitoring
```

---

# 200. Final Authentication State Diagram

```text
                         ┌───────────────┐
                         │   REGISTER    │
                         └───────┬───────┘
                                 ▼
                      ┌────────────────────┐
                      │ PENDING VERIFICATION│
                      └─────────┬──────────┘
                                │ OTP
                                ▼
                         ┌─────────────┐
                         │  VERIFIED   │
                         └──────┬──────┘
                                ▼
                         ┌─────────────┐
                         │    ACTIVE   │
                         └──────┬──────┘
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
        RESTRICTED          SUSPENDED         DEACTIVATED
             │                  │
             └──────────┬───────┘
                        ▼
                     ACTIVE
```

Authentication does not itself decide domain eligibility; it supplies the secure identity and authorization foundation that all other workflows depend on.

---

# 201. Final Checklist for TechCare

## Authentication

```text
[ ] One shared authentication system
[ ] One user identity per account
[ ] Registration
[ ] OTP verification
[ ] Login
[ ] JWT/session
[ ] Refresh token
[ ] Refresh rotation
[ ] Logout
[ ] Logout all
[ ] Forgot password
[ ] Reset password
[ ] Change password
[ ] Change email
[ ] Change phone
[ ] MFA
[ ] Session management
[ ] Account states
```

## Authorization

```text
[ ] Roles
[ ] Permissions
[ ] Policies
[ ] Resource authorization
[ ] Ownership checks
[ ] Organization scope
[ ] Least privilege
[ ] Separation of duties
[ ] Admin hierarchy
[ ] Privilege escalation protection
```

## Security

```text
[ ] Password hashing
[ ] OTP hashing/protection
[ ] Token protection
[ ] Rate limiting
[ ] Anti-enumeration
[ ] CSRF strategy where needed
[ ] XSS-aware token handling
[ ] HTTPS
[ ] Secret management
[ ] Audit logs
[ ] Correlation IDs
[ ] Security monitoring
```

## Testing

```text
[ ] Unit tests
[ ] Integration tests
[ ] API tests
[ ] Authorization tests
[ ] BOLA/IDOR tests
[ ] Privilege escalation tests
[ ] OTP tests
[ ] Refresh token tests
[ ] Session tests
[ ] Concurrency tests
[ ] E2E tests
```

---

# 202. Final Outcome

The final TechCare authentication architecture should provide:

```text
                    SECURE IDENTITY
                           │
                           ▼
                 ┌───────────────────┐
                 │  AUTHENTICATION   │
                 │ Login / OTP / MFA │
                 │ Password / Session│
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │  AUTHORIZATION    │
                 │ Roles / Permissions│
                 │ Policies / Scope  │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ RESOURCE ACCESS   │
                 │ Ownership / Org   │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ BUSINESS RULES    │
                 └─────────┬─────────┘
                           │
                           ▼
                    ALLOW / DENY
                           │
                           ▼
                      AUDIT EVENT
```

The central architectural rule is:

> **Authentication proves identity. Authorization grants capability. Resource authorization protects the specific object. Business rules decide whether the action is valid now.**

All TechCare roles should consume this same security foundation rather than creating separate authentication systems.
