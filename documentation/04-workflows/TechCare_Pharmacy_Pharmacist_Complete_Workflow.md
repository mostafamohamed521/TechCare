# TechCare — Complete Pharmacy & Pharmacist Workflow

## End-to-End Product, Functional, Backend, Frontend, Database, Security, Operations, and Testing Specification

> **Project:** TechCare  
> **Module:** Pharmacy / Pharmacist  
> **Audience:** Product Owner, Scrum Team, Full-Stack .NET Developers, QA, UI/UX, System Design  
> **Status:** Prototype / MVP Specification  
> **Core Principle:** Bring Healthcare Closer to the Patient

---

# 1. Module Overview

The Pharmacy Module connects patients with verified pharmacies and pharmacists. It covers the complete lifecycle of:

```text
Pharmacy Registration
        ↓
Pharmacist Registration / Staff Linking
        ↓
Identity & Professional Verification
        ↓
Pharmacy Approval
        ↓
Pharmacy Profile Setup
        ↓
Medicine Catalog
        ↓
Inventory Setup
        ↓
Pricing & Availability
        ↓
Patient Medicine Search
        ↓
Prescription-Based Search (when applicable)
        ↓
Cart / Order Creation
        ↓
Pharmacy Review of Order
        ↓
Accept / Reject / Request Clarification
        ↓
Stock Reservation
        ↓
Preparation
        ↓
Delivery / Pickup
        ↓
Order Completion
        ↓
Payment Settlement
        ↓
Rating / Complaint
        ↓
Pharmacy Earnings / Withdrawal
```

The pharmacy module is not only a product marketplace. It must also model the real pharmacy operation: pharmacy organization, pharmacist/staff access, medicine master data, stock, pricing, prescription handling, order preparation, fulfillment, refunds, and auditability.

---

# 2. Main Goals

The module should allow the platform to:

1. Register pharmacies and pharmacists.
2. Verify professional and business information.
3. Create one shared medicine master catalog.
4. Maintain pharmacy-specific inventory and prices.
5. Let patients discover nearby pharmacies.
6. Show medicine availability without exposing internal pharmacy data.
7. Let patients search by medicine name, active ingredient, form, strength, and availability.
8. Support prescription-linked medicine requests.
9. Prevent overselling through stock reservation.
10. Support delivery and pickup.
11. Manage order states from creation to completion.
12. Handle rejected items, substitutions, cancellations, returns, and refunds through controlled workflows.
13. Maintain auditable financial transactions.
14. Support ratings, complaints, notifications, and admin moderation.
15. Enforce least-privilege access for pharmacists and staff.

---

# 3. Core Domain Model Principle

## 3.1 One Shared Medicine Master Catalog

Do **not** create a separate medicine database for every pharmacy.

Use:

```text
Medicine Master Catalog
        │
        ├── Pharmacy A Inventory
        ├── Pharmacy B Inventory
        ├── Pharmacy C Inventory
        └── Pharmacy D Inventory
```

The master catalog stores properties that belong to the medicine itself.

Example:

```text
Medicine
├── Generic / Active Ingredient Name
├── Brand Name
├── Strength
├── Dosage Form
├── Route (when relevant)
├── Manufacturer
├── Prescription Requirement Flag
├── Package Description
├── Product Identifiers
└── Catalog Status
```

## 3.2 Pharmacy-Specific Inventory

Inventory stores what differs from one pharmacy to another:

```text
PharmacyInventory
├── PharmacyId
├── MedicineId
├── SKU / Pharmacy Product Code
├── Selling Price
├── OnHandQuantity
├── ReservedQuantity
├── AvailableQuantity
├── Availability Status
├── Delivery Available
├── Pickup Available
├── Reorder Threshold
└── Last Updated
```

Available quantity should be derived safely:

```text
Available Quantity = On Hand - Reserved
```

The exact implementation can use a computed value or carefully maintained transactional counters.

---

# 4. Main Actors

The pharmacy domain can involve several actors.

## 4.1 Pharmacy Owner / Manager

Can manage the pharmacy organization and authorized staff.

## 4.2 Pharmacist

Handles professional pharmacy workflows such as reviewing prescriptions and orders within the platform's defined scope.

## 4.3 Pharmacy Staff

May help with inventory, preparation, and order processing, according to assigned permissions.

## 4.4 Patient

Searches medicines, selects pharmacies, submits prescription/order requests, pays, receives/picks up orders, and rates the service.

## 4.5 Doctor

Can issue prescriptions through the Doctor module. Prescription data can be linked to the pharmacy workflow.

## 4.6 Delivery Staff / Courier (optional in prototype)

Can be represented internally or integrated later. The MVP may keep fulfillment coordination under the pharmacy.

## 4.7 Admin

Controls pharmacy verification, medicine master data, disputes, complaints, suspended pharmacies, catalog governance, and sensitive operations.

---

# 5. Pharmacy Registration Workflow

The pharmacy onboarding is separate from a regular personal account because the platform needs organization information.

```text
Create Account
    ↓
Select Pharmacy Role / Pharmacy Organization
    ↓
Enter Organization Data
    ↓
Add Responsible Pharmacist
    ↓
Upload Required Documents
    ↓
Set Pharmacy Location
    ↓
Submit for Verification
    ↓
Admin Review
```

---

# 6. Pharmacy Account Information

Possible account data:

```text
Email
Phone
Password
Confirm Password
```

The shared TechCare Authentication module handles:

```text
Register
Login
Logout
Forgot Password
Reset Password
OTP
Refresh Token
Change Password
Account Security
```

The Pharmacy module consumes the authentication foundation rather than recreating it.

---

# 7. Pharmacy Organization Information

The organization profile may contain:

```text
Pharmacy Name
Legal / Business Name (when applicable)
Phone
Email
Governorate
City
Area
Full Address
Latitude / Longitude
Opening Hours
Delivery Availability
Pickup Availability
About Pharmacy
Logo / Cover Image
```

Sensitive legal or verification fields remain private.

---

# 8. Pharmacist Registration and Linking

A pharmacist can either:

1. Register directly as a pharmacy professional, then link to a pharmacy after verification.
2. Be invited by a pharmacy owner/manager to join an existing pharmacy organization.

Recommended organization model:

```text
Pharmacy
    │
    ├── Owner / Manager
    ├── Pharmacist
    ├── Staff
    └── Other Authorized Members
```

Use a relation similar to:

```text
PharmacyStaff
├── PharmacyId
├── UserId
├── StaffRole
├── Status
├── JoinedAt
└── Permissions / Permission Group
```

---

# 9. Pharmacist Professional Information

Possible fields:

```text
Full Name
Professional Qualification
Professional Registration / License Number
Years of Experience
Specialization (when applicable)
Professional Bio
```

Professional documents may include:

```text
Identity Document
Graduation / Qualification Certificate
Professional License / Registration Document
Other Approved Documents
```

All professional documents must be private and access-controlled.

---

# 10. OTP / Verification

The pharmacy and pharmacist use the shared OTP infrastructure where applicable.

Typical sequence:

```text
Registration
    ↓
OTP Sent
    ↓
Enter OTP
    ↓
Verify Contact
    ↓
Submit Professional / Pharmacy Verification
```

Controls:

- OTP expiration.
- Retry and resend limits.
- Rate limiting.
- Server-side validation.
- Audit of important verification actions.

---

# 11. Pharmacy Verification States

Recommended states:

