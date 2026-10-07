# TechCare — Location & Geo Services Complete Workflow

> **Document Type:** Cross-System Workflow  
> **Platform:** TechCare Healthcare Platform  
> **Primary Stack:** ASP.NET Core Web API + SQL Server + React  
> **Architecture:** Modular Monolith / Clean Architecture friendly  
> **Status:** Implementation-ready workflow  
> **Scope:** Addresses, GPS, map pins, geocoding, reverse geocoding, distance, nearby discovery support, provider service areas, location privacy, booking snapshots, geo-driven events, auditing, security, testing, and future routing/live-location support.

---

## 1. Module Purpose

The **Location & Geo Services** module is a shared platform capability used by almost every TechCare business module that depends on physical location.

It should answer:

- Where is the patient/provider/branch?
- Which saved address is being used?
- How far are two locations apart?
- Is a destination inside a provider's service area?
- Which doctors/nurses/pharmacies/labs are nearby?
- What location data can each actor see?
- What happens when GPS is unavailable or inaccurate?
- How can location safely participate in booking, pricing, delivery, collection, and future emergency workflows?

### Core principle

> **Location owns geographic facts and rules. Search owns discovery/ranking. Finance owns money. Booking owns transactions. Authorization owns access. Notification owns communication.**

---

# 2. Why TechCare Needs a Dedicated Geo Module

Without a shared location layer, every role can end up implementing location differently.

### Bad architecture

```text
Patient -> latitude/longitude
Doctor -> own location logic
Nurse -> own distance formula
Pharmacy -> different coordinates table
Lab -> another location system
Blood Donor -> text-only location
```

This causes:

- duplicated code.
- inconsistent distances.
- privacy bugs.
- incorrect pricing.
- difficult maintenance.
- different coordinate conventions.
- hard-to-test business rules.

### Correct architecture

```text
                        TechCare API
                             |
                    Location & Geo Module
                             |
       ┌──────────────┬──────┼──────┬───────────────┐
       |              |      |      |               |
    Patient        Doctor   Nurse Pharmacy       Lab/Donor
       |              |      |      |               |
       └──────────────┴──────┼──────┴───────────────┘
                             |
                     SQL Server Spatial
                             |
                  External Geo Providers
```

---

# 3. Actors

## 3.1 Patient

Uses location for:

- saved addresses.
- current location.
- nearby doctors.
- nearby nurses.
- nearby pharmacies.
- nearby laboratories.
- blood donation discovery.
- home visit booking.
- medicine delivery.
- lab home sample collection.

## 3.2 Doctor

Uses location for:

- practice/clinic location.
- home-visit base.
- service radius/area.
- authorized booking destination.
- travel distance.

## 3.3 Nurse

Uses location for:

- nursing service area.
- home visit destination.
- travel-distance pricing.

## 3.4 Pharmacist / Pharmacy

Uses:

- branch location.
- nearby discovery.
- delivery area.
- pickup location.

## 3.5 Laboratory

Uses:

- branch location.
- nearby test search.
- home sample collection area.

## 3.6 Blood Donor

Uses:

- approximate area.
- donation center discovery.
- future proximity matching.

Exact donor coordinates should not be publicly exposed.

## 3.7 Admin

Uses:

- provider location verification.
- service-area configuration.
- geo-health monitoring.
- fraud/anomaly review.
- audited support access.

Admin does not automatically get unrestricted exact patient location access.

## 3.8 System

Responsible for:

- coordinate validation.
- geocoding.
- reverse geocoding.
- distance calculation.
- spatial filtering.
- service-area validation.
- caching.
- rate limiting.
- privacy filtering.
- auditing.

---

# 4. MVP Scope

The MVP should implement:

1. Patient saved addresses.
2. Current GPS capture.
3. Manual map pin.
4. Reverse geocoding.
5. Address search/geocoding.
6. Provider coordinates.
7. Radius-based service areas.
8. Nearby doctors.
9. Nearby nurses.
10. Nearby pharmacies.
11. Nearby laboratories.
12. Straight-line distance.
13. Booking location snapshots.
14. Server-side distance validation.
15. Location authorization/privacy.
16. Basic location audit.

---

# 5. Future Scope

Later versions may add:

- road routing.
- ETA.
- live tracking.
- geofencing.
- delivery tracking.
- emergency live location.
- route optimization.
- polygon service areas.
- map clustering.
- heatmaps.
- scheduling feasibility based on travel time.
- advanced GIS analytics.

Do not over-engineer the student MVP with all of these.

---

# 6. Location Types

TechCare should explicitly distinguish:

### Permanent/Saved Address

A reusable address owned by a user.

### Booking Location

The exact destination selected for one transaction.

### Provider Base Location

Where a provider or branch is based.

### Service Area

Where a provider agrees to serve.

### Current Device Location

Temporary GPS location captured from the device.

### Approximate Location

Privacy-reduced location representation.

### Delivery Location

Destination for pharmacy delivery.

### Collection Location

Destination for laboratory home sample collection.

### Emergency Location

Future high-sensitivity location associated with an emergency incident.

---

# 7. Coordinate Model

Every captured coordinate should carry metadata.

```text
Latitude
Longitude
AccuracyMeters
CapturedAtUtc
Source
```

Possible `Source` values:

```text
GPS
Network
ManualPin
AddressGeocoding
ProviderRegistration
AdminVerified
```

Example:

```json
{
  "latitude": 31.123456,
  "longitude": 31.234567,
  "accuracyMeters": 18.5,
  "source": "GPS",
  "capturedAtUtc": "2026-10-07T15:00:00Z"
}
```

---

# 8. Coordinate Validation

Always validate:

```text
Latitude  = -90 .. +90
Longitude = -180 .. +180
```

Also reject:

```text
NaN
Infinity
negative accuracy
impossible numeric values
```

If coordinates are for an operating region such as Egypt, an optional configurable sanity check may detect obviously impossible locations. Do not permanently hard-code the whole system to one country.

---

# 9. Coordinate Order — Critical Rule

One of the easiest geo bugs is reversing the coordinate order.

User-facing form:

```text
latitude, longitude
```

Spatial libraries often construct:

```text
X = longitude
Y = latitude
```

For example:

```csharp
new Point(longitude, latitude)
```

Use a shared helper so developers cannot accidentally swap them.

---

# 10. Accuracy Model

Not all location data is equally trustworthy.

Example operational categories:

```text
< 20m       Excellent
20–100m     Acceptable
100–500m    Weak
> 500m      Very weak
```

These are engineering categories, not clinical guarantees.

A weak GPS point may still be sufficient for nearby search but may be insufficient for exact arrival verification.

---

# 11. Address Structure

Recommended fields:

```text
Id
OwnerId
Label
Country
Governorate
City
District
Neighborhood
Street
BuildingNumber
Floor
Apartment
Landmark
PostalCode
FormattedAddress
Latitude
Longitude
GeoPoint
AccuracyMeters
Source
IsDefault
CreatedAtUtc
UpdatedAtUtc
DeletedAtUtc
RowVersion
```

