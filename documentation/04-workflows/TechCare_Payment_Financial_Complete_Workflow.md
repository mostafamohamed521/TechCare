# TechCare — Payment & Financial Complete Workflow

> **Document Type:** Cross-System Workflow Specification  
> **Module:** Payment & Financial Management  
> **Platform:** TechCare Healthcare Platform  
> **Architecture Context:** ASP.NET Core Web API + Frontend + Relational Database  
> **Purpose:** Define the complete financial lifecycle shared by Patient, Doctor, Nurse, Pharmacy, Laboratory, Blood Donor, and Admin modules.

---

# 1. Module Overview

The Payment & Financial module is a shared platform service responsible for handling every money-related action in TechCare without coupling financial logic directly to a single role.

The module must support:

- Service price calculation.
- Distance-based pricing where applicable.
- Cart and checkout totals.
- Payment creation.
- Payment gateway integration through an abstraction layer.
- Payment status tracking.
- Payment verification.
- Idempotent callbacks/webhooks.
- Failed payment recovery.
- Refunds.
- Partial refunds.
- Provider earnings.
- Platform revenue.
- Fees.
- Financial ledger.
- Provider withdrawals.
- Withdrawal approval and rejection.
- Financial reconciliation.
- Financial audit logs.
- Customer receipts and invoices.
- Notifications for financial events.
- Fraud and abuse signals.
- Reporting and financial dashboards.

The design must prevent the financial state of the platform from depending solely on the frontend or on a gateway redirect response.

---

# 2. Core Financial Principle

The most important rule is:

> **The frontend never decides that a payment is successful. The backend determines payment status from trusted server-side payment verification and/or signed gateway webhook events.**

A browser returning to `/payment/success` is not enough to mark an order or appointment as paid.

The backend should maintain its own financial records and map gateway events into internal states.

---

# 3. Supported Business Domains

The same payment engine serves multiple domains.

## 3.1 Doctor Services

Examples:

- Home consultation.
- In-person consultation where supported.
- Other approved medical services.

Typical flow:

```text
Patient selects Doctor
      ↓
Service selected
      ↓
Distance calculated if applicable
      ↓
Price displayed
      ↓
Booking created
      ↓
Doctor accepts
      ↓
Payment authorization / payment required according to policy
      ↓
Payment confirmed
      ↓
Appointment proceeds
      ↓
Service completed
      ↓
Provider earning finalized
```

## 3.2 Nurse Services

Examples:

- Home nursing visit.
- Wound care.
- Injection service where permitted by platform policy.
- Other approved nursing services.

Pricing may include:

```text
Base Service Price
+ Distance Charge
+ Optional approved service fee
- Discounts
= Customer Payable Amount
```

## 3.3 Pharmacy Orders

Typical flow:

```text
Patient chooses pharmacy
      ↓
Adds medicines to cart
      ↓
Stock checked
      ↓
Subtotal calculated
      ↓
Delivery fee calculated if delivery
      ↓
Discounts applied
      ↓
Total locked as order price snapshot
      ↓
Payment
      ↓
Pharmacy accepts / prepares
      ↓
Order delivered or picked up
      ↓
Completion
```

## 3.4 Laboratory Services

Typical flow:

```text
Patient selects lab/test
      ↓
Collection mode selected
      ↓
Test price calculated
      ↓
Home collection fee if applicable
      ↓
Booking confirmed
      ↓
Payment
      ↓
Sample collected / lab visit
      ↓
Testing completed
      ↓
Result published
      ↓
Financial transaction finalized
```

## 3.5 Blood Donation

Blood donation is not a commercial blood marketplace.

Therefore:

- The platform must not treat blood units as products for sale.
- Donor rewards, when enabled, are points/non-cash benefits governed by product policy.
- Any monetary payment associated with ordinary blood donation must not be modeled as selling blood.
- Donation-related financial records should be separated from normal provider service earnings.

---

# 4. Financial Actors

## 4.1 Patient

Can:

- View prices.
- Initiate payment.
- View payment methods available to them.
- Complete or retry payment.
- View receipts.
- View refunds.
- View transaction history.
- Request cancellation/refund according to business rules.

## 4.2 Doctor

Can:

- Define approved service prices.
- View gross completed earnings.
- View platform fees where applicable.
- View net earnings.
- View available balance.
- Request withdrawals.
- View transactions and payout history.

Doctor does not manually edit a finalized payment amount.

## 4.3 Nurse

Same financial capabilities as a Doctor for nursing services.

## 4.4 Pharmacy Owner / Manager

Can:

- Configure pharmacy product prices.
- View order revenue.
- View platform fees.
- View refunds affecting orders.
- View payout balance.
- Request withdrawals.
- View financial reports for the pharmacy organization.

## 4.5 Laboratory Owner / Manager

Can:

- Configure test/service prices.
- View revenue.
- View fees.
- View refunds.
- Request withdrawals.
- View financial reports.

## 4.6 Admin / Finance Admin

Can:

- Monitor payments.
- Review suspicious transactions.
- Approve financial exceptions.
- Process refunds according to permissions.
- Review withdrawals.
- Manage disputes.
- Reconcile gateway transactions.
- View platform revenue.
- Freeze provider payouts.
- Review audit logs.

## 4.7 Payment Gateway

External system responsible for processing payment instruments.

The gateway must be abstracted behind an interface such as:

```text
IPaymentGateway
```

The rest of the application should not directly depend on provider-specific SDK objects.

---

# 5. Payment Architecture

Recommended logical components:

```text
Frontend
   ↓
Checkout API
   ↓
Payment Application Service
   ↓
Payment Domain
   ├── Pricing Engine
   ├── Payment Orchestrator
   ├── Ledger Service
   ├── Refund Service
   ├── Withdrawal Service
   ├── Reconciliation Service
   └── Notification Service
           ↓
      Payment Gateway Adapter
           ↓
     External Gateway
```

The payment module should be designed so a gateway can be replaced without rewriting the business rules.

---

# 6. Separation of Concepts

Do not use one table or one status field to represent all financial concepts.

At minimum distinguish:

1. Business Order / Appointment / Booking.
2. Payment Attempt.
3. Payment Transaction.
4. Refund.
5. Ledger Entry.
6. Provider Payout / Withdrawal.
7. Financial Reconciliation Record.
8. Gateway Event / Webhook Event.
9. Invoice / Receipt.

Example:

```text
Pharmacy Order
   │
   ├── Payment Attempt #1 → FAILED
   ├── Payment Attempt #2 → SUCCEEDED
   │
   ├── Ledger Entry → CUSTOMER_PAYMENT
   ├── Ledger Entry → PLATFORM_FEE
   └── Ledger Entry → PROVIDER_PAYABLE
```

This is much safer than simply storing:

```text
Order.IsPaid = true
```

---

# 7. Money Representation

Never use floating point types for monetary values.

Recommended backend representation:

```csharp
decimal Amount
```

or an integer minor-unit representation if the chosen financial architecture requires it.

For an early ASP.NET Core implementation, `decimal` with strict currency rules is acceptable.

Every financial amount must have an explicit currency.

Example:

```text
Amount = 250.00
Currency = EGP
```

Never assume currency purely from UI locale.

---

# 8. Price Calculation Engine

The pricing engine is responsible for calculating the payable amount before payment.

## 8.1 Doctor/Nurse Example

```text
Base Price                 = 200.00
Distance                   = 8 km
Included Distance          = 3 km
Billable Distance          = 5 km
Price Per Extra KM         = 10.00
Distance Charge            = 50.00
Service Fee                = 0.00
Discount                   = 20.00
---------------------------------
Customer Total             = 230.00
```

## 8.2 Laboratory Example

```text
Test Price                 = 350.00
Home Collection Fee        = 50.00
Discount                   = 0.00
---------------------------------
Total                      = 400.00
```

## 8.3 Pharmacy Example

```text
Medicine A                 = 100.00
Medicine B                 = 150.00
Subtotal                   = 250.00
Delivery                   = 30.00
Discount                   = 25.00
---------------------------------
Total                      = 255.00
```

---

# 9. Price Snapshot Rule

A booking or order must store the financially relevant values at confirmation time.

Example fields:

```text
BaseAmountSnapshot
DistanceAmountSnapshot
ServiceFeeSnapshot
DeliveryFeeSnapshot
DiscountSnapshot
TaxSnapshot                (only if applicable)
TotalAmountSnapshot
Currency
PricingVersion
```

Why?

Because a provider may change their current price after a patient has already booked.

The historical transaction must remain reproducible.

---

# 10. Price Change Rule

Changing the provider's current service price must not silently mutate:

- Existing completed orders.
- Existing completed appointments.
- Existing payment transactions.
- Historical invoices.
- Existing provider earnings.

Only future transactions should use the new price.

---

# 11. Payment Lifecycle

Recommended payment status model:

```text
CREATED
   ↓
PENDING
   ├──→ REQUIRES_ACTION
   │       ↓
   │   PROCESSING
   │
   ├──→ FAILED
   │
   └──→ CANCELLED

PROCESSING
   ├──→ SUCCEEDED
   ├──→ FAILED
   └──→ UNKNOWN / REVIEW
```

For business-level financial state, additional concepts may be required:

```text
AUTHORIZED
CAPTURED
PARTIALLY_REFUNDED
REFUNDED
DISPUTED
```

Exact gateway mapping should be handled by the adapter.

---

# 12. Payment Attempt vs Payment Transaction

A user may attempt to pay multiple times.

Example:

```text
Payment Attempt #1
Card declined
↓
FAILED

Payment Attempt #2
3D Secure / verification required
↓
REQUIRES_ACTION

Payment Attempt #3
Payment succeeds
↓
SUCCEEDED
```

Do not overwrite attempt #1 with attempt #3.

Every attempt must remain auditable.