```text
REGISTERED
CONTACT_VERIFIED
PENDING_VERIFICATION
APPROVED
REQUIRES_UPDATE
REJECTED
SUSPENDED
```

Only an approved and active pharmacy should be discoverable as a verified pharmacy.

---

# 12. Admin Verification Workflow

Admin reviews:

```text
Pharmacy Organization Data
Pharmacy Location
Responsible Pharmacist
Professional Documents
Business/Registration Information
Contact Information
```

Possible decisions:

```text
Approve
Reject
Request Update
Suspend
```

The system should preserve the verification history.

---

# 13. Pharmacy Dashboard

The main dashboard can provide:

```text
Today's Orders
Pending Orders
Orders Awaiting Preparation
Ready for Pickup
Out for Delivery
Low Stock Items
Expired / Near-Expiry Alerts
Today's Sales
Pending Settlement
Unread Notifications
Rating
```

Navigation:

```text
Dashboard

Orders
├── New
├── Pending
├── Accepted
├── Needs Clarification
├── Preparing
├── Ready for Pickup
├── Out for Delivery
├── Completed
├── Rejected
├── Cancelled
├── Returned
└── Refunded

Medicines
├── Catalog Search
├── Inventory
├── Low Stock
└── Expiry Alerts

Prescriptions

Pharmacy Profile
Staff
Working Hours
Delivery Settings
Pickup Settings
Pricing / Offers (if enabled)

Financials
├── Sales
├── Transactions
├── Earnings
├── Fees / Commission
├── Refunds
└── Withdrawals

Reviews
Complaints
Notifications
Documents / Verification
Settings
```

---

# 14. Public Pharmacy Profile

Patients can see approved, non-sensitive information:

```text
Pharmacy Name
Verification Badge
Rating
Number of Reviews
Location / Area
Approximate Distance
Opening Hours
Delivery Availability
Pickup Availability
Available Medicines / Search Results
Service Information
```

Do not expose:

```text
Private Documents
Internal Staff Records
Financial Statements
Admin Notes
Inventory System Credentials
Patient Order History of other users
```

---

# 15. Medicine Master Catalog

The master catalog is controlled centrally.

Example medicine record:

```text
Medicine
├── ID
├── GenericName
├── BrandName
├── Strength
├── DosageForm
├── Manufacturer
├── PrescriptionRequired
├── PackageSize
├── ProductIdentifier(s)
├── Description
├── CatalogStatus
└── Created / Updated At
```

The pharmacy should reference this master record instead of creating duplicate medicine definitions.

---

# 16. Medicine Search

Patient search can support:

```text
Medicine Name
Brand Name
Generic Name
Active Ingredient
Strength
Dosage Form
```

Example:

```text
Search: Paracetamol
        ↓
Medicine Master Catalog
        ↓
Search eligible pharmacy inventories
        ↓
Return pharmacies with stock
```

---

# 17. Nearby Pharmacy Search

The patient can filter by:

```text
Distance
Price
Rating
Availability
Delivery
Pickup
Opening Hours
Verified Status
```

Example result:

```text
Pharmacy Alpha
Verified
4.7 ★
2.8 KM
Medicine Available
Price: 75 EGP
Delivery Available
```

Exact patient location must not be publicly exposed.

---

# 18. Inventory Creation

A pharmacy can add a medicine from the master catalog:

```text
Select Medicine
      ↓
Set Pharmacy SKU
      ↓
Set Price
      ↓
Set Quantity
      ↓
Set Availability
      ↓
Enable Delivery / Pickup
      ↓
Save
```

The pharmacy cannot silently alter master medicine identity fields that are centrally controlled.

---

# 19. Inventory Data

Recommended minimum fields:

```text
InventoryId
PharmacyId
MedicineId
SKU
Price
OnHandQuantity
ReservedQuantity
Availability
DeliveryAvailable
PickupAvailable
ReorderThreshold
CreatedAt
UpdatedAt
```

---

# 20. Batch-Level Inventory

For a more complete pharmacy model, medicine inventory can optionally be tracked by batch.

```text
InventoryBatch
├── InventoryId
├── BatchNumber
├── ExpiryDate
├── Quantity
├── PurchaseCost (private)
└── ReceivedAt
```

Batch tracking becomes especially valuable for:

```text
Expiry Management
Recall Handling
Traceability
Stock Rotation
```

The first prototype may keep batch handling simplified, but the domain should remain extensible.

---

# 21. Stock Rotation and Expiry

Recommended stock logic:

```text
Available
    ↓
Near Expiry Alert
    ↓
Expired
    ↓
Unavailable
```

Expired stock must never be treated as sellable stock.

When batch-level inventory is used, the system can support FEFO-like behavior:

```text
First Expire
→ First Out
```

---

# 22. Low Stock Alerts

If:

```text
Available Quantity <= Reorder Threshold
```

then:

```text
Low Stock Alert
```

The pharmacist/manager can see:

```text
Medicine
Current Available Quantity
Threshold
Last Updated
```

---

# 23. Inventory Reservation

This is one of the most important backend rules.

Suppose:

```text
On Hand = 10
Reserved = 3
Available = 7
```

If an order needs 2:

```text
Reserved = 5
Available = 5
```

The system must prevent two concurrent orders from reserving the same physical stock.

Reservation must be handled with transaction/concurrency-safe backend logic.

---

# 24. Stock Adjustment

Authorized pharmacy staff can perform controlled stock adjustments.

Reasons may include:

```text
Stock Received
Damaged
Expired
Manual Correction
Return to Supplier (if supported)
Internal Adjustment
```

Every adjustment should record:

```text
Medicine / Batch
Old Quantity
Adjustment Amount
New Quantity
Reason
User
Timestamp
```

Historical adjustments should be auditable.

---

# 25. Medicine Availability States

Possible item availability:

```text
IN_STOCK
LOW_STOCK
OUT_OF_STOCK
UNAVAILABLE
SUSPENDED
EXPIRED
```

The patient-facing availability should be derived from valid sellable stock, not from a manually entered frontend flag alone.

---

# 26. Prescription Workflow — Overview

The platform should distinguish between:

```text
Non-prescription / OTC-compatible purchase
```

and:

```text
Prescription-linked purchase
```

The exact medicines requiring prescription should be governed by the product's approved catalog and operational policies.

---

# 27. Doctor → Prescription → Pharmacy

A connected flow can be:

```text
Doctor Consultation
      ↓
Doctor creates Prescription
      ↓
Prescription stored securely
      ↓
Patient opens Prescription
      ↓
Find My Medicines
      ↓
Search Nearby Pharmacies
      ↓
Compare Availability / Price / Delivery
      ↓
Select Pharmacy
      ↓
Send Prescription-linked Order Request
```

The pharmacy can then review the prescription within its permissions.

---

# 28. Prescription Data

A prescription can contain:

```text
PrescriptionId
PatientId
DoctorId
IssuedAt
Status
Items
Instructions
Notes
```

Each item may contain:

```text
Medicine Reference
Quantity
Strength / Form when necessary
Directions
Refill Information when supported
```

The pharmacy-facing response should expose only what is needed to fulfill the order.

---

# 29. Prescription Review by Pharmacist

When a prescription-linked order arrives:

```text
New Order
    ↓
Prescription Attached?
   ├── No → Continue OTC rules if allowed
   └── Yes
         ↓
   Pharmacist Reviews
         ↓
   Medicine Match / Availability
         ↓
   Quantity / Product Validation
         ↓
   Accept or Request Clarification / Reject
```