Allow free-text address instructions in addition to structured fields.

---

# 12. Address Labels

Suggested labels:

```text
Home
Work
Family
Temporary
Other
```

The user may choose a custom label if the product allows it.

---

# 13. Patient Address Workflow

```text
Patient Dashboard
      |
      v
My Addresses
      |
      v
Add Address
      |
      v
Choose Method
  ├── Current GPS
  ├── Search Address
  ├── Drop Pin
  └── Manual Entry
      |
      v
Validate Coordinates
      |
      v
Geocode / Reverse Geocode
      |
      v
User Confirms
      |
      v
Choose Label
      |
      v
Save
      |
      v
Optional: Set Default
```

---

# 14. Add Address — Detailed Steps

## Step 1 — Open Add Address

UI:

```text
Address Label
Governorate
City
District
Street
Building
Floor
Apartment
Landmark
Location on Map
[Use My Current Location]
[Save]
```

## Step 2 — Capture location

Browser/device may return:

```text
Granted
Denied
Unavailable
Timeout
Low accuracy
```

## Step 3 — Validate

Backend validates all coordinates and data.

## Step 4 — Resolve address

Reverse geocoder may provide the readable address.

## Step 5 — User confirms

The generated address remains editable.

## Step 6 — Persist

Save under the authenticated owner.

---

# 15. GPS Permission Handling

GPS permission is not guaranteed.

### Granted

Use GPS.

### Denied

Do not break the workflow. Offer:

```text
Search address
Drop pin
Manual address
```

### Timeout

Show:

```text
We couldn't detect your location.
Please try again or select the address manually.
```

### Unavailable

Use fallback.

---

# 16. Important UX Rule

Always show the location currently being used.

Example:

```text
📍 Searching near

Home — Mansoura

[Change location]
```

This prevents patients from wondering why search results changed.

---

# 17. Default Address

Normally only one active address may be default.

Example:

```text
Home  -> true
Work  -> false
```

When user selects Work as default:

```text
Begin transaction

Home = false
Work = true

Commit
```

Handle race conditions so two concurrent requests cannot leave two defaults.

---

# 18. First Address Rule

If the user has no active addresses:

```text
First valid address -> IsDefault = true
```

unless the product explicitly allows no default.

---

# 19. Delete Address

Workflow:

```text
Delete
  |
  v
Verify ownership
  |
  v
Check if default
  |
  v
Check active business dependencies
  |
  v
Soft delete/archive
  |
  v
Audit event
```

Historical transactions should remain intact through snapshots.

---

# 20. Editing an Address

```text
Edit
  |
  v
Change fields / move pin
  |
  v
Validate
  |
  v
Save
  |
  v
Audit
```

Do not rewrite an old booking's historical visit location.

---

# 21. Historical Location Snapshot

Whenever location becomes part of a transaction, store a snapshot.

Example:

```json
{
  "addressText": "25 Example Street, Mansoura",
  "latitude": 31.000001,
  "longitude": 31.400002,
  "accuracyMeters": 20,
  "capturedAtUtc": "2026-10-07T15:20:00Z"
}
```

Why?

```text
Today:
Patient Home = Address A

Tomorrow:
Patient Home = Address B

Last week's booking:
must still point to Address A
```

---

# 22. Booking Location Workflow

For a doctor/nurse home visit:

```text
Patient selects provider
       |
       v
Select visit address
       |
       v
Get coordinates
       |
       v
Provider service-area check
       |
       +---- Outside -> stop / alternatives
       |
       v
Calculate distance
       |
       v
Pricing calculation
       |
       v
Show final quote
       |
       v
Patient confirms
       |
       v
Create booking + location snapshot
```

---

# 23. Provider Location

A provider's location should not automatically be treated as their private residential address.

Distinguish:

```text
Clinic location
Home-visit base
Public branch
Private residence
Service area
```

Only the intended public/operational location should be exposed.

---

# 24. Doctor Service Radius

Example:

```text
Base fee: 500 EGP
Included distance: 5 km
Extra rate: 5 EGP/km
Maximum radius: 25 km
```

These values belong to business/pricing configuration, not hard-coded into Geo.

Geo only answers:

```text
How far?
Inside or outside?
```

---

# 25. Nurse Service Radius

Same architecture.

Example:

```text
Base fee
Included kilometers
Extra kilometer rate
Maximum service radius
```

Reuse the shared geo service rather than duplicating calculations.

---

# 26. Distance Calculation

Two major distance concepts:

### Straight-Line Distance

Useful for:

- nearby sorting.
- radius filtering.
- first-pass candidate selection.

### Road Distance

Useful for:

- delivery ETA.
- real driving distance.
- future route planning.

Never display straight-line distance as driving distance.

---

# 27. Haversine

For basic geographic proximity:

```text
a = sin²(Δφ/2)
    + cos(φ1) × cos(φ2) × sin²(Δλ/2)

c = 2 × atan2(√a, √(1-a))

distance = R × c
```

where Earth radius is approximately:

```text
R = 6371 km
```

In production, use a tested geo library where possible.

---

# 28. .NET Spatial Architecture

Recommended stack:

```text
ASP.NET Core
   |
   +-- EF Core
   |
   +-- NetTopologySuite
   |
   +-- SQL Server geography
```

Use a common coordinate reference system such as WGS 84 (`SRID 4326`).

---

# 29. GeoCoordinate Value Object

Example:

```csharp
public sealed record GeoCoordinate(
    double Latitude,
    double Longitude,
    double? AccuracyMeters
);
```

Creation should be validated.

---

# 30. IGeoService

Suggested abstraction:

```csharp
public interface IGeoService
{
    double CalculateDistanceKm(
        GeoCoordinate source,
        GeoCoordinate destination);

    bool IsWithinRadius(
        GeoCoordinate source,
        GeoCoordinate destination,
        double radiusKm);

    Task<GeoAddress?> ReverseGeocodeAsync(
        GeoCoordinate coordinate,
        CancellationToken cancellationToken);

    Task<GeoCoordinate?> GeocodeAsync(
        string address,
        CancellationToken cancellationToken);
}
```

---

# 31. External Provider Abstraction

Do not make business modules depend directly on a map provider SDK.

Use:

```csharp
public interface IGeocodingProvider
{
    Task<GeoAddress?> ReverseGeocodeAsync(...);
    Task<IReadOnlyList<GeoSuggestion>> SearchAsync(...);
}
```

For routing:

```csharp
public interface IRoutingProvider
{
    Task<RouteResult> CalculateRouteAsync(...);
}
```

This lets TechCare replace a provider without rewriting business logic.

---

# 32. Geocoding

**Geocoding:**

```text
Address -> Coordinates
```

Example:

```text
"Mansoura University"
        |
        v
Geocoding provider
        |
        v
Latitude + Longitude
```

Used when a user searches an address.

---

# 33. Reverse Geocoding