---

# 13. Checkout Workflow

## Step 1 — User Reviews Order / Booking

Frontend requests a fresh server-side quote.

Example:

```http
POST /api/v1/payments/quotes
```

Server validates:

- Resource exists.
- User has access.
- Item/service is available.
- Provider is active.
- Provider serves the location if relevant.
- Stock exists for pharmacy products.
- Appointment slot is still available.
- Price is valid.
- Discounts are valid.

## Step 2 — Quote Generated

Response contains:

```json
{
  "quoteId": "...",
  "currency": "EGP",
  "subtotal": 250.00,
  "distanceFee": 30.00,
  "deliveryFee": 20.00,
  "discount": 25.00,
  "total": 275.00,
  "expiresAt": "..."
}
```

The quote is not a payment confirmation.

## Step 3 — Checkout Session Created

Server creates a payment intent/session using the trusted amount.

## Step 4 — Customer Completes Payment

Frontend redirects/opens the gateway payment UI or uses the gateway's approved client flow.

## Step 5 — Gateway Result

Backend receives success/failure through the provider's trusted mechanisms.

## Step 6 — Payment Verified

Backend verifies:

- Reference.
- Amount.
- Currency.
- Order/payment association.
- Signature/webhook integrity.
- Duplicate event state.

## Step 7 — Financial Commit

After successful verification:

```text
Payment → SUCCEEDED
Business resource → PAID / appropriate state
Ledger entries → CREATED
Receipt → AVAILABLE
Notification → SENT/QUEUED
```

---

# 14. Quote Expiration

Quotes should be time-bound when required for price consistency.

When a quote expires:

```text
Quote → EXPIRED
```

The user must request a new quote before payment.

Never trust an old client-side total.

---

# 15. Payment Intent Creation

Recommended conceptual entity:

```text
Payment
```

Fields may include:

```text
Id
CustomerUserId
BusinessObjectType
BusinessObjectId
ProviderOrganizationId / ProviderUserId
Amount
Currency
Status
Gateway
GatewayPaymentReference
InternalReference
IdempotencyKey
CreatedAt
UpdatedAt
ExpiresAt
PaidAt
```

Do not store raw card numbers, CVV, or other sensitive card data in TechCare.

The exact payment-data boundary must follow the gateway's compliant hosted/tokenized flow.

---

# 16. Idempotency

Idempotency is mandatory for payment creation and financial state transitions.

Example client header:

```http
Idempotency-Key: 7d7b5a...unique-request...
```

If the same request reaches the API twice:

```text
Request #1 → creates payment
Request #2 → returns existing result
```

It must not create two financial obligations.

---

# 17. Webhook Processing

Payment gateway webhooks may be retried or delivered more than once.

The server must persist every relevant webhook event.

Recommended entity:

```text
PaymentWebhookEvent
```

Fields:

```text
Id
Gateway
ExternalEventId
EventType
ReceivedAt
SignatureValid
ProcessedAt
ProcessingStatus
PayloadHash
PaymentId
ErrorCode
ErrorMessage
```

Unique constraint:

```text
(Gateway, ExternalEventId)
```

This protects against duplicate webhook processing.

---

# 18. Webhook Processing Sequence

```text
Gateway sends webhook
        ↓
Receive endpoint
        ↓
Validate signature/authenticity
        ↓
Persist raw/minimally required event metadata
        ↓
Check duplicate event ID
        ↓
Map gateway event
        ↓
Locate internal payment
        ↓
Verify amount/currency/reference
        ↓
Apply valid state transition
        ↓
Create ledger entry if needed
        ↓
Update business object
        ↓
Queue notification
        ↓
Mark webhook processed
```

Webhook processing should be resilient and retryable.

---

# 19. Payment Success Rules

A payment can move to `SUCCEEDED` only when backend verification confirms it.

Validation must include:

```text
Payment exists
AND
Gateway event/reference is valid
AND
Amount matches expected amount
AND
Currency matches expected currency
AND
Order/booking matches payment
AND
Event has not already been applied
```

---

# 20. Amount Mismatch

Example:

Expected:

```text
500 EGP
```

Gateway reports:

```text
450 EGP
```

The system must NOT silently mark the payment successful for 500 EGP.

Recommended state:

```text
REVIEW / PAYMENT_MISMATCH
```

Admin/finance workflow investigates the discrepancy.

---

# 21. Currency Mismatch

Expected:

```text
EGP
```

Received:

```text
USD
```

Reject or hold according to gateway/business rules.

Never infer conversion after the fact without an explicit approved conversion policy.

---

# 22. Failed Payment

When payment fails:

- Store the failed attempt.
- Preserve failure code where permitted.
- Do not expose sensitive provider details unnecessarily.
- Keep the order/booking in its appropriate unpaid state.
- Notify the patient.
- Allow retry where business rules allow.

Example:

```text
Payment Attempt → FAILED
Order            → PAYMENT_PENDING
```

---

# 23. Retry Payment

Retry should create a new payment attempt or new gateway intent according to the integration model.

Never mutate an old successful transaction into another attempt.

Example:

```text
Order #1001
 ├── Attempt #1 → FAILED
 ├── Attempt #2 → FAILED
 └── Attempt #3 → SUCCEEDED
```

Only one successful payment should settle the business obligation.

---

# 24. Duplicate Payment Protection

Critical example:

```text
User double-clicks Pay
        ↓
Two HTTP requests
        ↓
Two payment creation requests
```

Protection layers:

1. Idempotency key.
2. Unique internal payment reference where applicable.
3. Server-side business constraint.
4. Transactional state transition.
5. Post-payment reconciliation.

---

# 25. Overpayment

The platform must detect if received amount exceeds expected amount.

Possible handling:

```text
PAYMENT_OVERAGE_REVIEW
```

Do not automatically convert the difference into provider earnings.

---

# 26. Underpayment

If received amount is below expected:

```text
UNDERPAID_REVIEW
```

The related service/order should not be considered fully settled unless business rules explicitly permit partial settlement.

---

# 27. Business Payment Policy per Module

Different domains may have different payment timing.

## Doctor

Possible policy:

```text
Booking accepted
→ Payment required
→ Payment confirmed
→ Appointment confirmed
```

## Nurse

Same general pattern, with support for cancellation/no-show policy.

## Pharmacy

```text
Checkout
→ Payment
→ Pharmacy processing
→ Delivery/Pickup
```

Alternative cash-on-delivery, if product policy ever enables it, must be modeled as a separate payment method and settlement state. Do not mix it with online payment semantics.

## Laboratory

```text
Booking
→ Payment
→ Sample Collection / Visit
```

Exact policies should remain configurable by business rules rather than hard-coded in controllers.

---

# 28. Cancellation Before Payment

Example:

```text
Booking Created
↓
Not Yet Paid
↓
Cancelled
```

No refund should be created because no successful payment exists.

---

# 29. Cancellation After Successful Payment

Example:

```text
Payment SUCCEEDED
↓
Booking CANCELLED
↓
Refund Policy Evaluated
↓
Refund Created if eligible
```

Refund amount depends on the approved cancellation policy.

---

# 30. Refund Workflow

Refund should be represented as its own financial object.

```text
Refund
```

Suggested fields:

```text
Id
PaymentId
Amount
Currency
Reason
Status
GatewayRefundReference
RequestedByUserId
ApprovedByAdminId
CreatedAt
ProcessedAt
FailureReason
```

---

# 31. Refund States

Recommended:

```text
REQUESTED
   ↓
UNDER_REVIEW
   ├──→ REJECTED
   │
   └──→ APPROVED
          ↓
       PROCESSING
          ├──→ SUCCEEDED
          └──→ FAILED
```

An implementation may simplify states where the gateway provides immediate confirmation.

---

# 32. Full Refund

```text
Original Payment = 500
Refund = 500
```

After successful refund:

```text
Payment → REFUNDED
```

Corresponding ledger entries must reverse the correct financial obligations.

---

# 33. Partial Refund

```text
Original Payment = 500
Refund = 200
Remaining settled amount = 300
```

Payment state becomes:

```text
PARTIALLY_REFUNDED
```

until the entire refundable amount is reversed.

---

# 34. Double Refund Protection

Never allow:

```text
Refund #1 = 500
Refund #2 = 500
```

against a single 500 payment.

The server must validate:

```text
Sum(Completed Refunds) + Requested Refund <= Original Captured Amount
```

Use transactional locking/concurrency-safe logic.

---

# 35. Refund Failure

A refund request can fail due to gateway/provider issues.

In that case:

```text
Refund → FAILED
```

The original payment should not be marked fully refunded unless the refund actually succeeded.

Admin should be able to retry safely using idempotency.

---

# 36. Refund Reason Codes

Recommended controlled values:

```text
CUSTOMER_CANCELLATION
PROVIDER_CANCELLATION
PROVIDER_NO_SHOW
CUSTOMER_NO_SHOW
SERVICE_NOT_DELIVERED
ORDER_REJECTED
OUT_OF_STOCK
LAB_CANCELLATION
DUPLICATE_PAYMENT
PAYMENT_ERROR
ADMIN_ADJUSTMENT
OTHER_APPROVED_REASON
```

Free-text notes can supplement but should not replace reason codes.

---

# 37. Cancellation Policy Engine

The platform should not hard-code refund behavior across controllers.

Example conceptual interface:

```text
IRefundPolicyService
```

Input:

```text
BusinessType
CancellationActor
CancellationTime
ServiceState
PaymentState
```

Output:

```text
RefundEligible
RefundAmount
PlatformFeeTreatment
ProviderEarningTreatment
ReasonCode
```

---

# 38. Provider Earnings

Provider earnings should be derived from completed/eligible financial transactions, not simply copied from the booking amount.

Example:

```text
Customer Payment          = 500
Platform Fee              = 50
Other approved adjustment = 0
--------------------------------
Provider Gross Payable    = 450
```

The exact fee policy is a business configuration.

---

# 39. Provider Financial States

Suggested balances:

```text
Pending Balance
Available Balance
Withdrawn Amount
Refund Adjustment
Disputed/Held Amount
```

Example:

```text
Appointment Completed
        ↓
Pending Provider Earning
        ↓
Eligibility / Settlement Rule
        ↓
Available Balance
        ↓
Withdrawal Request
        ↓
Processing
        ↓
PAID
```

---

# 40. Why Pending and Available Must Be Separate

A provider may have completed a service while its payment is still:

- Under refund review.
- Under dispute.
- Awaiting settlement confirmation.
- Subject to an operational hold.

Therefore:

```text
Pending != Available
```

---

# 41. Provider Earning Entity

Suggested entity:

```text
ProviderEarning
```

Fields:

```text
Id
ProviderUserId / OrganizationId
BusinessType
BusinessObjectId
PaymentId
GrossAmount
PlatformFee
OtherFees
RefundAdjustment
NetAmount
Currency
Status
AvailableAt
CreatedAt
UpdatedAt
```

---

# 42. Ledger System

The ledger is the financial history of the platform.

Suggested entity:

```text
LedgerEntry
```

Possible entry types:

```text
CUSTOMER_PAYMENT
PLATFORM_FEE
PROVIDER_EARNING
REFUND
ADJUSTMENT
WITHDRAWAL_HOLD
WITHDRAWAL_DEBIT
CHARGEBACK
REVERSAL
```

---

# 43. Ledger Rules

A ledger entry should be immutable after posting.

If a financial correction is necessary:

```text
Original Entry
      ↓
Reversal / Adjustment Entry
```

Do not silently edit historical accounting facts.

---

# 44. Ledger Reference Rule

Every ledger entry should point back to a source transaction where possible.

Example:

```text
LedgerEntry.SourceType = Payment
LedgerEntry.SourceId   = PAY-123
```

This enables:

```text
Payment
→ Refund
→ Ledger
→ Provider Earning
→ Withdrawal
```

traceability.

---

# 45. Financial Transaction Example

For a 600 EGP doctor service:

```text
Customer pays 600

Ledger:
+600  CUSTOMER_PAYMENT
-60   PLATFORM_FEE
+540  PROVIDER_PAYABLE
```

The representation can be implemented using signed entries or a debit/credit model. Pick one accounting convention and use it consistently.

---

# 46. Accounting Consistency

The system should be able to answer:

```text
Where did this money come from?
Where did it go?
Which customer paid it?
Which service generated it?
Which provider earned it?
What fee was retained?
Was any part refunded?
Was anything withdrawn?
```

---

# 47. Withdrawal Workflow

Provider opens:

```text
Financial Dashboard
```

then:

```text
Available Balance
      ↓
Withdraw
      ↓
Enter amount
      ↓
Select verified payout method
      ↓
Server validates eligibility
      ↓
Withdrawal REQUESTED
```

---

# 48. Withdrawal Validation

Server must validate:

- Provider account is active.
- Provider is verified.
- Provider is not financially blocked.
- Amount > 0.
- Amount <= available balance.
- Payout destination belongs to/has been verified for the provider according to policy.
- No conflicting withdrawal is already reserving the same funds.
- Currency matches.

---

# 49. Withdrawal State Machine

```text
REQUESTED
   ↓
UNDER_REVIEW
   ├──→ REJECTED
   │
   └──→ APPROVED
          ↓
      PROCESSING
          ├──→ COMPLETED
          └──→ FAILED
```

Rejected/failed withdrawals must release any temporary hold correctly.

---

# 50. Withdrawal Reservation

When a provider requests a withdrawal, the amount may need to be reserved.

Example:

```text
Available Balance = 2,000
Withdrawal Request = 1,500
```

After reservation:

```text
Available spendable balance = 500
Reserved withdrawal = 1,500
```

This prevents two simultaneous withdrawal requests from consuming the same balance.

---

# 51. Concurrent Withdrawal Scenario

Without locking:

```text
Balance = 1,000

Request A asks for 800
Request B asks for 800

Both read 1,000
Both are approved

Result = invalid 1,600 withdrawal
```

Solution:

- Transaction.
- Row/application lock as appropriate.
- Atomic balance reservation.
- Unique/invariant checks.

---

# 52. Payout Destination Management

Provider payout details should be separated from the normal profile.

Example:

```text
PayoutMethod
```

Possible fields:

```text
Id
OwnerUserId / OrganizationId
Type
MaskedDestination
ProviderReference
Status
IsDefault
VerifiedAt
CreatedAt
```

Sensitive financial information should not be exposed to unrelated users.

---

# 53. Pharmacy Organization Earnings

Pharmacy earnings belong to the pharmacy organization, not automatically to the individual pharmacist.

Correct:

```text
Patient payment
      ↓
Pharmacy organization earning
      ↓
Pharmacy payout account
```

Individual staff members should not automatically receive pharmacy order revenue unless a separate approved incentive system exists.

---

# 54. Laboratory Organization Earnings

Same model:

```text
Patient payment
      ↓
Laboratory organization earning
      ↓
Laboratory payout account
```

Responsible professionals and collectors are employees/staff of the organization, not necessarily payout owners.

---

# 55. Invoice / Receipt Workflow

For a successful payment:

```text
Payment Confirmed
      ↓
Receipt/Invoice generated
      ↓
Available in Patient account
```

Receipt should contain enough information to explain the transaction without exposing unnecessary sensitive medical data.

Recommended fields:

```text
Receipt Number
Date
Customer
Business Type
Provider / Organization
Description
Subtotal
Fees
Discount
Total
Currency
Payment Reference
Payment Status
```

---

# 56. Receipt Privacy

A payment receipt should not expose:

- Unnecessary patient medical history.
- Diagnosis details.
- Private clinical notes.
- Full internal gateway payload.
- Sensitive provider payout details.

Financial documentation should contain the minimum necessary data.

---

# 57. Patient Financial Dashboard

Suggested sections:

```text
Payments
Orders
Appointments
Laboratory Payments
Refunds
Receipts
Transaction History
```

Each row:

```text
Reference
Date
Service
Provider
Amount
Status
Receipt
```

---

# 58. Provider Financial Dashboard

Doctor/Nurse:

```text
Today/Period Revenue
Pending Earnings
Available Balance
Platform Fees
Refund Adjustments
Withdrawals
Transaction History
```

Pharmacy/Lab:

```text
Organization Revenue
Pending Settlement
Available Balance
Orders/Bookings
Refunds
Withdrawals
```

---

# 59. Admin Financial Dashboard

Suggested KPIs:

```text
Gross Transaction Value
Successful Payments
Failed Payments
Refunds
Net Platform Fees
Provider Payables
Pending Withdrawals
Completed Withdrawals
Disputed Transactions
Payment Mismatches
```

Filters:

- Date range.
- Business domain.
- Provider.
- Organization.
- Payment method.
- Payment status.
- Refund status.
- Withdrawal status.
- Currency.

---

# 60. Payment Search

Admin should be able to search by:

```text
Internal Payment ID
Gateway Reference
Order ID
Booking ID
Patient ID
Provider ID
Organization ID
Receipt Number
```

Search must respect admin permissions.

---

# 61. Payment Methods

Use an abstraction:

```text
PaymentMethodType
```

Examples may include:

```text
CARD
BANK_TRANSFER
DIGITAL_WALLET
CASH
OTHER_APPROVED_METHOD
```

Only methods actually integrated and approved by product rules should be exposed to users.

---

# 62. Payment Method Tokenization

TechCare should prefer tokenized/hosted payment flows.

The application should store gateway references/tokens where required, not raw payment instrument data.

Never store CVV.

---

# 63. Payment Security

Required controls include:

- HTTPS.
- Server-side authorization.
- Input validation.
- Idempotency.
- Signed webhook verification.
- Replay protection.
- Rate limiting.
- Audit logging.
- Least privilege.
- Object-level authorization.
- No client-controlled amount.
- No trust in frontend payment success state.

---

# 64. No Client-Controlled Amount

Bad request:

```json
{
  "orderId": 100,
  "amount": 1
}
```

The API must not simply charge `1 EGP` because the client supplied it.

Instead:

```text
Order 100
↓
Server loads authoritative order
↓
Server calculates/reads trusted current payable amount
↓
Server creates payment for that amount
```

---

# 65. Object-Level Authorization

Example:

Doctor A must not be able to call:

```http
GET /api/v1/providers/doctor-b/earnings
```

and receive Doctor B's finances.

Likewise:

- Pharmacy staff cannot see another pharmacy's balance.
- Lab staff cannot see another lab's withdrawals.
- Patients cannot see another patient's receipts.

---

# 66. Financial Audit Log

Sensitive actions must be auditable.

Example events:

```text
PAYMENT_CREATED
PAYMENT_CONFIRMED
PAYMENT_FAILED
PAYMENT_MANUAL_REVIEW
REFUND_REQUESTED
REFUND_APPROVED
REFUND_REJECTED
REFUND_COMPLETED
WITHDRAWAL_REQUESTED
WITHDRAWAL_APPROVED
WITHDRAWAL_REJECTED
WITHDRAWAL_COMPLETED
BALANCE_HOLD_CREATED
BALANCE_HOLD_RELEASED
FINANCIAL_ADJUSTMENT_CREATED
```

Fields:

```text
ActorId
ActorRole
Action
EntityType
EntityId
Timestamp
CorrelationId
Reason
Metadata
```

---

# 67. Manual Financial Adjustment

Admin financial adjustments are dangerous and must not mutate historical payments directly.

Correct approach:

```text
Admin requests adjustment
      ↓
Permission check
      ↓
Reason required
      ↓
Approval if required
      ↓
Adjustment ledger entry
      ↓
Audit log
```

---

# 68. Separation of Duties

For sensitive finance operations, consider different permissions for:

```text
FINANCE_VIEW
FINANCE_REFUND
FINANCE_WITHDRAWAL_REVIEW
FINANCE_ADJUST
FINANCE_RECONCILE
FINANCE_EXPORT
```

A user who can view finances should not automatically be able to alter them.

---

# 69. Reconciliation

Reconciliation compares internal financial data to external gateway records.

Example:

```text
TechCare internal payments
        ↕
Gateway settlement/report
```

The system looks for:

- Missing internal payment.
- Missing gateway event.
- Amount mismatch.
- Currency mismatch.
- Duplicate payment.
- Unexpected success.
- Unexpected refund.
- Settlement difference.

---

# 70. Reconciliation Record

Suggested entity:

```text
ReconciliationRecord
```

Fields:

```text
Id
Gateway
Reference
InternalPaymentId
ExternalAmount
InternalAmount
Currency
Status
MismatchType
DetectedAt
ResolvedAt
ResolvedBy
ResolutionNote
```

---

# 71. Reconciliation States

```text
MATCHED
MISMATCH
MISSING_INTERNAL
MISSING_EXTERNAL
DUPLICATE
UNDER_REVIEW
RESOLVED
```

---

# 72. Financial Period Closing

For future scale, finance may need reporting periods.

Conceptually:

```text
Open Period
     ↓
Review
     ↓
Reconciled
     ↓
Closed
```

A closed reporting period should not be casually rewritten.

---

# 73. Disputes / Chargebacks

Where a payment method supports disputes:

```text
Payment SUCCEEDED
      ↓
Dispute received
      ↓
DISPUTED
      ↓
Provider earning may be held
      ↓
Admin review
      ├──→ Won
      └──→ Lost
```

The exact gateway dispute lifecycle must be mapped by the integration adapter.

---

# 74. Provider Balance Hold

Admin or system may temporarily hold provider funds when there is a legitimate financial reason.

Example:

```text
Available Balance = 5,000
Hold = 1,500
Spendable = 3,500
```

The hold should have:

```text
Reason
Amount
CreatedBy
CreatedAt
ReleaseCondition
Status
```

---

# 75. Notifications

Financial notifications should include:

## Patient

- Payment initiated.
- Payment successful.
- Payment failed.
- Refund requested.
- Refund completed.
- Receipt available.

## Doctor/Nurse

- Service payment successful.
- Earning created.
- Refund affecting earning.
- Withdrawal requested.
- Withdrawal approved/rejected/completed.

## Pharmacy/Lab

- Order/booking paid.
- Refund affects transaction.
- Payout update.

## Admin

- Payment mismatch.
- Large/unusual financial event.
- Reconciliation mismatch.
- Failed payout.
- Suspicious activity.

---

# 76. Financial Notification Rules

Notifications must not expose:

- Full payment credentials.
- Full payout destination.
- Sensitive gateway payload.

Example safe message:

```text
Your payment of 350 EGP was completed successfully.
Receipt: TC-2026-000123
```

---

# 77. Payment API Design

Example endpoint set:

```http
POST   /api/v1/payments/quotes
POST   /api/v1/payments
GET    /api/v1/payments/{id}
GET    /api/v1/payments
POST   /api/v1/payments/{id}/retry
POST   /api/v1/payments/{id}/cancel
GET    /api/v1/payments/{id}/receipt
```

Webhook:

```http
POST /api/v1/payments/webhooks/{gateway}
```

---

# 78. Refund API

```http
POST /api/v1/refunds
GET  /api/v1/refunds/{id}
GET  /api/v1/refunds
```

Admin operations where permitted:

```http
POST /api/v1/admin/refunds/{id}/approve
POST /api/v1/admin/refunds/{id}/reject
POST /api/v1/admin/refunds/{id}/retry
```

---

# 79. Provider Earnings API

```http
GET /api/v1/financials/earnings
GET /api/v1/financials/balance
GET /api/v1/financials/transactions
```

---

# 80. Withdrawal API

```http
POST /api/v1/withdrawals
GET  /api/v1/withdrawals/{id}
GET  /api/v1/withdrawals
POST /api/v1/withdrawals/{id}/cancel
```

Admin:

```http
GET  /api/v1/admin/withdrawals
POST /api/v1/admin/withdrawals/{id}/approve
POST /api/v1/admin/withdrawals/{id}/reject
POST /api/v1/admin/withdrawals/{id}/process
```

---

# 81. Reconciliation API

Admin-only example:

```http
POST /api/v1/admin/finance/reconciliation/run
GET  /api/v1/admin/finance/reconciliation
GET  /api/v1/admin/finance/reconciliation/{id}
POST /api/v1/admin/finance/reconciliation/{id}/resolve
```

---

# 82. Payment Quote API Validation

Input should identify the business object, not its price.

Example:

```json
{
  "businessType": "LAB_BOOKING",
  "businessObjectId": "LB-1001"
}
```

The server calculates the quote.

It should reject requests that attempt to override:

```text
subtotal
providerFee
servicePrice
deliveryFee
total
```

---

# 83. Payment Creation API Example

```json
{
  "quoteId": "QUOTE-123",
  "paymentMethod": "CARD",
  "idempotencyKey": "client-generated-unique-key"
}
```

The server verifies that the quote belongs to the authenticated user and is still valid.

---

# 84. Payment Response Example

```json
{
  "paymentId": "PAY-10001",
  "status": "REQUIRES_ACTION",
  "amount": 275.00,
  "currency": "EGP",
  "gatewaySession": {
    "clientToken": "..."
  }
}
```

Never return secret server credentials to the browser.

---

# 85. Payment Data Model

Recommended entities:

```text
PaymentQuote
Payment
PaymentAttempt
PaymentWebhookEvent
Refund
Receipt
LedgerEntry
ProviderEarning
BalanceHold
Withdrawal
PayoutMethod
ReconciliationRecord
FinancialAdjustment
FinancialAuditLog
```

---

# 86. PaymentQuote Fields

```text
Id
CustomerUserId
BusinessType
BusinessObjectId
Subtotal
DistanceFee
DeliveryFee
ServiceFee
Discount
Tax
Total
Currency
PricingVersion
ExpiresAt
Status
CreatedAt
```

---

# 87. PaymentAttempt Fields

```text
Id
PaymentId
AttemptNumber
Gateway
GatewayReference
Status
FailureCode
FailureMessageSafe
CreatedAt
CompletedAt
```

---

# 88. Payment Entity Fields

```text
Id
InternalReference
CustomerUserId
BusinessType
BusinessObjectId
ProviderUserId
ProviderOrganizationId
Amount
Currency
Status
Gateway
GatewayPaymentReference
IdempotencyKey
CreatedAt
UpdatedAt
PaidAt
```

---

# 89. Refund Entity Fields

```text
Id
PaymentId
BusinessType
BusinessObjectId
Amount
Currency
ReasonCode
CustomerNote
Status
RequestedByUserId
ReviewedByAdminId
GatewayRefundReference
CreatedAt
ProcessedAt
```

---

# 90. Receipt Entity Fields

```text
Id
PaymentId
ReceiptNumber
IssuedAt
CustomerUserId
BusinessDescription
Subtotal
Fees
Discount
Total
Currency
```

---

# 91. Withdrawal Entity Fields

```text
Id
OwnerType
OwnerId
PayoutMethodId
Amount
Currency
Status
RequestedAt
ReviewedAt
ProcessedAt
FailureReason
ExternalPayoutReference
```

---

# 92. Balance Strategy

Avoid storing a manually editable:

```text
Provider.Balance = 5000
```

as the only source of truth.

A safer strategy is to derive or update balances from ledger-backed transactions and controlled balance snapshots.

Example logical calculation:

```text
Available Balance
=
Eligible Provider Earnings
-
Completed/Reserved Withdrawals
-
Refund Adjustments
-
Active Holds
+
Approved Adjustments
```

---

# 93. Balance Snapshot

At scale, a cached balance can be stored for performance, but it must be reconcilable against the ledger.

The ledger remains the authoritative financial history.

---

# 94. Database Constraints

Recommended constraints/indexes include:

- Unique payment internal reference.
- Unique `(gateway, external event id)`.
- Unique idempotency key within the relevant security scope.
- Positive monetary amounts where appropriate.
- Currency required.
- Foreign keys to business objects where appropriate.
- Index on customer + created date.
- Index on provider + status.
- Index on organization + status.
- Index on withdrawal status.
- Index on reconciliation status.

---

# 95. Transaction Boundaries

A successful payment callback may need one database transaction around:

```text
Verify internal state
→ Update payment
→ Update business object
→ Create ledger entries
→ Create provider earning
→ Create receipt reference
→ Queue notification record
```

Do not perform fragile external gateway operations inside a long-running database transaction.

---

# 96. Outbox Pattern

For reliable financial event propagation, an outbox can be used.

Example:

```text
Database Transaction
 ├── Payment marked succeeded
 ├── Ledger entries created
 └── OutboxEvent created

Background worker
       ↓
Send notification / publish domain event
```

This prevents financial state from succeeding while downstream notifications silently disappear.

---

# 97. Event Examples

```text
PaymentSucceeded
PaymentFailed
RefundCompleted
ProviderEarningCreated
ProviderBalanceUpdated
WithdrawalRequested
WithdrawalCompleted
PaymentMismatchDetected
ReconciliationMismatchDetected
```

---

# 98. Background Jobs

Potential jobs:

```text
ProcessPaymentWebhooks
ExpirePaymentQuotes
RetryFailedWebhookProcessing
GenerateReceipts
UpdateFinancialSummaries
ReconcileGatewayPayments
RetryNotifications
DetectStuckPayments
DetectStuckWithdrawals
```