The pharmacist should not arbitrarily edit a doctor's prescription as a normal order-edit operation.

Where clarification/substitution is allowed by the approved workflow, the system should capture the action and reason.

---

# 30. Prescription Clarification

Potential state:

```text
NEEDS_CLARIFICATION
```

Examples:

```text
Medicine unavailable
Product configuration mismatch
Quantity issue
Required information missing
```

The system can notify the patient and/or use an approved doctor-pharmacist clarification flow.

The exact medical decision should not be hidden inside uncontrolled chat messages. Important changes must be structured and auditable.

---

# 31. Medicine Substitution Workflow

If the product later allows substitution workflows, it should be explicit:

```text
Original Requested Medicine
        ↓
Alternative Candidate
        ↓
Reason / Availability
        ↓
Authorization / Approval according to policy
        ↓
Patient/Prescriber Confirmation when required
        ↓
Final Dispensed Item
```

Do not silently replace one medicine with another in the database.

---

# 32. Patient Cart

For standard eligible medicines:

```text
Search Medicine
    ↓
Select Pharmacy
    ↓
Select Quantity
    ↓
Add to Cart
```

The cart should belong to a single pharmacy unless the product explicitly implements multi-pharmacy checkout.

Recommended MVP rule:

```text
One Cart = One Pharmacy
```

This simplifies:

```text
Stock Reservation
Delivery
Payment
Order Tracking
Refunds
```

---

# 33. Cart Validation

At checkout the backend revalidates:

```text
Medicine exists
Medicine is active
Pharmacy is active
Inventory is available
Quantity is allowed
Price is current
Prescription requirement is satisfied
Delivery/pickup method is available
```

The frontend must not be trusted for any of these checks.

---

# 34. Order Creation

Patient confirms:

```text
Pharmacy
Items
Quantities
Delivery / Pickup
Address when delivery
Notes
Payment Method
```

Then:

```text
Create Order
    ↓
Backend Validation
    ↓
Price Calculation
    ↓
Stock Reservation
    ↓
Order Created
```

---

# 35. Order Price Snapshot

At order confirmation, each order line should store the effective selling price at that moment.

Example:

```text
OrderItem
├── MedicineId
├── Quantity
├── UnitPriceSnapshot
└── LineTotal
```

This prevents later pharmacy price changes from rewriting historical orders.

---

# 36. Order Price Calculation

Recommended model:

```text
Medicine Subtotal
+ Delivery Fee
+ Other Approved Fees
- Discount
= Order Total
```

Example:

```text
Medicines          300 EGP
Delivery             40 EGP
Discount             20 EGP
--------------------------
Total               320 EGP
```

The final total must be calculated server-side.

---

# 37. Delivery vs Pickup

The patient chooses:

```text
DELIVERY
```

or:

```text
PICKUP
```

## Delivery

Requires a valid delivery address and applicable delivery fee.

## Pickup

Patient receives:

```text
Pharmacy Address
Pickup Instructions
Order Reference
Ready Time / Status
```

---

# 38. Pharmacy Order Dashboard

New order should display:

```text
Order Number
Patient Basic Information
Order Type
Items
Quantities
Prescription Status (when applicable)
Delivery/Pickup
Total
Payment Status
Created At
Required Action
```

Sensitive patient data should remain limited to the operational need.

---

# 39. Order State Machine

Recommended primary states:

```text
PENDING
   ↓
ACCEPTED
   ↓
RESERVED / PREPARING
   ↓
READY_FOR_PICKUP  OR  OUT_FOR_DELIVERY
   ↓
COMPLETED
```

Alternative branches:

```text
PENDING → REJECTED
PENDING → CANCELLED
ACCEPTED → CANCELLED
PREPARING → CANCELLED (only under controlled rules)
DELIVERY → DELIVERY_FAILED
```

Return/refund states can be represented separately.

---

# 40. Order Acceptance

Pharmacist reviews:

```text
Stock
Prescription Requirement
Requested Quantities
Delivery Capability
Pickup Capability
Patient Notes
```

If valid:

```text
PENDING → ACCEPTED
```

Then stock reservation is confirmed according to the chosen implementation.

---

# 41. Order Rejection

Possible reasons:

```text
Out of Stock
Invalid / Incomplete Prescription
Service Unavailable
Delivery Unavailable
Operational Issue
Other Approved Reason
```

State:

```text
PENDING → REJECTED
```

If money was already authorized/paid, the system triggers the appropriate refund/reversal workflow rather than silently deleting the order.

---

# 42. Order Preparation

After acceptance/reservation:

```text
ACCEPTED
   ↓
PREPARING
```

Pharmacy staff prepare:

```text
Medicine Items
Quantity
Packaging
Prescription Match
Order Reference
```

The system can provide an internal checklist.

---

# 43. Ready for Pickup

For pickup orders:

```text
PREPARING
   ↓
READY_FOR_PICKUP
```

Patient receives notification.

The pickup screen may show:

```text
Order Number
Pharmacy Name
Pharmacy Location
Ready Status
Pickup Instructions
```

At collection:

```text
READY_FOR_PICKUP
   ↓
COMPLETED
```

The exact identity/verification method can be added as an operational control later.

---

# 44. Out for Delivery

For delivery orders:

```text
PREPARING
   ↓
OUT_FOR_DELIVERY
   ↓
COMPLETED
```

If a separate courier module is not part of the prototype, pharmacy staff may mark fulfillment stages manually.

A future courier integration can introduce:

```text
ASSIGNED
PICKED_UP
IN_TRANSIT
DELIVERED
DELIVERY_FAILED
```

---

# 45. Delivery Address Privacy

The exact delivery address is sensitive.

Rules:

```text
Public Search
→ Never reveal patient address

Pharmacy Order
→ Reveal only to authorized pharmacy staff

Courier
→ Reveal only when necessary for fulfillment

Other Patients
→ Never reveal
```

The address should be handled under least privilege.

---

# 46. Order Completion

After pickup/delivery:

```text
FULFILLMENT CONFIRMED
      ↓
COMPLETED
```

The order should store:

```text
CompletedAt
CompletedBy / Fulfillment Actor
Final Amount
Payment Status
```

---

# 47. Stock Deduction

Inventory must reflect the physical sale correctly.

A safe transactional flow is:

```text
Check Available Stock
       ↓
Reserve Quantity
       ↓
Prepare Order
       ↓
Complete Fulfillment
       ↓
Finalize Stock Deduction
       ↓
Release/close Reservation
```

The exact accounting moment can vary, but the implementation must avoid:

```text
Negative stock caused by race conditions
Double deduction
Reservation leaks
```

---

# 48. Reservation Expiry

If the order remains unpaid or unconfirmed under a reservation-based flow:

```text
Reservation Active
    ↓
Reservation Expired
    ↓
Reserved Quantity Released
    ↓
Stock Becomes Available Again
```

All expiry logic must be enforced server-side.

---

# 49. Cancellation Workflow

Cancellation actors may include:

```text
Patient
Pharmacist / Pharmacy
Admin
System
```

The system must record:

```text
CancelledBy
Reason
CancelledAt
RefundStatus
```

Business rules may depend on the order state.

Example:

```text
PENDING → CANCELLED
```

may be allowed more easily than:

```text
OUT_FOR_DELIVERY → CANCELLED
```

---

# 50. Returns

A complete pharmacy module should model returns separately from simple cancellation.

Possible return flow:

```text
Order Completed
    ↓
Return Request
    ↓
Review
    ↓
Approved / Rejected
    ↓
Return Received (when applicable)
    ↓
Refund Decision
```

The exact eligibility rules should be defined by pharmacy/product policy and applicable requirements.

The system must not automatically make returned medicines sellable again without an authorized stock decision.

---

# 51. Refund Workflow

Recommended statuses:

```text
NOT_REQUIRED
PENDING
PROCESSING
COMPLETED
FAILED
REJECTED
```

A refund should be tied to the original transaction.

Do not simply edit the original order total to represent a refund.

---

# 52. Payment Workflow

Suggested model:

```text
Order Created
     ↓
Payment Initiated
     ↓
Payment Verified
     ↓
Order Accepted / Reserved
     ↓
Fulfillment
     ↓
Completed
     ↓
Settlement
```

Payment success should be verified on the server through the payment provider mechanism in use.

---

# 53. Financial Ledger

The pharmacy module should separate:

```text
Operational Order Status
```

from:

```text
Financial Transaction State
```

A financial transaction can record:

```text
TransactionId
OrderId
PharmacyId
GrossAmount
PlatformFee
Discount
RefundAmount
NetAmount
PaymentStatus
CreatedAt
```

This creates an auditable ledger rather than relying on a single mutable balance field.

---

# 54. Pharmacy Earnings

The pharmacy/owner dashboard may show:

```text
Gross Sales
Platform Commission / Fees
Delivery Fees
Discounts
Refunds
Net Earnings
Pending Settlement
Available for Withdrawal
```

---

# 55. Withdrawal Workflow

```text
Available Earnings
      ↓
Create Withdrawal Request
      ↓
PENDING
      ↓
APPROVED
      ↓
PROCESSING
      ↓
COMPLETED
```

Alternative:

```text
PENDING → REJECTED
```

A rejected withdrawal should retain the reason and audit trail.

---

# 56. Ratings & Reviews

After an eligible completed order, the patient may rate the pharmacy.

Example:

```text
Rating: 1–5 Stars
Review: Optional
```

Rating eligibility should be linked to the actual completed transaction, not merely to the existence of a user account.

---

# 57. Pharmacy Complaints

A patient can submit a complaint tied to an order.

```text
Completed / Relevant Order
        ↓
Complaint
        ↓
Pharmacy Response (when allowed)
        ↓
Admin Review
        ↓
Resolution
```

Complaint statuses:

```text
OPEN
UNDER_REVIEW
RESOLVED
CLOSED
```

The pharmacy cannot delete or manipulate the official complaint record.

---

# 58. Notifications

Pharmacy users should receive notifications for:

```text
New Order
Order Accepted
Order Rejected
Prescription Needs Clarification
Low Stock
Expiry Alert
Order Cancelled
Payment Confirmed
Refund Update
Ready for Pickup
Delivery Status
New Review
New Complaint
Verification Update
Withdrawal Update
Admin Announcement
```

Patients should receive relevant notifications such as:

```text
Order Created
Pharmacy Accepted Order
Order Rejected
Preparing
Ready for Pickup
Out for Delivery
Completed
Refund Update
Prescription Clarification
```

---

# 59. Staff Permissions

Do not give every pharmacy user full access.

Suggested role model:

```text
PharmacyOwner / Manager
    ├── Pharmacy Profile
    ├── Staff Management
    ├── Financials
    ├── Inventory
    ├── Orders
    └── Reports

Pharmacist
    ├── Orders
    ├── Prescription Review
    ├── Inventory (as allowed)
    ├── Medicine Preparation
    └── Patient-facing pharmacy operations

Staff
    ├── Inventory (limited)
    ├── Order Preparation
    └── Fulfillment Tasks
```

The exact role matrix is configurable.

---

# 60. Object-Level Authorization

Every sensitive operation must verify organization ownership or assigned permission.

Example:

```text
Pharmacist A belongs to Pharmacy A.

Request:
GET /api/pharmacies/B/orders/123
```

Must fail unless Pharmacist A has explicit authorization for Pharmacy B.

Likewise, staff member permissions should be checked for every mutation.

---

# 61. Prescription Privacy

Prescription content is sensitive.

Rules:

```text
Patient
→ Own prescriptions

Doctor
→ Prescriptions created by/available to doctor under policy

Pharmacy
→ Only prescription data required to fulfill the order

Admin
→ Only as necessary for authorized support, compliance, or dispute workflows

Other users
→ No access
```

---

# 62. Patient Data Privacy

Pharmacy staff should only see the patient information necessary to fulfill an order.

Examples of information that may be operationally needed:

```text
Patient Name
Contact Information needed for fulfillment
Delivery Address
Order Details
Relevant Prescription Information
```

The complete patient medical chart should not automatically appear in the pharmacy dashboard.

---

# 63. Audit Logging

Important pharmacy events should be logged:

```text
Pharmacy Approved
Pharmacy Suspended
Staff Added
Staff Removed
Medicine Added to Inventory
Price Changed
Stock Adjusted
Prescription Viewed
Order Accepted
Order Rejected
Reservation Created
Reservation Released
Order Cancelled
Refund Created
Refund Completed
Withdrawal Created
Withdrawal Processed
Complaint Updated
```

Audit records should include actor, action, time, and relevant target identifiers.

---

# 64. Security Requirements

The Pharmacy Module handles:

```text
Personal Data
Prescription Data
Potentially Sensitive Health Information
Location Data
Financial Data
Professional Documents
```

Recommended controls:

```text
JWT / secure session handling
Role-based authorization
Organization-level authorization
Object-level authorization
Rate limiting
Secure file storage
Input validation
Output filtering
Server-side price verification
Server-side stock validation
Audit logging
Secure document access
Protection against duplicate order submission
Concurrency controls
```

---

# 65. Inventory Concurrency Rules

For a medicine with:

```text
Available = 1
```

and two patients attempting to buy at the same time:

```text
Request A → reserve 1
Request B → reserve 1
```

Only one transaction may successfully reserve the final unit.

The other must receive a controlled error such as:

```text
STOCK_UNAVAILABLE
```

This logic must be enforced by the backend/database transaction strategy.

---

# 66. Duplicate Submission Protection

Patients can accidentally click checkout twice.

Use a suitable idempotency/deduplication strategy for critical operations where appropriate.

Example concept:

```text
Client Request A
Client Retry A
      ↓
Same logical operation
      ↓
One Order
```

The exact mechanism can be implemented using an idempotency key or another consistent backend approach.

---

# 67. Error Codes

Suggested API error codes:

```text
PHARMACY_NOT_VERIFIED
PHARMACY_SUSPENDED
STAFF_NOT_AUTHORIZED
MEDICINE_NOT_FOUND
MEDICINE_UNAVAILABLE
STOCK_UNAVAILABLE
PRESCRIPTION_REQUIRED
PRESCRIPTION_INVALID
PRESCRIPTION_NEEDS_CLARIFICATION
ORDER_NOT_FOUND
ORDER_INVALID_STATUS
ORDER_EXPIRED
CANNOT_CANCEL_ORDER
REFUND_NOT_ALLOWED
UNAUTHORIZED
FORBIDDEN
PRICE_CHANGED
```

Example response:

```json
{
  "success": false,
  "message": "The requested medicine is no longer available in the selected quantity.",
  "code": "STOCK_UNAVAILABLE"
}
```

---

# 68. Order State Machine — Detailed