**Reverse geocoding:**

```text
Coordinates -> Human-readable address
```

Used when:

- user uses current location.
- user drops a pin.
- provider registers a location.

---

# 34. Geocoding Validation

Never blindly trust a geocoder.

Validate:

```text
result exists
coordinates valid
provider response usable
confidence/accuracy reasonable
address is acceptable
```

Store provider metadata only as needed.

Possible metadata:

```text
ProviderName
ProviderPlaceId
GeocodedAtUtc
Confidence
```

---

# 35. Address Search Suggestions

Frontend should debounce address searches.

Concept:

```text
User types:
"Mansoura U..."
        |
      wait 300–500 ms
        |
        v
GET /geocode/search
```

Do not hit the external provider for every keystroke.

---

# 36. Location Search Endpoint

Example:

```http
GET /api/v1/locations/geocode/search?q=Mansoura
```

Response:

```json
{
  "data": [
    {
      "placeId": "provider-id",
      "label": "Mansoura, Dakahlia, Egypt"
    }
  ]
}
```

---

# 37. Current Location Endpoint

Example:

```http
POST /api/v1/locations/current
```

This should be used for purpose-specific temporary location workflows rather than automatically creating a permanent address.

---

# 38. Nearby Search

Generic flow:

```text
Search Center
   |
   v
Spatial filtering
   |
   v
Business filtering
   |
   v
Distance calculation
   |
   v
Ranking
   |
   v
Pagination
   |
   v
Privacy-safe response
```

---

# 39. Nearby Doctors

Example:

```http
GET /api/v1/doctors/nearby?latitude=31.0001&longitude=31.4002&radiusKm=10&specialtyId=5&homeVisit=true
```

Server should:

1. validate coordinates.
2. spatially filter.
3. filter by specialty.
4. filter by verified/active providers.
5. enforce service-area rules.
6. calculate distance.
7. rank through Search/Discovery.
8. paginate.

---

# 40. Nearby Nurses

Same concept:

```http
GET /api/v1/nurses/nearby
```

Possible filters:

```text
service type
availability
rating
distance
price
```

---

# 41. Nearby Pharmacies

Example:

```http
GET /api/v1/pharmacies/nearby
```

Possible filters:

```text
medicine
open status
delivery
pickup
distance
rating
```

---

# 42. Nearby Laboratories

Example:

```http
GET /api/v1/laboratories/nearby
```

Possible filters:

```text
test
home collection
available slots
distance
price
rating
```

---

# 43. Blood Donor Proximity

Blood donation matching should prioritize:

```text
blood-type/clinical compatibility rules
availability
proximity
authorized facility workflow
```

Do not expose donor exact coordinates to patients.

Prefer:

```text
Nearby
< 2 km
2–5 km
5–10 km
```

or an area label.

---

# 44. Service Area Validation

Start simple with a radius:

```text
distance <= serviceRadiusKm
```

Future options:

- polygon.
- governorate.
- city.
- district.

---

# 45. Radius Workflow

```text
Patient destination
        |
        v
Distance from provider base
        |
   ┌────┴─────┐
   |          |
 Inside     Outside
   |          |
 Allow      Reject
```

---

# 46. Polygon Service Areas — Future

Provider can draw an area:

```text
Provider
   |
   v
Draw polygon
   |
   v
Save spatial polygon
   |
   v
Patient destination
   |
   v
Point-in-polygon
```

---

# 47. Search Radius Expansion

Optional algorithm:

```text
5 km
  |
  v
Enough results?
  |
  +-- Yes -> return
  |
  +-- No -> 10 km
                |
                v
              20 km
                |
                v
             maximum
```

Always enforce a configurable maximum radius.

---

# 48. Do Not Calculate Every Distance in Memory

Bad:

```csharp
var allProviders = await db.Providers.ToListAsync();

foreach (var p in allProviders)
{
    // distance calculation
}
```

This does not scale.

Better:

```text
Spatial database filter
        |
        v
Small candidate set
        |
        v
Distance/ranking
```

---

# 49. SQL Server Spatial

Recommended:

```text
geography
```

Example conceptually:

```sql
Location geography
```

Add spatial indexes for high-volume searchable points.

---

# 50. Spatial Indexes

Likely spatial entities:

```text
ProviderLocation.GeoPoint
PharmacyBranch.GeoPoint
LaboratoryBranch.GeoPoint
```

The exact index configuration should be tuned to real query patterns.

---

# 51. Transaction Location Snapshot

Suggested fields:

```text
Id
BookingId
AddressText
Latitude
Longitude
AccuracyMeters
Source
CapturedAtUtc
DistanceKm
DistanceType
CreatedAtUtc
```

For pharmacy/lab transactions, use typed snapshot entities when practical.

---

# 52. Distance-Based Pricing

Example:

```text
Base fee = 500
Included = 5 km
Distance = 12 km
Extra = 7 km
Rate = 5 EGP/km

Surcharge = 35 EGP
Final = 535 EGP
```

Architecture:

```text
Geo -> provides distance
Pricing -> calculates money
Finance -> owns quote/payment/refund/ledger
```

---

# 53. Never Trust Frontend Distance

Bad:

```json
{
  "distanceKm": 2.1,
  "price": 510
}
```

Correct:

```text
Frontend -> destination coordinates
Backend -> validates coordinates
Geo -> calculates distance
Pricing -> calculates quote
Finance -> processes payment
```

---

# 54. Location Privacy Classification

A useful classification is:

```text
Private
Approximate
Public
TransactionOnly
```

For public doctor search, return:

```text
2.4 km away
```

instead of patient coordinates.

---

# 55. Location Visibility Matrix

| Data | Patient | Provider | Pharmacy | Lab | Donor | Admin |
|---|---:|---:|---:|---:|---:|---:|
| Own saved address | ✅ | N/A | N/A | N/A | N/A | Restricted |
| Public branch location | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Patient exact booking destination | Own | Authorized | Relevant only | Relevant only | ❌ | Restricted |
| Donor exact coordinates | Own | ❌ | ❌ | ❌ | Own | Restricted |
| Provider public area | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

# 56. Exact Location Disclosure Timing

Recommended:

```text
Before booking:
    approximate distance / area

After authorized booking:
    exact destination

After cancellation:
    access removed according to retention policy
```

---

# 57. Purpose-Limited Access

Prefer:

```http
GET /api/v1/bookings/{bookingId}/visit-location
```

over:

```http
GET /api/v1/patients/{patientId}/location
```

The booking relation provides context for authorization.

---

# 58. IDOR Protection

Attack scenario:

```http
GET /api/v1/patient/addresses/124
```

Attacker changes the ID to another user's address.

Backend must verify:

```text
address.OwnerId == currentUser.Id
```

Do not rely on UI hiding IDs.

---

# 59. Provider Cannot Browse Patient Locations

Never expose a generic endpoint such as:

```http
GET /api/v1/patients/locations
```

Instead use a transaction-scoped endpoint with resource authorization:

```http
GET /api/v1/bookings/{bookingId}/visit-location
```

---

# 60. Admin Access

Admin should use least privilege.

Possible model:

```text
Normal admin:
    operational geo metrics

Authorized support workflow:
    specific exact location

Every sensitive access:
    audited
```

---

# 61. Location Consent

Distinguish:

### Device/browser permission

```text
May the browser share current GPS?
```

### Application authorization

```text
May this TechCare actor access this saved/private location?
```

GPS permission does not grant universal TechCare access.

---

# 62. Current Location Lifecycle

```text
Requested
   |
   v
Captured
   |
   v
Validated
   |
   v
Used
   |
   v
Expired / Discarded
```

Do not silently turn a temporary GPS point into a permanent address.

---

# 63. Stale Location

Every current-location point should have a timestamp.

Possible classification:

```text
Fresh
Stale
Expired
```

A stale GPS point should not automatically be presented as current.

---

# 64. Booking Snapshot Immutability

Critical invariant:

```text
Booking created with Location A
Patient later changes default address to B
Booking remains Location A
```

---

# 65. Pharmacy Branch Model

One pharmacy may have multiple branches:

```text
Pharmacy
  |
  +-- Branch A -> Location
  |
  +-- Branch B -> Location
```

Inventory belongs to the branch that actually holds stock.

---

# 66. Pharmacy Delivery

```text
Patient address
      |
      v
Pharmacy delivery-area check
      |
      +-- Outside -> unavailable
      |
      v
Distance
      |
      v
Delivery pricing
      |
      v
Order
```

---

# 67. Pharmacy Pickup

```text
Choose medicine
      |
      v
Choose branch
      |
      v
Show branch location
      |
      v
Patient navigates/picks up
```

---

# 68. Laboratory Search

```text
Patient selects test
      |
      v
Find labs offering test
      |
      v
Geo filter
      |
      v
Home collection?
      |
      v
Service area validation
      |
      v
Distance
      |
      v
Results
```

---

# 69. Laboratory Home Collection

```text
Patient
   |
   v
Select lab
   |
   v
Home collection
   |
   v
Choose address
   |
   v
Service-area check
   |
   v
Distance
   |
   v
Collection quote
   |
   v
Booking
```

---

# 70. Notifications and Geo

Geo may emit events such as:

```text
ProviderOutsideServiceArea
AddressChanged
ProviderLocationUpdated
ServiceAreaUpdated
```

Notification module owns:

- template.
- channels.
- preferences.
- retry.
- delivery.

Geo should not implement email/SMS/push logic.

---

# 71. Outbox Pattern

For important geo-related business events:

```text
Business transaction
   |
   +-- DB state change
   +-- Outbox event
   |
   v
Commit
   |
   v
Worker
   |
   v
Notification/other consumers
```

---

# 72. Security — Coordinate Spoofing

Clients can submit arbitrary coordinates.

For ordinary nearby search this may simply mean the user searches from another point.

For sensitive functions such as arrival verification or location-based eligibility, consider:

- freshness checks.
- accuracy thresholds.
- consistency checks.
- device attestation where available.

Do not assume GPS itself is tamper-proof.

---

# 73. Rate Limiting

High-value targets:

```text
geocode search
reverse geocode
nearby search
distance API
```

Reasons:

- provider quota abuse.
- database load.
- information enumeration.
- infrastructure cost.

---

# 74. Geo Caching

Appropriate caches can reduce external calls.

Example key:

```text
geo:reverse:{roundedLat}:{roundedLng}
```

Do not cache exact private patient location in a shared public cache.

---

# 75. Geocoding Provider Failure

Example:

```text
Patient drops pin
   |
   v
Reverse geocoder timeout
```

Fallback:

```text
Keep coordinates
Ask user to confirm/edit address text
Save if business rules allow
```

---

# 76. Provider Timeout

External calls should use:

```text
CancellationToken
bounded timeout
controlled retry
fallback
```

Do not hold a request indefinitely.

---

# 77. Logging

Good:

```text
Geo provider failed
operation=reverse-geocode
provider=...
durationMs=...
correlationId=...
```

Avoid general logs containing:

```text
patient latitude
patient longitude
full home address
```

unless explicitly required and secured.

---

# 78. Audit Logging

Audit sensitive events:

```text
AddressCreated
AddressUpdated
AddressDeleted
DefaultAddressChanged
ProviderLocationChanged
ServiceAreaChanged
PrivateLocationViewed
```

Useful metadata:

```text
ActorId
Action
SubjectType
SubjectId
Purpose
TimestampUtc
CorrelationId
```

---

# 79. Data Retention

Conceptual policy:

```text
Saved addresses:
    retained until deletion subject to transaction requirements

Booking snapshots:
    retained with the transaction according to platform policy

Live location:
    shortest practical period

Operational logs:
    limited retention

Raw external responses:
    shortest practical retention
```

Exact retention periods should be finalized by the project's product/legal requirements.

---

# 80. API Endpoints

## Addresses

```http
GET    /api/v1/locations/addresses
POST   /api/v1/locations/addresses
GET    /api/v1/locations/addresses/{id}
PUT    /api/v1/locations/addresses/{id}
DELETE /api/v1/locations/addresses/{id}
PATCH  /api/v1/locations/addresses/{id}/default
```

## Geocoding

```http
GET  /api/v1/locations/geocode/search?q=
POST /api/v1/locations/geocode
POST /api/v1/locations/reverse-geocode
```

## Current Location

```http
POST /api/v1/locations/current
```

## Distance

```http
POST /api/v1/locations/distance
```

## Nearby

```http
GET /api/v1/locations/nearby/doctors
GET /api/v1/locations/nearby/nurses
GET /api/v1/locations/nearby/pharmacies
GET /api/v1/locations/nearby/laboratories
```

---

# 81. Create Address DTO

```csharp
public sealed record CreateAddressRequest(
    string Label,
    string Country,
    string Governorate,
    string City,
    string? District,
    string? Street,
    string? BuildingNumber,
    string? Floor,
    string? Apartment,
    string? Landmark,
    string? FormattedAddress,
    double Latitude,
    double Longitude,
    double? AccuracyMeters,
    string Source,
    bool IsDefault
);
```

Backend derives the owner from the authenticated user, not a client-supplied owner ID.

---

# 82. Address Response DTO

```csharp
public sealed record AddressResponse(
    Guid Id,
    string Label,
    string FormattedAddress,
    string? Governorate,
    string? City,
    string? District,
    string? Street,
    bool IsDefault
);
```

Coordinates should be returned only when the purpose actually needs them.

---

# 83. Public Location DTO

```csharp
public sealed record PublicLocationResponse(
    string AreaName,
    double DistanceKm
);
```

Useful for privacy-safe discovery.

---

# 84. Authorized Visit Location DTO

```csharp
public sealed record VisitLocationResponse(
    string AddressText,
    double Latitude,
    double Longitude,
    double? AccuracyMeters
);
```