Jobs must be idempotent.

---

# 99. Stuck Payment Detection

Example:

```text
Payment = PROCESSING
Created long ago
No confirmed gateway event
```

System may mark it for:

```text
REVIEW
```

or trigger a status verification with the gateway.

Do not automatically mark failed unless business/gateway evidence supports that transition.

---

# 100. Stuck Withdrawal Detection

Same principle.

```text
Withdrawal = PROCESSING
No provider confirmation
```

The system should:

- Retry safe status lookup.
- Alert finance.
- Avoid creating duplicate payouts.

---

# 101. Financial Dashboard UX

## Patient Page

```text
------------------------------------------
My Payments
------------------------------------------
Total Paid
Refunded
Pending

[ Search ] [ Filter ]

Payment # | Service | Amount | Status
```

## Provider Page

```text
------------------------------------------
Financial Overview
------------------------------------------
Pending      Available      Withdrawn
  2,000         4,500          10,000

[Withdraw]

Recent Earnings
Recent Refunds
Withdrawal History
```

## Admin Page

```text
------------------------------------------
Finance Dashboard
------------------------------------------
GMV | Success | Refunds | Fees | Payables

Payments
Withdrawals
Refunds
Reconciliation
Disputes
Audit Logs
```

---

# 102. Financial Filters

Common filters:

```text
Date From
Date To
Status
Business Type
Provider
Organization
Payment Method
Currency
Amount Range
```

---

# 103. Export Rules

Financial exports should:

- Require authorization.
- Be auditable.
- Use secure generated files.
- Mask sensitive values where appropriate.
- Avoid exposing payment credentials.
- Include export actor and timestamp in audit logs.

Suggested endpoint:

```http
POST /api/v1/admin/finance/exports
```

---

# 104. Search vs Direct Authorization

Even if a user knows a payment ID, the API must verify access.

Example:

```http
GET /api/v1/payments/PAY-999
```

The server checks:

```text
Current user
      ↓
owns payment?
OR
is authorized provider?
OR
is authorized organization member?
OR
has finance admin permission?
```

---

# 105. Integration with Appointment Workflow

Doctor/Nurse flow:

```text
Request
 ↓
Accepted
 ↓
Quote
 ↓
Payment
 ↓
Confirmed
 ↓
Service
 ↓
Completed
 ↓
Earning
```

A failed payment must not accidentally unlock a paid-only appointment state.

---

# 106. Integration with Pharmacy Workflow

```text
Cart
 ↓
Price Snapshot
 ↓
Quote
 ↓
Payment
 ↓
Order Paid
 ↓
Stock Reservation / Fulfillment Policy
 ↓
Preparation
 ↓
Delivery/Pickup
 ↓
Completed
 ↓
Earning
```

The exact point at which stock is committed should match the pharmacy inventory policy.

---

# 107. Integration with Laboratory Workflow

```text
Lab Test Booking
 ↓
Quote
 ↓
Payment
 ↓
Booking Confirmed
 ↓
Collection / Lab Visit
 ↓
Processing
 ↓
Result
 ↓
Completion
 ↓
Earning
```

---

# 108. Integration with Blood Donor Workflow

The financial module should generally NOT create normal provider earnings for voluntary blood donation.

Instead it may support:

```text
Donation
 ↓
Eligibility/confirmation
 ↓
Reward Points Event
```

If non-cash platform benefits are introduced, they should be represented separately from monetary payments.

---

# 109. Discounts

Discounts should have explicit rules.

Possible entity:

```text
Discount / Promotion
```

Server validates:

- Active period.
- Eligibility.
- Usage count.
- Business scope.
- Minimum amount.
- Maximum discount.
- Single-use/limited-use conditions.

---

# 110. Promo Abuse Protection

Protect against:

- Reusing a one-time code.
- Multiple concurrent redemption.
- Client-side price manipulation.
- Applying promotion to unsupported business types.
- Creating many fake accounts to reuse a promotion.

Use transactional counters/constraints where needed.

---

# 111. Platform Fees

Fee model should be configurable.

Example conceptual configuration:

```text
FeeType = PERCENTAGE
Value = 10%
```

or:

```text
FeeType = FIXED
Value = 25 EGP
```

Do not hard-code fee values across controllers.

---

# 112. Fee Snapshot

When a payment is finalized, the actual fee applied must be stored as a snapshot.

This protects historical financial calculations if fee configuration changes later.

---

# 113. Tax Support

Tax support may be included as a configurable field even when not enabled in the initial prototype.

Example:

```text
TaxAmount
TaxRateSnapshot
TaxRuleVersion
```

Do not invent a tax rate in code.

---

# 114. Platform Revenue

Conceptual calculation:

```text
Platform Revenue
=
Collected Platform Fees
-
Refunded Fees
-
Approved Revenue Adjustments
```

Exact accounting treatment should be confirmed by the project's finance policy.

---

# 115. Financial Reports

Admin reports may include:

```text
Daily Revenue
Revenue by Domain
Revenue by Provider
Fees Collected
Refund Volume
Payment Failure Rate
Average Transaction Value
Withdrawals
Outstanding Provider Payables
```

---

# 116. Financial Analytics Rules

Dashboards must clearly distinguish:

```text
Gross Transaction Value
Platform Revenue
Provider Payables
Net Collected
Refunded Amount
Pending Amount
```

Do not label all successful payments as platform revenue.

---

# 117. Payment Failure Metrics

Track failure categories:

```text
CUSTOMER_CANCELLED
CARD_DECLINED
AUTH_REQUIRED
GATEWAY_ERROR
TIMEOUT
AMOUNT_MISMATCH
WEBHOOK_ERROR
UNKNOWN
```

Metrics should support operational troubleshooting.

---

# 118. Refund Metrics

Track:

```text
Refund Requested
Refund Approved
Refund Rejected
Refund Processing Time
Refund Failed
Refund by Domain
Refund by Reason
```

---

# 119. Withdrawal Metrics

Track:

```text
Requested
Approved
Rejected
Processing
Completed
Failed
Average Processing Time
```

---

# 120. Fraud / Abuse Signals

The platform may flag:

- Excessive payment retries.
- Repeated failed payments.
- Multiple payments against one business object.
- Repeated refund behavior.
- Abnormally frequent withdrawal attempts.
- Payout account changes followed by immediate withdrawal attempts.
- Repeated promotion abuse.
- Suspicious webhook mismatch patterns.

These should be signals for review, not automatic accusations.

---

# 121. Payment Rate Limiting

Recommended rate-limited operations:

```text
Payment creation
Payment retry
Refund request
Withdrawal request
Webhook endpoints according to gateway/network constraints
```

Rate limits should avoid blocking legitimate provider/gateway traffic.

---

# 122. Correlation IDs

Every financial request/event should carry a traceable correlation ID.

Example:

```text
CorrelationId = FIN-2026-ABC123
```

Use it in:

- Application logs.
- Payment records where appropriate.
- Webhook processing.
- Audit logs.
- Support investigations.

---

# 123. Logging Rules

Log enough to troubleshoot without leaking sensitive information.

Good:

```text
PaymentId
InternalReference
GatewayReference masked/limited
Status
Amount
Currency
CorrelationId
```

Avoid:

```text
Card number
CVV
Authentication secrets
Private gateway credentials
```

---

# 124. Exception Handling

Payment APIs must return safe, stable errors.

Example:

```json
{
  "code": "PAYMENT_NOT_AVAILABLE",
  "message": "The payment could not be started. Please try again."
}
```

Detailed gateway exception data remains server-side.

---

# 125. Frontend Payment State Handling

The frontend should understand:

```text
PENDING
REQUIRES_ACTION
PROCESSING
SUCCEEDED
FAILED
REVIEW
```

It should not assume that returning from the gateway means success.

Recommended UX:

```text
Return from gateway
      ↓
Call backend payment status API
      ↓
Backend returns authoritative state
      ↓
Render result
```

---

# 126. Payment Result Page

For uncertain status:

```text
We're confirming your payment.
Please wait while TechCare verifies the transaction.
```

Do not falsely show success before confirmation.

---

# 127. Receipt Download

Suggested endpoint:

```http
GET /api/v1/payments/{id}/receipt
```

Authorization is required.

Receipt file access must be object-level protected.

---

# 128. Payment History Pagination

Payment history can become large.

Use pagination:

```http
GET /api/v1/payments?page=1&pageSize=20
```

Prefer indexed sorting by:

```text
CreatedAt DESC
```

---

# 129. Financial API Authorization Matrix

| Action | Patient | Doctor | Nurse | Pharmacy | Laboratory | Admin |
|---|---:|---:|---:|---:|---:|---:|
| Create own payment | ✅ | ❌* | ❌* | ❌* | ❌* | Controlled |
| View own payments | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| View own earnings | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Request withdrawal | ❌ | ✅ | ✅ | ✅ | ✅ | ✅/finance |
| Request refund | ✅ | Limited | Limited | Limited | Limited | ✅ |
| Approve refund | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Approve withdrawal | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Reconcile | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Financial export | Restricted | Own scope only | Own scope only | Org scope | Org scope | ✅ |

`*` Providers may create payment records only when a business workflow explicitly requires provider-side payment actions; ordinary customer checkout is patient-side.

---

# 130. Service Completion and Earnings

Provider earning should not always be created at booking creation.

Typical rule:

```text
Payment Confirmed
      ↓
Service Completed
      ↓
Provider Earning Eligible
```

This protects the provider ledger from prematurely treating cancelled/unfulfilled services as completed earnings.

---

# 131. Refund and Earning Interaction

Example:

```text
Customer paid = 500
Provider earning = 450
Platform fee = 50

Full refund
```