```text
                    ┌───────────────┐
                    │    PENDING    │
                    └───────┬───────┘
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
             ACCEPTED    REJECTED   CANCELLED
                 │
                 ↓
            PREPARING
                 │
        ┌────────┴────────┐
        ↓                 ↓
READY_FOR_PICKUP    OUT_FOR_DELIVERY
        │                 │
        │                 ↓
        │              COMPLETED
        ↓
    COMPLETED
```

Invalid transitions should be rejected server-side.

---

# 69. Prescription State Machine — Suggested

```text
CREATED
   ↓
AVAILABLE_TO_PATIENT
   ↓
SUBMITTED_TO_PHARMACY
   ↓
UNDER_REVIEW
   ├── ACCEPTED
   ├── NEEDS_CLARIFICATION
   └── REJECTED
```

The prescription itself and the pharmacy order should remain separate domain concepts.

---

# 70. Inventory State / Stock Flow

```text
RECEIVED
   ↓
ON_HAND
   ↓
RESERVED
   ↓
FULFILLED
   ↓
DEDUCTED
```

Alternative:

```text
ON_HAND
   ↓
EXPIRED / DAMAGED / ADJUSTED
```

The system should record why stock moved rather than simply overwriting the quantity.

---

# 71. Frontend Pages — Pharmacy

Recommended pages:

```text
/Pharmacy/Register
/Pharmacy/Verification
/Pharmacy/Dashboard
/Pharmacy/Profile
/Pharmacy/Staff
/Pharmacy/Medicines
/Pharmacy/Inventory
/Pharmacy/Inventory/{id}
/Pharmacy/LowStock
/Pharmacy/ExpiryAlerts
/Pharmacy/Prescriptions
/Pharmacy/Prescriptions/{id}
/Pharmacy/Orders
/Pharmacy/Orders/{id}
/Pharmacy/Orders/{id}/Prepare
/Pharmacy/Delivery
/Pharmacy/Pickup
/Pharmacy/Reviews
/Pharmacy/Complaints
/Pharmacy/Financials
/Pharmacy/Transactions
/Pharmacy/Refunds
/Pharmacy/Withdrawals
/Pharmacy/Notifications
/Pharmacy/Documents
/Pharmacy/Settings
```

---

# 72. Frontend Pages — Patient Pharmacy Journey

The Patient can use:

```text
/Medicines
/Medicines/Search
/Medicines/{id}
/Pharmacies/Nearby
/Pharmacies/{id}
/Prescriptions
/Prescriptions/{id}
/Prescriptions/{id}/FindMedicines
/Cart
/Checkout
/Orders
/Orders/{id}
```

---

# 73. Suggested Pharmacy API Groups

```text
/api/auth/*
/api/pharmacies/*
/api/pharmacy/staff/*
/api/medicines/*
/api/pharmacy/inventory/*
/api/prescriptions/*
/api/pharmacy/orders/*
/api/pharmacy/returns/*
/api/pharmacy/refunds/*
/api/pharmacy/financials/*
/api/pharmacy/withdrawals/*
/api/pharmacy/reviews/*
/api/pharmacy/complaints/*
/api/notifications/*
```

Route conventions should be standardized across the whole TechCare project.

---

# 74. Example API Endpoints

## Pharmacy Profile

```http
GET  /api/pharmacies/me
PUT  /api/pharmacies/me
```

## Staff

```http
GET    /api/pharmacy/staff
POST   /api/pharmacy/staff/invite
PUT    /api/pharmacy/staff/{id}
DELETE /api/pharmacy/staff/{id}
```

## Medicines

```http
GET /api/medicines
GET /api/medicines/{id}
```

## Inventory

```http
GET  /api/pharmacy/inventory
POST /api/pharmacy/inventory
PUT  /api/pharmacy/inventory/{id}
POST /api/pharmacy/inventory/{id}/adjust
```

## Orders

```http
GET  /api/pharmacy/orders
GET  /api/pharmacy/orders/{id}
POST /api/pharmacy/orders/{id}/accept
POST /api/pharmacy/orders/{id}/reject
POST /api/pharmacy/orders/{id}/prepare
POST /api/pharmacy/orders/{id}/ready
POST /api/pharmacy/orders/{id}/dispatch
POST /api/pharmacy/orders/{id}/complete
POST /api/pharmacy/orders/{id}/cancel
```

## Prescriptions

```http
GET  /api/pharmacy/prescriptions
GET  /api/pharmacy/prescriptions/{id}
POST /api/pharmacy/prescriptions/{id}/review
POST /api/pharmacy/prescriptions/{id}/clarification
```

## Financials

```http
GET  /api/pharmacy/financials
GET  /api/pharmacy/transactions
GET  /api/pharmacy/withdrawals
POST /api/pharmacy/withdrawals
```

---

# 75. Suggested Database Entities

A complete starting model could contain:

```text
User
Role
Pharmacy
PharmacyStaff
PharmacyDocument
PharmacyWorkingHour
PharmacyServiceSettings

Medicine
MedicineIngredient
MedicineManufacturer (optional)
MedicineIdentifier (optional)

PharmacyInventory
InventoryBatch
InventoryAdjustment
InventoryReservation

Prescription
PrescriptionItem
PrescriptionReview
PrescriptionClarification

Cart
CartItem

Order
OrderItem
OrderStatusHistory
OrderDelivery
OrderPickup

Payment
Transaction
Refund
ProviderEarning
Withdrawal

Review
Complaint
Notification
AuditLog
```

The actual relational model can be simplified for the prototype and expanded later.

---

# 76. Important Database Relationships

Conceptually:

```text
User 1 ─── 1 PharmacyStaff ─── N Pharmacy

Pharmacy 1 ─── N PharmacyInventory
Medicine 1 ─── N PharmacyInventory

PharmacyInventory 1 ─── N InventoryBatch
PharmacyInventory 1 ─── N InventoryReservation

Patient 1 ─── N Prescription
Doctor 1 ─── N Prescription
Prescription 1 ─── N PrescriptionItem

Patient 1 ─── N Order
Pharmacy 1 ─── N Order
Order 1 ─── N OrderItem

Order 1 ─── N Payment / Transaction Events
Order 1 ─── N Refund Events
Order 1 ─── N Status History
Order 1 ─── N Complaint / Review (according to business rules)
```

---

# 77. Order History

Never rely only on one `Status` field for accountability.

Recommended:

```text
Current Status
+
OrderStatusHistory
```

Example history:

```text
PENDING              10:00
ACCEPTED             10:02
PREPARING            10:10
READY_FOR_PICKUP     10:35
COMPLETED            11:05
```

Each event stores:

```text
Previous Status
New Status
Actor
Timestamp
Reason (when applicable)
```

---

# 78. Medicine Search — Advanced Flow

A robust search experience can work like:

```text
Patient enters medicine name
          ↓
Normalize query
          ↓
Search generic / brand / ingredient
          ↓
Find active catalog medicines
          ↓
Join eligible pharmacy inventory
          ↓
Filter by available stock
          ↓
Calculate distance
          ↓
Sort by patient preference
          ↓
Show pharmacy options
```

Possible sort options:

```text
Nearest
Lowest Price
Highest Rating
Fastest Availability
```

---

# 79. Distance and Pharmacy Discovery

The platform can use location to compare:

```text
Pharmacy Distance
Delivery Availability
Estimated Delivery Fee
```

Server-side location calculations should be preferred for trusted pricing and filtering.

The public UI should display approximate distance rather than exposing exact private addresses where unnecessary.