Only return through an authorized transaction context.

---

# 85. Nearby Result DTO

```csharp
public sealed record NearbyProviderResponse(
    Guid ProviderId,
    string Name,
    string Specialty,
    double DistanceKm,
    decimal BasePrice,
    bool Available,
    double Rating
);
```

Search/Discovery may enrich this with ranking metadata.

---

# 86. Distance Request

```json
{
  "source": {
    "latitude": 31.0001,
    "longitude": 31.4002
  },
  "destination": {
    "latitude": 31.0201,
    "longitude": 31.4302
  }
}
```

Response:

```json
{
  "straightLineDistanceKm": 4.21
}
```

---

# 87. API Error Model

Suggested codes:

```text
INVALID_COORDINATES
LOCATION_NOT_FOUND
GEOCODING_FAILED
REVERSE_GEOCODING_FAILED
LOCATION_PERMISSION_DENIED
LOCATION_TOO_INACCURATE
LOCATION_STALE
OUTSIDE_SERVICE_AREA
LOCATION_ACCESS_DENIED
GEO_PROVIDER_UNAVAILABLE
```

Example:

```json
{
  "success": false,
  "error": {
    "code": "OUTSIDE_SERVICE_AREA",
    "message": "This provider does not currently serve the selected location."
  }
}
```

---

# 88. Service Classes

Suggested application services:

```text
AddressService
GeoCalculationService
GeocodingService
NearbySearchService
ServiceAreaService
LocationPrivacyService
LocationAuditService
```

Exact class boundaries may follow the team's existing Clean Architecture/CQRS conventions.

---

# 89. Geo Query Service

Conceptual abstraction:

```csharp
public interface IGeoQueryService
{
    IQueryable<TEntity> WithinRadius<TEntity>(
        IQueryable<TEntity> source,
        GeoCoordinate center,
        double radiusKm);
}
```

The implementation should use database-side spatial filtering.

---

# 90. Domain Enums

```csharp
public enum LocationSource
{
    GPS,
    Network,
    ManualPin,
    AddressGeocoding,
    ProviderRegistration,
    AdminVerified
}
```

```csharp
public enum DistanceType
{
    StraightLine,
    Road
}
```

```csharp
public enum ServiceAreaType
{
    Radius,
    Polygon,
    AdministrativeArea
}
```

```csharp
public enum LocationVisibility
{
    Private,
    Approximate,
    Public,
    TransactionOnly
}
```

---

# 91. Database Tables

Suggested:

```text
Addresses
ProviderLocations
ServiceAreas
BookingLocationSnapshots
LocationAccessAudits
GeocodingCache
```

Future:

```text
LiveLocationSessions
LiveLocationPoints
Geofences
GeoEvents
```

---

# 92. Addresses Table

Possible columns:

```text
Id
UserId
Label
Country
Governorate
City
District
Neighborhood
Street
BuildingNumber
Floor
Apartment
Landmark
PostalCode
FormattedAddress
Latitude
Longitude
GeoPoint
AccuracyMeters
Source
IsDefault
CreatedAtUtc
UpdatedAtUtc
DeletedAtUtc
RowVersion
```

---

# 93. ProviderLocations Table

```text
Id
ProviderId
Latitude
Longitude
GeoPoint
AccuracyMeters
LocationType
ServiceRadiusKm
IsActive
CreatedAtUtc
UpdatedAtUtc
RowVersion
```

---

# 94. ServiceAreas Table

```text
Id
ProviderId
AreaType
RadiusKm
Polygon
Governorate
City
District
IsActive
CreatedAtUtc
UpdatedAtUtc
```

Only applicable fields should be populated for the chosen service-area type.

---

# 95. BookingLocationSnapshots Table

```text
Id
BookingId
AddressText
Latitude
Longitude
AccuracyMeters
Source
CapturedAtUtc
DistanceKm
DistanceType
CreatedAtUtc
```

For strongly typed transactions, specialized snapshot entities are often cleaner than generic polymorphic references.

---

# 96. Location Access Audit Table

```text
Id
ActorId
SubjectType
SubjectId
Action
Purpose
TimestampUtc
CorrelationId
IpAddress
```

Avoid storing raw coordinates in generic audit logs unless necessary.

---

# 97. Soft Delete

Saved addresses should normally support:

```text
DeletedAtUtc
DeletedBy
```

This protects audit/history and avoids accidental hard deletion.

Deleted records should not appear in normal APIs.

---

# 98. Concurrency

Example race:

```text
Request A -> Home becomes default
Request B -> Work becomes default
```

Expected:

```text
exactly one active default
```

Use:

- transaction.
- unique constraint where feasible.
- row-version/concurrency token.
- deterministic conflict handling.

---

# 99. Idempotency

Potential duplicate-request operations:

```text
Create Address
Save Current Location
Start Live Location Session
Confirm Location-Dependent Booking
```

Where duplicate submissions are dangerous, use idempotency keys.

---

# 100. Transaction Boundaries

For a booking:

```text
Validate request
      |
      v
Calculate/validate geo values
      |
      v
Begin short DB transaction
      |
      +-- booking
      +-- location snapshot
      +-- outbox event
      |
      v
Commit
```

Do not keep a DB transaction open while waiting on a slow external routing/geocoding provider.

---

# 101. Nearby Search Order

Recommended logical order:

```text
1. Validate location
2. Normalize filters
3. Spatial pre-filter
4. Radius filter
5. Business filters
6. Visibility/authorization
7. Calculate distance
8. Search ranking
9. Pagination
10. Privacy-safe DTO
```

---

# 102. Geo vs Search Boundary

### Geo owns

```text
Coordinates
Address
Distance
Radius
Point-in-area
Geo provider adapters
```

### Search owns

```text
What entities to return
Filters
Sorting
Ranking
Discovery experience
```

### Finance owns

```text
Money
Quotes
Payments
Refunds
Ledger
```

---

# 103. Location + Rating

Geo gives:

```text
Distance = 4.7 km
```

Rating gives:

```text
4.8 / 5
```

Search decides how to combine them.

---

# 104. Location + Availability

Geo:

```text
Distance = 5.2 km
```

Availability:

```text
Next slot = tomorrow 10:00
```

Search combines both.

---

# 105. Provider Verification

Provider locations may have status:

```text
Pending
Verified
Rejected
Suspended
```

Nearby search should normally expose only eligible active/verified provider locations.

---

# 106. Suspicious Location Changes

Example:

```text
Provider location:
Mansoura
        |
seconds later
Alexandria
```

Instead of immediately accusing the provider, generate:

```text
LocationAnomalyDetected
```

for operational review where appropriate.

---

# 107. Scheduling + Location — Future

Future scheduling may calculate:

```text
Previous appointment end
+
travel time
+
next appointment start
```

Example:

```text
Appointment A ends: 15:00
Appointment B starts: 15:05
Travel required: 30 min
```

System can flag a likely scheduling conflict.