The system must reverse the correct amounts according to configured refund/fee policy.

It must not simply subtract 500 from provider balance if provider was never actually credited 500.

---

# 132. Provider Earning Clawback

If funds were already made available/paid out and later a valid refund/dispute occurs, the system may need a negative adjustment.

Example:

```text
Original earning +450
Refund adjustment -450
```

The result must be explicitly linked and audited.

---

# 133. Negative Provider Balance

A provider may temporarily enter a negative balance only if the business/finance policy permits clawbacks after previous settlement.

This should be represented explicitly and not silently hidden.

---

# 134. Financial Holds

Possible hold reasons:

```text
REFUND_RISK
DISPUTE
VERIFICATION_REQUIRED
SUSPENDED_ACCOUNT
RECONCILIATION_MISMATCH
MANUAL_REVIEW
```

Every hold requires a reason and audit trail.

---

# 135. Account Suspension Interaction

When a provider is suspended:

- Existing completed financial records remain accessible according to authorization policy.
- New withdrawals may be blocked depending on reason.
- Existing pending withdrawals may be reviewed.
- Historical payments are not deleted.
- Financial data remains auditable.

---

# 136. Deleted Account Interaction

Soft deletion/anonymization rules should preserve required financial records.

Do not physically destroy transaction history merely because a profile is deactivated.

Customer-facing privacy deletion policies should be designed separately from financial retention requirements.

---

# 137. Medical Privacy and Finance

A financial transaction may reference a medical service.

The finance module should store only what it needs.

Example:

```text
Good:
Doctor Consultation — 300 EGP
```

Potentially unnecessary:

```text
Diagnosis = ...
Medication = ...
Clinical Note = ...
```

Clinical data belongs to the clinical modules.

---

# 138. Separation from Clinical Records

Payment data should link to:

```text
AppointmentId
BookingId
OrderId
LabBookingId
```

rather than copying entire clinical records into payment tables.

---

# 139. Refund Request UX

Patient sees:

```text
Payment Details
--------------------------
Amount: 500 EGP
Status: Paid

[Request Refund]
```

Then:

```text
Reason
Additional Note
Expected refund amount
Policy summary
[Submit]
```

The server recalculates eligibility.

---

# 140. Withdrawal UX

Provider enters:

```text
Amount: 1500 EGP
Payout Method: [Verified Method]

Available: 4000 EGP
```

Backend validates the final amount.

---

# 141. Admin Refund UX

Admin should see:

```text
Refund Request
-------------------------------
Payment: PAY-1001
Customer: User #123
Business: Nurse Booking #9001
Original: 500 EGP
Requested: 500 EGP
Reason: PROVIDER_CANCELLATION

Financial Impact
Provider: -450
Platform: -50

[Approve] [Reject]
```

Amounts are calculated by the backend.

---

# 142. Admin Withdrawal Review UX

```text
Withdrawal Request
-------------------------------
Owner: Laboratory #18
Amount: 4,500 EGP
Status: UNDER_REVIEW
Payout Method: Masked
Available Balance: 8,700 EGP

[Approve] [Reject]
```

---

# 143. Financial Support Workflow

Customer support may search a payment reference without having permission to execute a financial mutation.

Example support role:

```text
FINANCE_SUPPORT_VIEW
```

versus:

```text
FINANCE_REFUND_APPROVE
```

This keeps privileges separated.

---

# 144. End-to-End Doctor Payment Scenario

```text
1. Patient selects doctor service.
2. Backend calculates price.
3. Quote generated.
4. Patient accepts.
5. Appointment/payment record created.
6. Payment session created.
7. Patient completes gateway flow.
8. Gateway sends trusted event.
9. Backend validates signature.
10. Backend validates amount/currency/reference.
11. Payment marked SUCCEEDED.
12. Appointment becomes financially confirmed.
13. Patient receives notification.
14. Receipt is generated.
15. Doctor completes consultation.
16. Provider earning is created.
17. Platform fee is recorded.
18. Earning becomes available according to settlement rule.
19. Doctor requests withdrawal.
20. Withdrawal is reserved and reviewed.
21. Payout completed.
22. Ledger remains fully traceable.
```

---

# 145. End-to-End Nurse Payment Scenario

```text
Patient chooses nurse
↓
Service + distance price
↓
Quote
↓
Booking accepted
↓
Payment
↓
Visit completed
↓
Earning
↓
Withdrawal
```

Cancellation/refund branches must be supported at every applicable state.

---

# 146. End-to-End Pharmacy Payment Scenario

```text
Patient
 ↓
Medicine search
 ↓
Pharmacy selected
 ↓
Cart
 ↓
Stock validation
 ↓
Quote
 ↓
Payment
 ↓
Order paid
 ↓
Pharmacy processing
 ↓
Delivery/Pickup
 ↓
Completion
 ↓
Pharmacy earning
 ↓
Withdrawal
```

---

# 147. End-to-End Laboratory Payment Scenario

```text
Patient
 ↓
Test selected
 ↓
Collection mode
 ↓
Quote
 ↓
Payment
 ↓
Lab booking
 ↓
Sample collection / visit
 ↓
Processing
 ↓
Result
 ↓
Completion
 ↓
Lab earning
```

---

# 148. Failed Payment Scenario

```text
Patient starts payment
↓
Gateway failure
↓
Payment Attempt = FAILED
↓
Payment remains unpaid
↓
Patient notified
↓
Retry available
```

No provider earning is created.

---

# 149. Duplicate Webhook Scenario

```text
Gateway event E123 arrives
↓
Processed successfully

E123 arrives again
↓
Duplicate detected
↓
No second payment transition
↓
No second earning
↓
No second notification event
```

---

# 150. Duplicate Checkout Scenario

```text
Frontend sends same idempotency key twice
↓
First request creates payment
↓
Second request finds existing operation
↓
Same payment returned
```

---

# 151. Refund Scenario

```text
Payment = 700
Service cancelled
↓
Refund policy
↓
Refund = 700
↓
Refund sent to gateway
↓
Gateway confirms
↓
Refund = SUCCEEDED
↓
Ledger reversal
↓
Provider earning reversed if already created
↓
Patient notified
```

---

# 152. Partial Refund Scenario

```text
Payment = 700
↓
Refund 200
↓
Payment = PARTIALLY_REFUNDED
↓
Later refund 500
↓
Payment = REFUNDED
```

---

# 153. Withdrawal Scenario

```text
Available = 5,000
↓
Provider requests 2,000
↓
Reserve 2,000
↓
UNDER_REVIEW
↓
APPROVED
↓
PROCESSING
↓
COMPLETED
↓
Ledger withdrawal debit
↓
Available balance reduced
```

---

# 154. Withdrawal Failure Scenario

```text
Withdrawal = PROCESSING
↓
Payout fails
↓
Withdrawal = FAILED
↓
Reserved amount released
↓
Funds become available again
↓
Provider notified
```

---

# 155. Reconciliation Mismatch Scenario

```text
Gateway report says 800 EGP success
TechCare says 600 EGP
↓
Mismatch detected
↓
Payment held for review
↓
Reconciliation case created
↓
Finance admin investigates
↓
Adjustment/resolution recorded
```

---

# 156. Manual Adjustment Scenario

```text
Finance admin opens case
↓
Reason entered
↓
Permission checked
↓
Adjustment approved
↓
Ledger adjustment created
↓
Audit log created
↓
Balance updated/recomputed
```

---

# 157. Financial State Machine — Payment

```text
                 ┌───────────────┐
                 │    CREATED    │
                 └──────┬────────┘
                        ↓
                 ┌───────────────┐
                 │    PENDING    │
                 └──────┬────────┘
                        ↓
            ┌───────────┼───────────┐
            ↓           ↓           ↓
         FAILED    REQUIRES_ACTION  PROCESSING
                                    │
                              ┌─────┴─────┐
                              ↓           ↓
                         SUCCEEDED      FAILED
                              │
                              ↓
                     PARTIALLY_REFUNDED
                              │
                              ↓
                           REFUNDED
```

---

# 158. Financial State Machine — Provider Earning

```text
CREATED
  ↓
PENDING
  ├──→ HELD
  │      ↓
  │   RELEASED
  │
  └──→ AVAILABLE
          ↓
       WITHDRAWN
```

Alternative terminal states may include reversal/adjustment flows.

---

# 159. Financial State Machine — Withdrawal

```text
REQUESTED
   ↓
UNDER_REVIEW
 ┌─┴──────────┐
 ↓            ↓
REJECTED     APPROVED
               ↓
           PROCESSING
           ┌───┴────┐
           ↓        ↓
      COMPLETED   FAILED
```

---

# 160. Financial State Machine — Refund

```text
REQUESTED
   ↓
UNDER_REVIEW
 ┌─┴────────┐
 ↓          ↓
REJECTED   APPROVED
             ↓
          PROCESSING
          ┌────┴────┐
          ↓         ↓
      SUCCEEDED    FAILED
```

---

# 161. Suggested ASP.NET Core Layering

```text
TechCare.Api
TechCare.Application
TechCare.Domain
TechCare.Infrastructure
TechCare.Persistence
```

Payment-specific examples:

```text
Application
 ├── CreatePayment
 ├── GetPaymentStatus
 ├── RequestRefund
 ├── ApproveRefund
 ├── RequestWithdrawal
 ├── ReviewWithdrawal
 └── ReconcilePayments

Domain
 ├── Payment
 ├── Refund
 ├── Withdrawal
 ├── LedgerEntry
 └── ProviderEarning

Infrastructure
 ├── GatewayAdapter
 ├── WebhookVerifier
 └── PayoutProviderAdapter
```

---

# 162. Payment Gateway Abstraction

Recommended contract concept:

```csharp
public interface IPaymentGateway
{
    Task<GatewayPaymentSession> CreatePaymentAsync(...);
    Task<GatewayPaymentStatus> GetPaymentStatusAsync(...);
    Task<GatewayRefundResult> RefundAsync(...);
    Task<bool> VerifyWebhookAsync(...);
}
```

Do not make controllers depend on one specific gateway SDK.

---

# 163. Financial Domain Services

Possible services:

```text
IPricingService
IPaymentService
IRefundService
ILedgerService
IEarningsService
IWithdrawalService
IReconciliationService
IReceiptService
IPaymentGateway
```

---

# 164. Business Rules Should Not Live in Controllers

Avoid:

```text
Controller:
if cancellation...
if refund...
if fee...
if earnings...
```

Prefer application/domain services where rules are centralized and testable.

---

# 165. API Response Consistency

All financial APIs should use a consistent envelope/error model where the rest of TechCare uses one.

Example error codes:

```text
QUOTE_EXPIRED
PAYMENT_ALREADY_SETTLED
PAYMENT_NOT_FOUND
PAYMENT_FAILED
REFUND_NOT_ELIGIBLE
REFUND_AMOUNT_INVALID
WITHDRAWAL_INSUFFICIENT_BALANCE
WITHDRAWAL_ALREADY_PENDING
PAYOUT_METHOD_NOT_VERIFIED
FINANCIAL_ACTION_NOT_AUTHORIZED
```

---

# 166. Concurrency Rules

High-risk concurrent operations:

- Checkout/payment creation.
- Stock/payment interaction.
- Double refunds.
- Double withdrawal.
- Balance reservation.
- Duplicate webhook events.
- Promotion redemption.

Every such operation needs concurrency-safe design.

---

# 167. Optimistic Concurrency

Where appropriate, entities can use:

```text
RowVersion
```

or another concurrency token.

For example:

```text
Withdrawal balance reservation
```

A stale request should fail instead of overwriting a newer state.

---

# 168. Financial Invariants

The system should enforce invariants such as:

```text
RefundedAmount <= CapturedAmount
```

```text
WithdrawalAmount <= SpendableBalance
```

```text
CompletedWithdrawalAmount cannot be negative
```

```text
Provider earning cannot exceed the configured eligible customer settlement
```

```text
A single payment cannot be settled twice
```

---

# 169. Testing Strategy

Payment requires unit + integration + concurrency + end-to-end testing.

## Unit Tests

Test:

- Price calculation.
- Distance fee.
- Discount rules.
- Fee calculation.
- Refund policy.
- Provider earning.
- Withdrawal eligibility.
- State transitions.

## Integration Tests

Test:

- Payment creation.
- Gateway adapter.
- Webhook processing.
- Refund flow.
- Ledger writes.
- Withdrawal workflow.

## End-to-End Tests

Test:

- Full patient purchase.
- Full provider earning.
- Full refund.
- Full withdrawal.

---

# 170. Required Payment Test Cases

### TC-PAY-001
Successful payment.

Expected:

```text
Payment = SUCCEEDED
Business object financially confirmed
Ledger created
Receipt available
```

### TC-PAY-002
Failed payment.

Expected:

```text
Payment = FAILED
No provider earning
```

### TC-PAY-003
Duplicate payment request.

Expected:

```text
One logical payment
```

### TC-PAY-004
Duplicate webhook.

Expected:

```text
One financial transition
```

### TC-PAY-005
Amount mismatch.

Expected:

```text
No automatic success
Review case
```

---

# 171. Required Refund Test Cases

### TC-REF-001
Full refund.

### TC-REF-002
Partial refund.

### TC-REF-003
Refund exceeds remaining refundable amount.

Expected:

```text
Rejected
```

### TC-REF-004
Duplicate refund event.

Expected:

```text
No duplicate balance mutation
```

### TC-REF-005
Refund gateway failure.

Expected:

```text
Refund = FAILED
Retry available safely
```

---

# 172. Required Withdrawal Test Cases

### TC-WD-001
Valid withdrawal.

### TC-WD-002
Insufficient balance.

### TC-WD-003
Two simultaneous withdrawals.

Expected:

```text
Combined amount cannot exceed spendable balance
```

### TC-WD-004
Failed payout.

Expected:

```text
Reservation released
```

### TC-WD-005
Suspended provider attempts withdrawal.

Expected:

```text
Rejected or held according to policy
```

---

# 173. Security Test Cases

### SEC-PAY-001
Change amount from frontend.

Expected:

```text
Server ignores client amount
```

### SEC-PAY-002
Use another user's payment ID.

Expected:

```text
403 or secure not-found behavior
```

### SEC-PAY-003
Forge webhook signature.

Expected:

```text
Webhook rejected
```

### SEC-PAY-004
Replay valid webhook event.

Expected:

```text
No duplicate processing
```

### SEC-PAY-005
Call admin refund endpoint as patient.

Expected:

```text
403
```

---

# 174. Financial Integration Test Matrix

| Scenario | Patient | Provider | Finance | Expected Result |
|---|---:|---:|---:|---|
| Successful doctor payment | ✅ | ✅ | ✅ | Paid + earning path |
| Failed payment | ✅ | ✅ | ✅ | No earning |
| Full refund | ✅ | ✅ | ✅ | Financial reversal |
| Partial refund | ✅ | ✅ | ✅ | Partial reversal |
| Withdrawal | ❌ | ✅ | ✅ | Payout workflow |
| Reconciliation mismatch | ❌ | ❌ | ✅ | Review case |
| Duplicate webhook | ✅ | ✅ | ✅ | Idempotent |
| Unauthorized payment read | ✅ | ✅ | ❌ | Denied |

---

# 175. Database Relationship Overview

```text
User
 ├──< Payment
 ├──< Refund (through Payment)
 ├──< Withdrawal
 ├──< FinancialAuditLog
 └──< PayoutMethod

Payment
 ├──< PaymentAttempt
 ├──< PaymentWebhookEvent
 ├──< Refund
 ├──< Receipt
 ├──< LedgerEntry
 └──< ProviderEarning

Provider / Organization
 ├──< ProviderEarning
 ├──< Withdrawal
 ├──< BalanceHold
 └──< PayoutMethod

Payment
 ───── references ─────
Appointment / Booking / Order / LabBooking
```

---

# 176. Full Financial Sequence Diagram

```text
Patient
  │
  │ Request Quote
  ▼
TechCare API
  │
  ├── Validate Business Object
  ├── Calculate Price
  └── Create Quote
  │
  ▼
Patient confirms
  │
  ▼
Payment Service
  │
  ├── Validate Quote
  ├── Create Payment
  └── Create Gateway Session
  │
  ▼
Gateway
  │
  ├── Process payment
  └── Send callback/webhook
  │
  ▼
Webhook Endpoint
  │
  ├── Verify signature
  ├── Detect duplicates
  ├── Validate amount/currency
  ├── Update Payment
  ├── Update Business Object
  ├── Write Ledger
  ├── Create Earning
  └── Emit Notification
  │
  ▼
Patient / Provider / Admin
```

---

# 177. Full Refund Sequence Diagram

```text
Patient / Admin
      │
      ▼
Refund Request
      │
      ▼
Refund Service
      │
      ├── Validate Payment
      ├── Validate Refund Policy
      ├── Calculate Refund
      └── Create Refund
      │
      ▼
Payment Gateway
      │
      ▼
Refund Confirmation
      │
      ▼
TechCare
      │
      ├── Refund SUCCEEDED
      ├── Ledger Reversal
      ├── Earning Adjustment
      └── Notification
```

---

# 178. Full Withdrawal Sequence Diagram

```text
Provider
   │
   ▼
Withdrawal Request
   │
   ▼
Financial Service
   │
   ├── Verify Provider
   ├── Verify Payout Method
   ├── Lock/Reserve Balance
   └── Create Withdrawal
   │
   ▼
Finance Admin / Payout Provider
   │
   ├── Approve
   └── Process
   │
   ▼
Completion
   │
   ├── Withdrawal = COMPLETED
   ├── Ledger Debit
   └── Notification
```

---

# 179. Operational Monitoring

Monitor:

```text
Payment success rate
Payment failure rate
Webhook latency
Webhook failure count
Refund success rate
Withdrawal failure rate
Reconciliation mismatches
Stuck processing records
Duplicate webhook count
Duplicate payment attempts
```

Alerts should focus on anomalies that can affect money or customer experience.

---

# 180. Observability

Recommended structured logs:

```text
CorrelationId
PaymentId
BusinessObjectId
EventType
StatusBefore
StatusAfter
Gateway
Amount
Currency
DurationMs
```

Do not log secrets.

---

# 181. API Documentation Requirements

Swagger/OpenAPI should describe:

- Authentication.
- Request bodies.
- Response models.
- Financial statuses.
- Error codes.
- Authorization requirements.
- Idempotency headers.
- Webhook endpoint expectations.

---

# 182. Frontend Routes — Patient

Example:

```text
/payments
/payments/:id
/payments/:id/receipt
/payments/:id/refund
/checkout/:id
```

---

# 183. Frontend Routes — Provider

Example:

```text
/financials
/financials/transactions
/financials/earnings
/financials/withdrawals
/financials/payout-methods
```

---

# 184. Frontend Routes — Admin

Example:

```text
/admin/finance
/admin/finance/payments
/admin/finance/refunds
/admin/finance/withdrawals
/admin/finance/reconciliation
/admin/finance/adjustments
/admin/finance/reports
/admin/finance/audit
```

---

# 185. Error UX

### Payment failed

Show:

```text
Payment unsuccessful.
Your order/booking has not been charged successfully.
[Try Again]
```

### Payment pending

Show:

```text
Your payment is being confirmed.
```

### Refund pending

Show:

```text
Your refund request is being processed.
```

### Withdrawal failed

Show:

```text
The withdrawal could not be completed.
Your funds remain in your TechCare balance.
```

---

# 186. Business Rule: No Financial Mutation from GET

GET endpoints should not:

- Charge payments.
- Approve refunds.
- Create withdrawals.
- Post ledger adjustments.

Use explicit mutation endpoints.

---

# 187. Business Rule: Never Trust Client Status

Client says:

```json
{
  "paymentStatus": "SUCCEEDED"
}
```

This has no authority.

The server uses its own state.

---

# 188. Business Rule: Never Trust Client Ownership

A request containing:

```json
{
  "providerId": 99
}
```

does not grant access to provider 99.

Authorization must be established from authenticated identity and server-side relationships.

---

# 189. Business Rule: Payment Resource Ownership

Every payment must have a clear relationship to:

```text
Customer
Business Object
Provider/Organization where relevant
```

This is required for authorization and reconciliation.

---

# 190. Business Rule: Financial Records Are Append-Friendly

Prefer:

```text
Add transaction/reversal/adjustment
```

over:

```text
Edit historical transaction directly
```

---

# 191. Business Rule: Historical Reproducibility

For a payment made today, the system should be able to reproduce:

```text
How total was calculated.
What fee was applied.
What discount was applied.
What provider earned.
What was refunded.
What was withdrawn.
```

Price and fee snapshots are therefore essential.

---

# 192. Business Rule: One Successful Settlement Per Obligation

For an order/booking that requires one settlement:

```text
Successful Payment Count <= 1
```

unless the business explicitly supports installments/split payments.

---

# 193. Future Support: Split Payments

Not required for the basic prototype but architecture may later support:

```text
One Business Object
 ↓
Multiple Payment Components
```

Example:

```text
Wallet = 100
Card   = 200
Total  = 300
```

Do not implement until the product requires it.

---

# 194. Future Support: Subscriptions

Potential future feature:

```text
Subscription
Recurring Payment
Renewal
Retry
Cancellation
```

Not required for the current core service model.

---

# 195. Future Support: Coupons and Wallets

Architecture may later support:

- Promotional wallet credits.
- Loyalty discounts.
- Store credits.
- Gift balance.

These should be separate financial concepts, not fake payment successes.

---

# 196. Future Support: Cash on Delivery

If introduced for pharmacy, define separately:

```text
PAYMENT_METHOD = CASH
PAYMENT_STATUS = PENDING_SETTLEMENT
```

Then on successful delivery/collection:

```text
Cash Collected
↓
Settled
```

Do not pretend an online gateway captured money when it did not.

---

# 197. Future Support: Multiple Currencies

Would require:

- Currency on every monetary record.
- Exchange rate snapshot.
- Settlement currency policy.
- Rounding rules.
- Gateway support.

Do not assume multi-currency support merely by adding a currency dropdown.

---

# 198. MVP Recommendation

For the TechCare prototype, implement:

### Must Have

```text
Server-side pricing
Payment abstraction
One online payment gateway integration
Payment attempts
Payment status
Idempotency
Webhook verification
Payment history
Receipt
Refund workflow
Provider earnings
Withdrawal request
Admin withdrawal review
Ledger records
Financial audit log
Basic reconciliation support
```

### Nice to Have

```text
Multiple gateways
Advanced dispute workflow
Automated reconciliation dashboard
Advanced fraud scoring
Complex promotions
Multiple currencies
Subscriptions
Split payments
```

---

# 199. Definition of Done — Payment

The Payment module is considered complete when:

- Patient can receive an authoritative quote.
- Patient can initiate payment.
- Backend verifies payment success.
- Duplicate requests are safe.
- Duplicate webhooks are safe.
- Failed payments are handled.
- Payment status is queryable.
- Receipt is generated.
- Refunds are controlled.
- Partial refunds are supported if required.
- Provider earnings are calculated from finalized transactions.
- Provider can view balance.
- Provider can request withdrawal.
- Withdrawal cannot exceed spendable balance.
- Admin can review withdrawals.
- Ledger entries are traceable.
- Financial actions are audited.
- Object-level authorization is enforced.
- Sensitive payment data is not stored improperly.
- Automated tests cover critical financial invariants.
- Reconciliation cases can be detected and reviewed.

---

# 200. Final Full-System Payment Map

```text
                         TECHCARE FINANCE
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
     PRICING                PAYMENTS               LEDGER
        │                      │                      │
   ┌────┴────┐          ┌──────┴──────┐         ┌────┴─────┐
   │         │          │             │         │          │
Distance  Services   Attempts      Webhooks  Earnings  Adjustments
   │         │          │             │         │          │
   └────┬────┘          └──────┬──────┘         └────┬─────┘
        │                       │                     │
        └───────────────────────┼─────────────────────┘
                                │
                           BUSINESS FLOWS
                                │
       ┌───────────┬────────────┼────────────┬────────────┐
       │           │            │            │            │
     Doctor      Nurse      Pharmacy       Lab       Other Approved
       │           │            │            │            │
       └───────────┴────────────┴────────────┴────────────┘
                                │
                          REFUNDS / HOLDS
                                │
                                ▼
                           WITHDRAWALS
                                │
                                ▼
                       ADMIN / RECONCILIATION
```

---

# 201. Recommended Implementation Order

Build in this logical order so the team avoids circular dependencies:

```text
1. Money / Currency primitives
2. Pricing engine
3. Payment entities
4. Payment attempts
5. Gateway abstraction
6. Payment creation
7. Payment status endpoint
8. Webhook processing
9. Idempotency
10. Receipt
11. Ledger
12. Provider earnings
13. Refunds
14. Withdrawals
15. Admin finance controls
16. Reconciliation
17. Notifications
18. Reports
19. Advanced monitoring
```

---

# 202. Ownership Recommendation for the Team

Because TechCare already has role-specific owners, Payment & Financial should be treated as a **shared cross-system module**.

Recommended split:

```text
Authentication Owner
→ Financial authorization foundation

Patient Owner
→ Checkout/payment UX from patient side

Doctor Owner
→ Doctor pricing + earnings integration

Nurse Owner
→ Nurse pricing + earnings integration

Pharmacy Owner
→ Order pricing + pharmacy settlement integration

Laboratory Owner
→ Test pricing + laboratory settlement integration

Blood Donor Owner
→ Ensure donation rewards remain separate from commercial payment flows

Admin Owner
→ Refunds / withdrawals / reconciliation / finance dashboard
```

One person should still be the technical owner of the shared Payment core to prevent seven different implementations of money logic.

---

# 203. Final Engineering Rules

```text
RULE 1  → Backend owns the amount.
RULE 2  → Backend owns payment status.
RULE 3  → Webhooks are verified and idempotent.
RULE 4  → Every retry is safe.
RULE 5  → Refunds cannot exceed captured funds.
RULE 6  → Withdrawals cannot exceed spendable balance.
RULE 7  → Historical financial facts are not silently edited.
RULE 8  → Provider earnings are not the same thing as customer payment.
RULE 9  → Platform revenue is not the same thing as GMV.
RULE 10 → Pharmacy/Lab organization funds belong to the organization.
RULE 11 → Blood donation is not treated as a blood marketplace.
RULE 12 → Sensitive payment credentials never belong in TechCare databases.
RULE 13 → Financial endpoints require object-level authorization.
RULE 14 → Every financial mutation is auditable.
RULE 15 → Every critical financial operation is concurrency-safe.
RULE 16 → Every payment amount is stored with its currency.
RULE 17 → Every finalized payment remains traceable to a business object.
RULE 18 → Notifications are downstream effects, not the source of truth.
RULE 19 → Reconciliation must be possible even when external callbacks fail.
RULE 20 → Financial state must remain internally consistent after retries, failures, and concurrent requests.
```

---

# 204. Final End-to-End Architecture

```text
                         ┌───────────────────────┐
                         │      TechCare UI      │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │      ASP.NET API      │
                         └───────────┬───────────┘
                                     │
                         ┌───────────┴───────────┐
                         │ Payment Application   │
                         │       Services        │
                         └───────────┬───────────┘
                                     │
          ┌──────────────────────────┼─────────────────────────┐
          │                          │                         │
          ▼                          ▼                         ▼
   ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
   │   Pricing   │           │   Payment   │           │   Refund    │
   │   Engine    │           │   Service   │           │   Service   │
   └──────┬──────┘           └──────┬──────┘           └──────┬──────┘
          │                          │                         │
          └──────────────────────────┼─────────────────────────┘
                                     │
                                     ▼
                           ┌───────────────────┐
                           │  Financial Core   │
                           │ Ledger / Earnings │
                           └─────────┬─────────┘
                                     │
                     ┌───────────────┼────────────────┐
                     │               │                │
                     ▼               ▼                ▼
                Withdrawals    Reconciliation     Audit
                     │               │                │
                     └───────────────┼────────────────┘
                                     │
                                     ▼
                           External Payment Gateway
```

---

# 205. Conclusion

The Payment & Financial module should be treated as one of the most sensitive shared components in TechCare.

The primary objective is not merely to add a payment button. The objective is to create a traceable financial lifecycle:

```text
PRICE
  ↓
QUOTE
  ↓
PAYMENT ATTEMPT
  ↓
PAYMENT VERIFICATION
  ↓
SETTLEMENT
  ↓
LEDGER
  ↓
PROVIDER EARNING
  ↓
REFUND / HOLD / ADJUSTMENT when applicable
  ↓
WITHDRAWAL
  ↓
RECONCILIATION
  ↓
AUDIT
```

Every transition must be:

```text
Authenticated
Authorized
Validated
Idempotent
Concurrency-safe
Auditable
Traceable
```

This design gives TechCare a financial foundation that can support Doctor, Nurse, Pharmacy, Laboratory, and other approved service flows without duplicating payment logic in every module.