---

# 80. Delivery Fee

If delivery pricing is enabled:

```text
Delivery Base Fee
+
Distance Component
+
Any Approved Surcharges
-
Discount
=
Delivery Charge
```

The complete order total then becomes:

```text
Medicine Subtotal
+
Delivery Charge
-
Order Discount
=
Final Total
```

All financial values are calculated on the server.

---

# 81. Order Notes

Patient may provide operational notes such as:

```text
Call before arrival
Leave order at approved location
Preferred contact method
```

Sensitive medical information should not be unnecessarily entered into free-text delivery notes.

---

# 82. Out-of-Stock During Checkout

Possible situation:

```text
Patient sees stock = 2
     ↓
Another order consumes stock
     ↓
Patient checks out quantity = 2
```

Backend must re-check inventory.

Possible result:

```text
STOCK_UNAVAILABLE
```

The frontend then refreshes the available quantity rather than creating an invalid order.

---

# 83. Price Changed During Checkout

Possible situation:

```text
Patient opens product at 70 EGP
     ↓
Pharmacy changes price to 75 EGP
     ↓
Patient submits checkout
```

The backend should revalidate the price.

Depending on product policy:

```text
Refresh Cart
```

or

```text
Confirm New Price
```

Never silently charge a stale client-provided price.

---

# 84. Pharmacy Closed

The system should know pharmacy working hours.

If a patient attempts a delivery/pickup request outside operational rules:

```text
Pharmacy Closed / Service Unavailable
```

The patient can select another pharmacy or another eligible fulfillment time if supported.

---

# 85. Pharmacist Unavailable

The pharmacy organization can have multiple staff members.

This avoids blocking the entire pharmacy because one pharmacist is unavailable.

The order can be assigned to an authorized staff member according to operational rules.

---

# 86. Admin Controls

Admin can manage:

```text
Pharmacy Verification
Pharmacist Verification
Pharmacy Suspension
Medicine Master Catalog
Prescription Requirement Flags
Medicine Activation / Deactivation
Complaints
Reviews
Refund Disputes
Financial Disputes
Audit Logs
```

Admin should not casually edit historical financial records or completed orders without an auditable correction workflow.

---

# 87. Pharmacy Suspension

If a pharmacy is suspended:

```text
Pharmacy Status → SUSPENDED
```

The system should block or restrict:

```text
New Patient Orders
Public Booking / Discovery as an active pharmacy
Inventory Sales
```

Existing orders may follow an explicit resolution workflow.

---

# 88. Document Expiration

Professional/business documents may expire.

Suggested flow:

```text
Document Active
     ↓
Near Expiration Alert
     ↓
Expired
     ↓
Verification Review
```

Depending on the document, an expired critical credential may cause:

```text
Pharmacy / Pharmacist → REQUIRES_UPDATE or SUSPENDED
```

The final policy is controlled by the platform/admin rules.

---

# 89. Staff Invitation Workflow

Pharmacy manager:

```text
Staff
 ↓
Invite
 ↓
Email / Phone Invitation
 ↓
User Accepts
 ↓
Account Linked to Pharmacy
 ↓
Role Assigned
```

Possible status:

```text
INVITED
ACTIVE
SUSPENDED
REMOVED
```

---

# 90. Staff Removal

When staff member leaves:

```text
ACTIVE
  ↓
REMOVED / INACTIVE
```

The system must immediately block their access to the pharmacy's protected resources.

Historical actions remain in the audit trail.

---

# 91. Search Result Rules

Patient-facing pharmacy search should favor:

```text
Verified
Active
Open / Eligible
Has Requested Medicine in Sellable Stock
Supports Requested Fulfillment Method
```

Avoid showing a pharmacy as "available" simply because its profile exists.

---

# 92. Patient Order Tracking

The patient should see:

```text
Order Created
Accepted
Preparing
Ready for Pickup / Out for Delivery
Completed
```

For rejected/cancelled orders:

```text
Reason (when appropriate)
Refund Status
```

---

# 93. Pharmacist Order Actions

The pharmacy user can see action buttons based on the current state.

Example:

```text
PENDING
→ Accept / Reject

ACCEPTED
→ Start Preparation

PREPARING
→ Mark Ready / Dispatch

READY_FOR_PICKUP
→ Complete Pickup

OUT_FOR_DELIVERY
→ Mark Delivered
```

Do not show impossible actions for the current state.

---

# 94. Server-Side Status Transition Validation

Frontend action:

```text
POST /orders/123/complete
```

Backend must verify:

```text
Order exists
Current status is eligible
Actor has permission
Fulfillment requirements are satisfied
Payment status is valid where required
```

Then transition safely.

---

# 95. Testing Scenarios — Registration

```text
Valid pharmacy registration
Duplicate email
Duplicate phone
Missing required pharmacy data
Invalid professional data
Invalid document type
Oversized document
OTP expired
OTP incorrect
OTP retry limit exceeded
```

---

# 96. Testing Scenarios — Verification

```text
Admin approve pharmacy
Admin reject pharmacy
Admin request update
Suspended pharmacy cannot receive new orders
Expired critical document triggers expected restriction
Unauthorized user cannot modify verification
```

---

# 97. Testing Scenarios — Inventory

```text
Add medicine to inventory
Update price
Update stock
Prevent negative stock
Low stock alert
Expired batch becomes unavailable
Stock adjustment requires reason
Unauthorized staff cannot adjust stock
Concurrent stock reservation
Reservation release
```

---

# 98. Testing Scenarios — Prescription

```text
Prescription linked to correct patient
Prescription linked to correct doctor
Pharmacy sees only authorized prescription data
Pharmacist reviews prescription
Prescription accepted
Prescription rejected
Prescription needs clarification
Invalid prescription blocked
Unauthorized user cannot view prescription
```

---

# 99. Testing Scenarios — Orders

```text
Create valid order
Create order with unavailable medicine
Create order with insufficient stock
Price changed before checkout
Pharmacy rejects order
Pharmacy accepts order
Prepare order
Ready for pickup
Dispatch delivery
Complete order
Cancel before fulfillment
Invalid cancellation after restricted state
Duplicate checkout request
```

---

# 100. Testing Scenarios — Payments / Refunds

```text
Successful payment
Failed payment
Webhook / server verification
Payment retry
Refund created
Refund completed
Refund failed
Duplicate refund prevention
Historical transaction integrity
Withdrawal request
Withdrawal rejection
Withdrawal completion
```

---

# 101. Testing Scenarios — Security

```text
Patient accesses another patient's order
Patient accesses pharmacy internal inventory
Pharmacist accesses another pharmacy
Staff accesses restricted financial data
Unauthorized prescription access
Unauthorized document download
Forged price
Forged quantity
Forged order status
Expired JWT
Insufficient role permissions
Concurrent stock race condition
```

---

# 102. Non-Functional Requirements

## Performance

The system should support efficient:

```text
Medicine Search
Nearby Pharmacy Search
Inventory Queries
Order Lists
```

Indexes and pagination should be used where appropriate.

## Reliability

Critical operations such as:

```text
Stock Reservation
Order Creation
Payment Recording
Refund Recording
```

must be transactional and recoverable.

## Security

Sensitive data must be protected through authentication, authorization, validation, secure storage, and auditing.

## Maintainability

The pharmacy module should remain modular and should not tightly couple the medicine catalog to order or payment logic.

---

# 103. Recommended Architecture Direction for Full-Stack .NET