This belongs to Scheduling/Availability, not core Geo.

---

# 108. Emergency Location — Future

Future workflow:

```text
Patient presses Emergency
        |
        v
Confirm emergency action
        |
        v
Capture current location
        |
        v
Create emergency incident
        |
        v
Share only with authorized recipients
        |
        v
Notify emergency workflow
```

Do not implement emergency location as unrestricted location sharing.

---

# 109. Location + Medical Data

Keep boundaries clear:

```text
Location Module
    = geographic information

Medical Module
    = medical information

Emergency Module
    = controlled combination
```

---

# 110. Frontend State

React may keep:

```text
currentLocation
selectedLocation
savedAddresses
locationPermission
locationLoading
locationError
```

Backend remains authoritative for persisted addresses, ownership, distance, service eligibility, and transactional snapshots.

---

# 111. React Location Hook

Conceptual:

```text
useCurrentLocation()
```

Responsibilities:

```text
ask permission
capture GPS
handle timeout
return coordinate
return accuracy
```

It must not calculate booking prices.

---

# 112. Location Selector UI

Example:

```text
┌─────────────────────────────┐
│ 📍 Searching near           │
│ Home — Mansoura             │
│                             │
│ [ Change ]                  │
└─────────────────────────────┘
```

---

# 113. Map UI

```text
┌───────────────────────────────┐
│ Search address                │
├───────────────────────────────┤
│                               │
│             📍                │
│             MAP               │
│                               │
├───────────────────────────────┤
│ [Use current location]        │
│ [Confirm location]            │
└───────────────────────────────┘
```

---

# 114. Accessibility

Location features must support:

- keyboard navigation.
- screen readers.
- visible labels.
- manual location entry.
- text alternative to a map.
- clear permission messages.
- usable fallback when the map cannot load.

The map must not be the only possible input method.

---

# 115. Search Near Me Flow

```text
Click "Near Me"
      |
      v
Request browser permission
      |
      v
Capture GPS
      |
      v
Send coordinates
      |
      v
Nearby API
      |
      v
Results
```

---

# 116. Saved Address Search Flow

```text
Patient chooses Home
       |
       v
Load saved coordinates
       |
       v
Call nearby endpoint
       |
       v
Show results
```

---

# 117. Change Location Mid-Search

```text
Searching near Home
       |
       v
User chooses Work
       |
       v
Invalidate Home results
       |
       v
Search near Work
```

Never silently show results for the wrong location.

---

# 118. Loading/Error States

Show:

```text
Detecting location...
```

Then:

```text
Location found
```

or:

```text
Couldn't access location
```

The UI should never stay in an infinite loading state.

---

# 119. Provider Search Example

Request:

```json
{
  "latitude": 31.0001,
  "longitude": 31.4002,
  "radiusKm": 10,
  "specialtyId": 5,
  "homeVisitOnly": true
}
```

Server:

```text
Coordinate validation
      |
Spatial filter
      |
Doctor filter
      |
Service-area check
      |
Availability
      |
Distance
      |
Ranking
      |
Pagination
```

---

# 120. Search Ranking Ownership

Distance is an input, not the whole ranking model.

Possible ranking factors:

```text
Distance
Rating
Availability
Price
Verification
```

Conceptual:

```text
Score =
    distanceScore * W1
  + ratingScore   * W2
  + availability  * W3
  + priceScore    * W4
```

Weights belong to Search/Discovery.

---

# 121. Privacy-Safe Search

For public discovery:

```text
Doctor:
2.4 km away
```

rather than exposing patient coordinates.

Always return the minimum required data.

---

# 122. Public Provider Location

A provider's public location should represent the business/service location intended for patients.

Do not accidentally publish:

```text
doctor's private residence
nurse's home
employee's private address
```

---

# 123. Blood Donor Privacy Rules

Never show:

```text
Donor exact street
Donor exact latitude/longitude
```

Prefer:

```text
Nearby donor available
Approximately 4 km away
Area name
```

and coordinate through an approved donation/facility workflow.

---

# 124. Location Bucketing

For privacy-sensitive discovery:

```text
Exact coordinate
      |
      v
Privacy transformation
      |
      v
Approximate area / distance bucket
```

---

# 125. Do Not Round Booking Coordinates

For confirmed home visits, arbitrary coordinate rounding may make operations worse.

Keep the exact transactional snapshot securely while limiting who can view it.

---

# 126. Purpose-Based Access

Suggested purposes:

```text
SEARCH
BOOKING
DELIVERY
COLLECTION
EMERGENCY
SUPPORT
AUDIT
```

The purpose can influence what precision is returned.

---

# 127. Caching Public Search

Nearby provider searches may use short TTL caches.

Avoid caching private data in shared caches.

Provider location changes should invalidate or naturally expire related public-search caches.

---

# 128. Cache Invalidation

After:

```text
Provider location updated
Service area updated
Pharmacy branch moved
Lab branch moved
```

invalidate the relevant public geo/search caches.

---

# 129. Monitoring

Useful metrics:

```text
Geocoding success rate
Reverse geocoding success rate
Geo provider latency
Spatial query latency
Nearby search latency
Cache hit ratio
Outside-service-area attempts
GPS denial rate
Invalid coordinate rate
External provider failures
```

Avoid identifying users in metric labels.

---

# 130. Health / Operational Monitoring

Operational dashboard may show:

```text
Mapped providers
Verified locations
Geo errors
Outside-area requests
Average nearby query latency
External provider availability
```

Use aggregated data for ordinary operational dashboards.

---

# 131. Configuration

Do not scatter geo constants through code.

Example:

```json
{
  "Geo": {
    "DefaultSearchRadiusKm": 10,
    "MaxSearchRadiusKm": 50,
    "MaxGpsAccuracyMeters": 500,
    "GeocodingTimeoutSeconds": 5,
    "CacheTtlSeconds": 300
  }
}
```

Examples only; final values are product/engineering decisions.

---

# 132. Map/API Key Security

Never commit private map provider keys into Git.

Use:

```text
environment variables
secret manager
protected configuration
```

Restrict browser keys according to the map provider's supported controls.

Never expose server-side secrets to React.

---

# 133. Provider Failure Resilience

The platform should still support:

```text
existing saved addresses
saved coordinates
nearby search using known coordinates
manual pin
```

even if the external geocoder is temporarily unavailable.

---

# 134. Testing Strategy

Tests should cover:

### Unit

- coordinate validation.
- distance formula.
- radius.
- address rules.

### Integration

- EF Core spatial queries.
- authorization.
- snapshots.
- service area.

### E2E

- GPS allowed.
- GPS denied.
- address creation.
- nearby search.
- booking.

### Security

- IDOR.
- privilege escalation.
- rate-limit bypass.
- exact-location disclosure.
- mass assignment.

---

# 135. Unit Tests — Coordinates

Test:

```text
Latitude 90 -> valid
Latitude 91 -> invalid
Latitude -90 -> valid
Latitude -91 -> invalid

Longitude 180 -> valid
Longitude 181 -> invalid
Longitude -180 -> valid
Longitude -181 -> invalid
```

Also test:

```text
NaN
Infinity
negative accuracy
```

---

# 136. Unit Tests — Distance

Test:

```text
same point -> 0
near points -> expected approximate distance
far points -> expected approximate distance
```

Use a tolerance for floating-point comparisons.

---

# 137. Unit Tests — Radius

```text
distance < radius -> true
distance = radius -> true
distance > radius -> false
```

Boundary behavior must be explicit.

---

# 138. Default Address Tests

```text
first address -> default
second chosen -> previous false
delete default -> valid fallback
cannot update another user's address
```

---

# 139. Booking Snapshot Test

```text
Create booking at Location A
Change patient's saved address to B
Read booking
Expected: Location A
```

Critical integration test.

---

# 140. Service Area Test

Example:

```text
Provider radius = 20 km

Destination = 10 km
Expected = allowed

Destination = 25 km
Expected = rejected
```

---

# 141. Pricing Integration Test

Example:

```text
Base = 500
Included = 5
Distance = 12
Rate = 5

Expected surcharge = 35
Expected total = 535
```

Pricing calculation itself belongs to the Financial/Pricing module.

---

# 142. Geocoder Failure Test

```text
Reverse geocoder timeout
```

Expected:

```text
No unhandled exception
Coordinates may remain available
Manual confirmation/fallback works
```

---

# 143. Authorization Tests

Test:

```text
Patient A cannot read Patient B address
Doctor A cannot read Doctor B private location
Doctor cannot access arbitrary patient location
Donor exact location is not exposed
Admin access matches policy
```

---

# 144. E2E — GPS Granted

```text
Login
   |
Profile
   |
Add Address
   |
Allow browser location
   |
Map pin appears
   |
Confirm
   |
Save
   |
Open doctor search
   |
Nearby doctors displayed
```

---

# 145. E2E — GPS Denied

```text
Add Address
   |
Deny permission
   |
Manual/search fallback shown
   |
User selects pin
   |
Save succeeds
```

---

# 146. E2E — Provider Location Change

```text
Doctor updates location
       |
Search patient nearby
       |
Expected distance changes
```

Make sure public-search caching does not produce unacceptable stale data.

---

# 147. E2E — Default Race

Fire two simultaneous default-setting requests.

Expected:

```text
Exactly one active default
```

---

# 148. Duplicate Request Test

Send the same address creation request twice with the same idempotency key.

Expected:

```text
One persisted address
```

---

# 149. Spatial Query Performance

Test realistic workloads such as:

```text
100 concurrent nearby searches
500 concurrent nearby searches
provider location updates
geocoder bursts
```

Measure:

```text
SQL latency
spatial query time
CPU
memory
cache behavior
external latency
```

---

# 150. Common Mistakes

Avoid:

```text
1. Storing only text addresses.
2. Trusting frontend distance.
3. Reversing latitude/longitude.
4. Exposing donor exact location.
5. Exposing patient location before authorization.
6. Rewriting historical booking addresses.
7. Calling geocoding on every keystroke.
8. Calculating every distance in application memory.
9. Missing spatial indexes.
10. No rate limiting.
11. Logging exact coordinates everywhere.
12. No GPS fallback.
13. Mixing Geo with Search/Finance.
14. Hard-coding one map provider.
15. No concurrency tests.
```

---

# 151. Complete Doctor Booking Scenario

```text
Patient Login
    |
    v
Choose Cardiology
    |
    v
Choose Home Visit
    |
    v
Select Home Address
    |
    v
Nearby Doctors
    |
    v
Geo filters/ranks candidates
    |
    v
Patient selects Doctor
    |
    v
Service-area validation
    |
    v
Distance
    |
    v
Pricing quote
    |
    v
Patient confirms
    |
    v
Booking + location snapshot
    |
    v
Outbox event
    |
    v
Provider notification
```

---

# 152. Complete Nurse Scenario

```text
Patient
   |
   v
Nurse search
   |
   v
Selected address
   |
   v
Nearby filter
   |
   v
Service-area check
   |
   v
Distance
   |
   v
Quote
   |
   v
Booking
   |
   v
Location snapshot
```

---

# 153. Complete Pharmacy Scenario

```text
Patient searches medicine
       |
       v
Inventory finds branches with stock
       |
       v
Geo filters nearby branches
       |
       v
Distance
       |
       v
Delivery or pickup
       |
       v
Order
```

---

# 154. Complete Laboratory Scenario

```text
Patient selects test
       |
       v
Labs offering test
       |
       v
Geo filter
       |
       v
Home collection support
       |
       v
Service area
       |
       v
Distance
       |
       v
Booking
```

---

# 155. Complete Blood Donation Scenario

```text
Blood request
       |
       v
Compatibility rules
       |
       v
Available donor candidates
       |
       v
Approximate proximity
       |
       v
Authorized donation workflow
       |
       v
Facility coordination
```

Exact donor addresses are not publicly disclosed.

---

# 156. Domain Events

Possible events:

```text
AddressCreated
AddressUpdated
AddressDeleted
DefaultAddressChanged
ProviderLocationUpdated
ServiceAreaUpdated
LocationValidated
PrivateLocationViewed
```

---

# 157. Event Ownership

Geo owns:

```text
AddressCreated
ProviderLocationUpdated
ServiceAreaUpdated
```

Booking owns:

```text
BookingCreated
BookingConfirmed
BookingCancelled
```

Finance owns:

```text
QuoteCreated
PaymentAuthorized
RefundCompleted
```

Notification consumes the events.

---

# 158. Project Structure

Suggested .NET organization:

```text
TechCare
├── Domain
│   └── Locations
│       ├── Entities
│       ├── ValueObjects
│       ├── Enums
│       └── Events
│
├── Application
│   └── Locations
│       ├── Addresses
│       ├── Geocoding
│       ├── Nearby
│       ├── ServiceAreas
│       └── Privacy
│
├── Infrastructure
│   └── Locations
│       ├── Persistence
│       ├── Spatial
│       ├── Providers
│       └── Caching
│
└── API
    └── Locations
        ├── Controllers
        ├── Requests
        └── Responses
```

---

# 159. Suggested Commands

If using CQRS:

```text
CreateAddressCommand
UpdateAddressCommand
DeleteAddressCommand
SetDefaultAddressCommand
UpdateProviderLocationCommand
UpdateServiceAreaCommand
```

---

# 160. Suggested Queries

```text
GetMyAddressesQuery
GetAddressByIdQuery
SearchGeocodingSuggestionsQuery
ReverseGeocodeQuery
FindNearbyDoctorsQuery
FindNearbyNursesQuery
FindNearbyPharmaciesQuery
FindNearbyLaboratoriesQuery
CheckServiceAreaQuery
```

---

# 161. Scrum/Jira Epic

```text
EPIC: Location & Geo Services
```