A modular monolith is a strong fit for the project prototype.

Possible conceptual modules:

```text
Identity / Authentication
Patients
Doctors
Nurses
Pharmacy
Laboratory
Blood Donation
Appointments
Orders
Payments
Notifications
Reviews / Complaints
Admin
```

Inside Pharmacy:

```text
Pharmacy
├── Organization
├── Staff
├── Catalog Integration
├── Inventory
├── Prescriptions
├── Orders
├── Fulfillment
├── Financials
└── Reviews / Complaints
```

The implementation can use ASP.NET Core Web API + Entity Framework Core and the team's chosen frontend technology.

---

# 104. Service Boundaries Inside Pharmacy

Keep domain responsibilities separated:

```text
PharmacyService
InventoryService
PrescriptionService
OrderService
FulfillmentService
PaymentIntegrationService
RefundService
NotificationService
```

For example:

```text
OrderService
```
should not contain every inventory rule inline.

Instead:

```text
OrderService
    ↓
InventoryService / Domain Logic
    ↓
Reservation
```

This makes testing and maintenance easier.

---

# 105. Recommended Transactional Operations

Operations that should be carefully transactional include:

```text
Create Order + Reserve Stock
Accept Order + Confirm Reservation
Finalize Fulfillment + Deduct Stock
Release Reservation
Record Refund
Record Financial Settlement
```

The precise transaction boundaries should be decided by the backend architecture.

---

# 106. Background Jobs / Scheduled Processing

A production-oriented implementation may use background processing for:

```text
Reservation Expiration
Low Stock Alerts
Near-Expiry Alerts
Order Reminder Notifications
Document Expiry Alerts
Financial Reconciliation Tasks
```

These are system processes, not UI-only timers.

---

# 107. Search and Filtering Strategy

For an initial version, standard indexed database queries may be enough.

As the catalog grows, search can evolve to:

```text
Full-text Search
Normalized Medicine Name Search
Synonym Handling
Ingredient Search
Fuzzy Search (carefully controlled)
```

Medicine identity should remain anchored to the master catalog even when a search layer is added.

---

# 108. Reporting

Pharmacy manager/admin reports may include:

```text
Orders by Period
Sales by Period
Top Medicines
Low Stock Medicines
Expired Stock
Rejected Orders
Cancellation Rate
Refunds
Average Order Value
Ratings
Complaints
Earnings
```

Reports should use trusted transactional data.

---

# 109. Operational Edge Cases

The system should account for:

```text
Medicine becomes unavailable after patient opens the page
Price changes before checkout
Pharmacy closes after order creation
Staff member is removed while an order is pending
Prescription expires or becomes unavailable
Order is cancelled during preparation
Delivery fails
Pickup not collected
Refund fails
Payment succeeds but order processing fails
Stock reservation expires
Two users reserve the final unit simultaneously
```

Each edge case needs a defined outcome and audit trail.

---

# 110. Full Patient → Pharmacy Journey

```text
Patient Login
      ↓
Open Medicines
      ↓
Search Medicine
      ↓
Select Medicine
      ↓
View Nearby Pharmacies
      ↓
Compare
├── Distance
├── Price
├── Rating
├── Stock
├── Delivery
└── Pickup
      ↓
Select Pharmacy
      ↓
Select Quantity
      ↓
Cart
      ↓
Checkout
      ↓
Choose Delivery / Pickup
      ↓
Enter Delivery Address (if needed)
      ↓
Backend Validates Stock / Price / Rules
      ↓
Payment
      ↓
Order Created
      ↓
Pharmacy Receives Order
      ↓
Pharmacist Reviews
      ↓
Accept / Reject / Clarification
      ↓
Reserve Stock
      ↓
Prepare
      ↓
Pickup Ready / Out for Delivery
      ↓
Completed
      ↓
Patient Rates Pharmacy
```

---

# 111. Full Doctor → Pharmacy Journey

```text
Doctor Consultation
      ↓
Prescription Created
      ↓
Patient Receives Prescription
      ↓
Find My Medicines
      ↓
System Maps Prescription Items
      ↓
Nearby Pharmacy Search
      ↓
Patient Selects Pharmacy
      ↓
Prescription-linked Order
      ↓
Pharmacist Review
      ↓
Accept / Clarify / Reject
      ↓
Stock Reservation
      ↓
Preparation
      ↓
Delivery / Pickup
      ↓
Completed
```

Important: the prescription remains a clinical record and should not be converted into a generic editable shopping list without preserving the clinical source.

---

# 112. Full Pharmacy Internal Journey

```text
Pharmacy Approved
      ↓
Manager Configures Pharmacy
      ↓
Adds / Invites Staff
      ↓
Adds Medicines from Master Catalog
      ↓
Sets Prices
      ↓
Sets Stock
      ↓
Sets Delivery/Pickup
      ↓
Patient Searches
      ↓
Order Arrives
      ↓
Staff / Pharmacist Reviews
      ↓
Prescription Review if required
      ↓
Accept / Reject / Clarify
      ↓
Reserve Stock
      ↓
Prepare
      ↓
Fulfill
      ↓
Complete
      ↓
Transaction Recorded
      ↓
Earnings Updated
      ↓
Rating / Complaint
      ↓
Withdrawal
```

---

# 113. Nurse/Doctor/Patient/Pharmacy Integration

The Pharmacy module should integrate with other TechCare modules through clear contracts.

## Doctor → Pharmacy

```text
Prescription
```

## Patient → Pharmacy

```text
Search
Cart
Order
Payment
Delivery/Pickup
Review
```

## Pharmacy → Patient

```text
Availability
Order Status
Notifications
```

## Admin → Pharmacy

```text
Verification
Catalog Governance
Complaints
Suspension
```

---

# 114. MVP vs Future Enhancements

## MVP / Prototype Focus

```text
Pharmacy Registration
Pharmacist/Staff Account
Verification
Pharmacy Profile
Medicine Master Catalog
Basic Inventory
Medicine Search
Nearby Pharmacies
Prescription-linked Orders
Cart
Checkout
Pickup / Delivery Selection
Order Status
Basic Payment Integration
Ratings
Complaints
Financial Summary
Notifications
Admin Control
```

## Future Enhancements

```text
Advanced Courier Platform
Real-time Driver Tracking
Advanced Route Optimization
Automated Replenishment
Supplier Integration
Advanced Batch/Traceability Workflows
AI Search Assistance
Medication Interaction Support
Advanced Forecasting
Multi-Pharmacy Checkout
Advanced Promotions Engine
Deep Analytics
External Healthcare Integrations
```

Future AI should support safe assistance and information workflows, not autonomous prescribing or unsafe clinical decisions.

---

# 115. Definition of Done — Pharmacy Feature

A feature is not Done because the UI page exists.

The feature should include:

```text
[ ] Frontend UI
[ ] Backend Endpoint / Application Logic
[ ] Database Model / Migration
[ ] Server-side Validation
[ ] Authorization
[ ] Error Handling
[ ] Loading / Empty / Error States
[ ] Business Rules
[ ] Tests
[ ] Integration Test Coverage where appropriate
[ ] Code Review
[ ] API Documentation
[ ] User-facing Documentation where needed
```

---

# 116. Core Business Rules Checklist

```text
[ ] Only approved/active pharmacies can be publicly bookable/sellable.
[ ] Pharmacy staff access is organization-scoped.
[ ] Medicine master data is centralized.
[ ] Pharmacy-specific price and stock are stored separately from master data.
[ ] Available stock is not allowed to become negative.
[ ] Concurrent reservations are handled safely.
[ ] Expired stock is not sellable.
[ ] Critical stock movements are auditable.
[ ] Prescription-required items follow the prescribed workflow.
[ ] Pharmacists cannot silently modify the doctor's prescription.
[ ] Order prices are snapshotted at checkout/order confirmation.
[ ] Client-submitted price and total are never trusted.
[ ] Exact patient delivery addresses are protected.
[ ] Historical transactions are traceable.
[ ] Refunds are separate financial events.
[ ] Ratings are tied to eligible completed transactions.
[ ] Complaints cannot be deleted by pharmacy staff.
[ ] Staff removal immediately affects access.
[ ] Critical status transitions are server-validated.
```

---

# 117. Final Pharmacy Workflow Diagram

```text
                           TECHCARE PHARMACY
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
             PHARMACY OWNER                    PHARMACIST / STAFF
                    │                               │
                    └───────────────┬───────────────┘
                                    ↓
                         Registration / Login
                                    ↓
                         OTP / Account Verification
                                    ↓
                         Pharmacy Verification
                                    ↓
                               APPROVED
                                    ↓
                         Pharmacy Profile Setup
                                    ↓
                         Add Authorized Staff
                                    ↓
                         Medicine Master Catalog
                                    ↓
                           Inventory Management
                                    ↓
                         Prices / Stock / Availability
                                    ↓
                 ┌──────────────────┴──────────────────┐
                 │                                     │
          Patient Search                         Doctor Prescription
                 │                                     │
                 └──────────────────┬──────────────────┘
                                    ↓
                              Select Pharmacy
                                    ↓
                              Cart / Request
                                    ↓
                         Checkout / Validation
                                    ↓
                         Stock Reservation
                                    ↓
                           Pharmacy Review
                                    ↓
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
                 ACCEPT          REJECT       CLARIFICATION
                    │                               │
                    ↓                               ↓
                PREPARING ←─────────────────────────
                    │
             ┌──────┴───────┐
             ↓              ↓
       PICKUP READY    OUT FOR DELIVERY
             │              │
             └──────┬───────┘
                    ↓
                COMPLETED
                    ↓
             Payment Settlement
                    ↓
              Pharmacy Earnings
                    ↓
           Rating / Complaint
                    ↓
               Withdrawal
```

---

# 118. Final End-to-End Pharmacy Journey

```text
1. Pharmacy registers.
2. Pharmacist account is created or linked.
3. OTP/contact verification is completed.
4. Pharmacy submits professional/business documents.
5. Admin reviews and approves the pharmacy.
6. Pharmacy manager configures profile, hours, location, delivery and pickup.
7. Authorized staff are added.
8. Medicines are selected from the master catalog.
9. Pharmacy configures price, stock and availability.
10. Inventory may be tracked by batch and expiry where supported.
11. Patient searches for a medicine.
12. System finds verified pharmacies with valid sellable stock.
13. Patient compares price, distance, rating and fulfillment options.
14. Patient either selects medicines directly or enters through a doctor prescription.
15. Patient chooses one pharmacy.
16. Patient adds eligible medicines to cart/order.
17. Backend validates medicine, price, prescription requirements and stock.
18. Backend calculates the final amount.
19. Backend reserves stock safely.
20. Payment is initiated and verified where applicable.
21. Pharmacy receives the order.
22. Pharmacist/staff reviews the order.
23. Prescription is reviewed when required.
24. Pharmacy accepts, rejects, or requests clarification according to workflow rules.
25. Accepted order moves to preparation.
26. Staff prepare the medicines.
27. Order becomes ready for pickup or out for delivery.
28. Patient receives status notifications.
29. Order is fulfilled.
30. Order becomes completed.
31. Inventory is finalized and the reservation is closed.
32. Financial transaction and provider earnings are recorded.
33. Patient may rate the pharmacy.
34. Patient may open a complaint when appropriate.
35. Pharmacy can monitor earnings and transactions.
36. Pharmacy requests withdrawal.
37. Withdrawal moves through the configured financial process.
38. All sensitive and critical events remain auditable.
```

---

# 119. Module Ownership in the Five-Person Team

According to the proposed team split:

```text
Member 1 → Authentication + Patient
Member 2 → Doctor
Member 3 → Nurse
Member 4 → Pharmacy / Pharmacist
Member 5 → Laboratory + Blood Donor
```

The **Pharmacy owner** is accountable for the Pharmacy module end-to-end:

```text
Database
    ↓
ASP.NET Core Backend
    ↓
APIs
    ↓
Frontend
    ↓
Validation
    ↓
Authorization
    ↓
Testing
    ↓
Integration
```

This does not mean the member works in isolation. Other members should review pull requests and integration points.

---

# 120. Pharmacy Module Final Checklist

### Identity & Verification

```text
[ ] Pharmacy registration
[ ] Pharmacist registration/linking
[ ] OTP/contact verification
[ ] Professional documents
[ ] Admin approval
[ ] Suspension / re-verification
```

### Pharmacy Operations

```text
[ ] Profile
[ ] Location
[ ] Hours
[ ] Delivery
[ ] Pickup
[ ] Staff
```

### Medicine Operations

```text
[ ] Master catalog
[ ] Pharmacy inventory
[ ] Prices
[ ] Stock
[ ] Reservation
[ ] Low stock
[ ] Expiry
[ ] Adjustments
```

### Clinical/Prescription Integration

```text
[ ] Doctor prescription
[ ] Prescription review
[ ] Clarification workflow
[ ] Controlled substitution workflow if enabled
[ ] Prescription privacy
```

### Orders

```text
[ ] Search
[ ] Cart
[ ] Checkout
[ ] Price snapshot
[ ] Accept
[ ] Reject
[ ] Preparing
[ ] Pickup
[ ] Delivery
[ ] Complete
[ ] Cancel
[ ] Return
[ ] Refund
```

### Financials

```text
[ ] Payment
[ ] Transactions
[ ] Fees / Commission
[ ] Refunds
[ ] Earnings
[ ] Withdrawals
[ ] Audit trail
```

### User Experience

```text
[ ] Patient medicine search
[ ] Nearby pharmacy results
[ ] Filters
[ ] Order tracking
[ ] Notifications
[ ] Rating
[ ] Complaints
```

### Security

```text
[ ] Role authorization
[ ] Organization-level authorization
[ ] Object-level authorization
[ ] Secure documents
[ ] Prescription privacy
[ ] Address privacy
[ ] Input validation
[ ] Rate limiting
[ ] Concurrency controls
[ ] Audit logging
```

---

# Conclusion

The Pharmacy Module in TechCare should be implemented as a complete **pharmacy organization + pharmacist/staff + medicine catalog + inventory + prescription + ordering + fulfillment + financial** domain, not merely as a page where a patient searches for a medicine.

The key architecture principle is:

```text
Central Medicine Catalog
        +
Pharmacy-Specific Inventory
        +
Verified Pharmacy Organization
        +
Pharmacist / Staff Permissions
        +
Prescription-aware Order Flow
        +
Transactional Stock Reservation
        +
Auditable Financial Events
        +
Privacy / Role Security
        =
Complete TechCare Pharmacy Module
```

This workflow is intended to be the main functional reference for implementation, database design, API design, frontend screens, QA scenarios, and future integration with the Patient, Doctor, Admin, Payment, Notification, and broader TechCare modules.