Suggested stories:

```text
LOC-001 Address CRUD
LOC-002 Default Address
LOC-003 Current GPS
LOC-004 Reverse Geocoding
LOC-005 Address Search
LOC-006 Spatial Data
LOC-007 Nearby Doctors
LOC-008 Nearby Nurses
LOC-009 Nearby Pharmacies
LOC-010 Nearby Laboratories
LOC-011 Service Areas
LOC-012 Distance Integration
LOC-013 Privacy
LOC-014 Audit
LOC-015 Rate Limiting
LOC-016 Integration Tests
LOC-017 E2E
LOC-018 Performance
```

---

# 162. Suggested Team Ownership

For a 5-person full-stack team:

### Developer 1

```text
Geo domain
Address CRUD
Database spatial model
```

### Developer 2

```text
Geocoding
Provider adapters
Spatial queries
```

### Developer 3

```text
Patient
Doctor
Nurse integration
Service areas
```

### Developer 4

```text
Pharmacy
Laboratory
Blood Donor integration
```

### Developer 5

```text
React maps
Location selector
Privacy UX
E2E/security tests
```

Ownership can change according to the team's actual branch/module assignments.

---

# 163. Definition of Ready

A location-related ticket is ready when:

```text
Business purpose defined
Actor defined
Precision defined
Authorization defined
Retention defined
Fallback defined
API contract defined
Success criteria defined
Failure behavior defined
Tests identified
```

---

# 164. Definition of Done

A geo feature is done when:

```text
Code complete
Validation complete
Authorization complete
Spatial migration complete
Business integration complete
Error handling complete
Audit complete
Frontend states complete
Unit tests complete
Integration tests complete
E2E test complete
Sensitive logging reviewed
Documentation updated
```

---

# 165. Critical Business Invariants

The following must always hold:

```text
1. Latitude and longitude are valid.
2. Every private address has an owner.
3. A user has at most one active default address.
4. Historical transaction locations are immutable.
5. Frontend distance is never trusted for money.
6. Exact private location is not returned without authorization.
7. Donor exact location is not publicly exposed.
8. Provider service location is distinct from private address where necessary.
9. Geo calculations are shared across the platform.
10. External provider failure does not corrupt local state.
11. Search does not own raw geo infrastructure.
12. Finance does not calculate raw geography.
```

---

# 166. Recommended MVP Implementation Order

### Phase 1 — Foundation

```text
GeoCoordinate
Validation
Addresses
Default address
SQL spatial mapping
```

### Phase 2 — Maps

```text
GPS
Manual pin
Reverse geocoding
Address search
```

### Phase 3 — Discovery

```text
Nearby doctors
Nearby nurses
Nearby pharmacies
Nearby labs
```

### Phase 4 — Business Rules

```text
Service radius
Booking snapshot
Distance integration
```

### Phase 5 — Security

```text
Policies
IDOR protection
Audit
Rate limiting
Privacy DTOs
```

### Phase 6 — Quality

```text
Integration tests
E2E
Performance
Caching
Failure handling
```

---

# 167. Architecture Diagram

```text
                         ┌───────────────────────┐
                         │       React UI        │
                         └───────────┬───────────┘
                                     |
                                     v
                         ┌───────────────────────┐
                         │   ASP.NET Core API    │
                         └───────────┬───────────┘
                                     |
                ┌────────────────────┼─────────────────────┐
                |                    |                     |
                v                    v                     v
        Location & Geo          Search/Discovery       Financial
                |                    |                     |
      ┌─────────┼─────────┐          |                     |
      |         |         |          |                     |
   Address    Geo     ServiceArea    |                     |
      |         |         |          |                     |
      └─────────┼─────────┘          |                     |
                |                    |                     |
                v                    v                     v
          SQL Server            Provider Queries         Pricing
          Spatial DB
                |
                v
        External Geo Providers
        ├── Geocoding
        ├── Reverse Geocoding
        └── Routing (future)
```

---

# 168. Final Decision Flow

```text
START
  |
  v
Does the feature require location?
  |
  +-- No --> Continue
  |
  +-- Yes
        |
        v
Do we have a valid selected location?
        |
     +--+--+
     |     |
    Yes    No
     |     |
     |     v
     |   GPS / Saved / Search / Pin
     |           |
     |           v
     |        Validate
     |           |
     +-----------+
                 |
                 v
Does the feature need exact/private location?
                 |
           +-----+-----+
           |           |
          Yes          No
           |           |
     Authorization   Approximate/public
           |
           +-----------+
                 |
                 v
Need distance?
                 |
           +-----+-----+
           |           |
          Yes          No
           |
     Calculate on server
           |
           v
Need service-area validation?
           |
        +--+--+
        |     |
       Yes    No
        |
   Validate area
        |
        v
Return purpose-specific response
        |
        v
END
```

---

# 169. Final Team Rules

Before merging any feature that uses location, ask:

```text
1. What location is required?
2. Exact or approximate?
3. Who owns it?
4. Who can see it?
5. Why is it needed?
6. How long is it retained?
7. Is GPS required?
8. What is the fallback?
9. Who calculates distance?
10. Straight-line or road?
11. Is service area involved?
12. Does it affect money?
13. Is historical data snapshotted?
14. Is the map provider abstracted?
15. Is the endpoint rate-limited?
16. Are sensitive coordinates excluded from logs?
17. Is IDOR prevented?
18. Are concurrency cases handled?
19. Are privacy/security tests present?
20. Is the behavior documented?
```

---

# 170. Most Important Separation

```text
LOCATION
    ↓
Coordinates / address / distance / area

SEARCH
    ↓
Find / filter / rank

BOOKING
    ↓
Create / confirm / cancel transaction

FINANCE
    ↓
Quote / payment / refund / ledger

NOTIFICATION
    ↓
Communicate events

AUTHORIZATION
    ↓
Control who can access exact location
```

This separation keeps TechCare maintainable and prevents Geo from becoming a giant business module.

---

# 171. Completion Status

**Location & Geo Services Workflow — COMPLETE**

Covered:

- Address management
- GPS
- Manual location
- Map pin
- Geocoding
- Reverse geocoding
- Distance calculation
- SQL Server spatial architecture
- Nearby search support
- Doctor/Nurse service areas
- Pharmacy/Laboratory proximity
- Blood donor privacy
- Booking snapshots
- Delivery/collection
- Privacy
- Authorization
- IDOR protection
- Auditing
- Rate limiting
- Caching
- External provider abstraction
- Failure handling
- Concurrency
- Idempotency
- APIs
- DTOs
- Entities
- React UX
- Testing
- E2E
- Performance
- Scrum ownership
- MVP/Future scope
- Cross-module boundaries
- End-to-end scenarios

---

# 172. Next Workflow

After Location & Geo Services, the next major cross-system workflow is:

```text
5. 🔎 Search & Discovery Workflow
```

The Search & Discovery module should consume the Location module for proximity instead of implementing its own coordinate/distance logic.
