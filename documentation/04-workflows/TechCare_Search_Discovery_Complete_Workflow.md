# TechCare — Search & Discovery Complete Workflow

> **Document Type:** Cross-System Workflow  
> **Platform:** TechCare Healthcare Platform  
> **Primary Stack:** ASP.NET Core Web API + SQL Server + React  
> **Architecture:** Modular Monolith / Clean Architecture friendly  
> **Status:** Ready for team implementation  
> **Scope:** Search, filtering, discovery, ranking, autocomplete, provider discovery, medicine discovery, laboratory discovery, blood-donation discovery, personalization, pagination, indexing, privacy, relevance, analytics, and integration with Location, Rating, Availability, Pharmacy, Laboratory, Doctor, Nurse, Blood Donor, Authentication, Notification, and Financial modules.

---

# 1. Purpose

The Search & Discovery module is responsible for helping TechCare users find the **right healthcare service, provider, pharmacy, laboratory, medicine, or donation opportunity** quickly and safely.

The module answers:

- What is the user looking for?
- Which entities match the query?
- Which filters apply?
- Which results should appear first?
- How should distance affect ranking?
- How should rating affect ranking?
- How should availability affect ranking?
- How should price affect ranking?
- Which results are verified and visible?
- Which results should never be shown?
- How should empty results be handled?
- How should search work in Arabic and English?
- How should autocomplete work?
- How should pagination work?
- How should search stay fast as data grows?

The most important architectural rule is:

> **Search decides what to find, filter, rank, and present. Location decides where something is and how far away it is.**

---

# 2. Core Architectural Principle

Search must not own the data of every module.

Instead:

```text
Search & Discovery
       |
       +---- Doctor Module
       +---- Nurse Module
       +---- Pharmacy Module
       +---- Laboratory Module
       +---- Blood Donor Module
       +---- Rating Module
       +---- Availability Module
       +---- Location Module
       +---- Financial/Pricing Module
```

Search is an orchestration/discovery capability.

The source modules remain the source of truth.

Example:

```text
Search says:
Doctor is 3.2 km away

Location says:
Distance = 3.2 km

Rating says:
Rating = 4.8

Availability says:
Next available slot = 18:00

Doctor says:
Specialty = Orthopedics
```

Search combines these values to build a useful result.

---

# 3. Business Goals

The Search module should provide:

1. Fast provider discovery.
2. Accurate filtering.
3. Relevant ranking.
4. Distance-aware results.
5. Availability-aware results.
6. Rating-aware results.
7. Price-aware results.
8. Verified-provider filtering.
9. Arabic/English search.
10. Autocomplete.
11. Typo tolerance where practical.
12. Pagination.
13. Stable sorting.
14. Privacy-safe results.
15. No unauthorized data leakage.
16. Search analytics.
17. Graceful empty-result handling.
18. A shared search experience across the platform.

---

# 4. Actors

## 4.1 Patient

The primary search user.

Can search for:

- Doctors.
- Nurses.
- Pharmacies.
- Medicines.
- Laboratories.
- Tests.
- Blood donation opportunities.
- Healthcare services.

## 4.2 Doctor

May search for:

- Patients only where an authorized business workflow explicitly permits it.
- Relevant requests/bookings.
- Clinical/service data.

The generic public discovery engine should not expose private patient records.

## 4.3 Nurse

May search authorized operational records.

## 4.4 Pharmacist

May search:

- Medicines.
- Orders.
- Pharmacy inventory.
- Authorized patient/order workflows.

## 4.5 Laboratory Staff

May search:

- Tests.
- Orders.
- Collection requests.
- Authorized patient records.

## 4.6 Blood Donor

May search donation opportunities and authorized requests.

## 4.7 Admin

May use advanced search for moderation, support, audits, and operations.

## 4.8 System

Handles:

- Query parsing.
- Filtering.
- Ranking.
- Pagination.
- Search analytics.
- Index updates.
- Suggestions.
- Security filtering.

---

# 5. Scope

## 5.1 MVP

The MVP should include:

- Doctor search.
- Nurse search.
- Pharmacy search.
- Laboratory search.
- Medicine search.
- Test search.
- Basic autocomplete.
- Filters.
- Distance sorting.
- Rating sorting/filtering.
- Price filtering.
- Availability filtering.
- Verified/active filtering.
- Pagination.
- Arabic/English normalization.
- Empty-state handling.
- Basic search analytics.

## 5.2 Future

Future capabilities may include:

- Full-text engine such as Elasticsearch/OpenSearch.
- Semantic search.
- AI-assisted query understanding.
- Personalized ranking.
- Search by natural language.
- Voice search.
- typo correction with learning.
- synonym intelligence.
- behavioral recommendations.
- trending searches.
- advanced geographic polygons.
- multi-criteria optimization.

Do not make AI/semantic search a dependency for the first working MVP.

---

# 6. Search Categories

TechCare should treat search types as separate use cases.

```text
Doctor Search
Nurse Search
Pharmacy Search
Medicine Search
Laboratory Search
Test Search
Blood Donation Search
Service Search
Admin Search
```

Do not force all entity types through one giant query unless there is a real product requirement for a unified global search.

---

# 7. Doctor Search

Example user request:

```text
Orthopedic doctor near me
```

Search may use:

```text
Specialty
Sub-specialty
Name
Gender (if product requires it)
Price range
Rating
Availability
Home visit
Distance
Verified status
Experience
```

---

# 8. Doctor Search Workflow

```text
Patient opens Doctors
        |
        v
Select specialty / enter query
        |
        v
Select location
        |
        v
Apply filters
        |
        v
Search candidate doctors
        |
        v
Security + eligibility filters
        |
        v
Location distance
        |
        v
Availability
        |
        v
Rating
        |
        v
Ranking
        |
        v
Pagination
        |
        v
Return result cards
```

---

# 9. Doctor Result Card

Recommended display:

```text
Dr. Ahmed Hassan
Orthopedic Specialist
⭐ 4.8 (231 reviews)
📍 3.2 km away
💰 From 500 EGP
🕒 Next slot: Today 19:00
✓ Verified
🏠 Home Visit

[View Profile]
[Book]
```

The card should show enough information to choose without exposing sensitive data.

---

# 10. Nurse Search

Search filters:

```text
Nursing service type
Experience
Rating
Price
Availability
Home visit
Distance
Verified status
```

Example:

```text
Nurse for elderly home care near Mansoura
```

---

# 11. Pharmacy Search

Patients can search by:

- Pharmacy name.
- Medicine.
- Area.
- Nearby.
- Open status.
- Delivery.
- Pickup.
- Rating.
- Distance.

Important:

> Pharmacy discovery must be branch-aware when a pharmacy has multiple branches.

---

# 12. Pharmacy Search Workflow

```text
Patient searches medicine
        |
        v
Inventory module finds branches with stock
        |
        v
Search filters by location
        |
        v
Distance calculation
        |
        v
Availability / delivery filters
        |
        v
Price comparison where supported
        |
        v
Rank results
        |
        v
Display branches
```

---

# 13. Medicine Search

Medicine search should separate:

```text
Medicine identity
        |
        +---- generic name
        +---- brand name
        +---- active ingredient
        +---- dosage/form
        +---- strength
```

Example:

```text
Paracetamol

Results:
- Brand A 500mg tablets
- Brand B 500mg tablets
- Generic Paracetamol 500mg
```

The Pharmacy/Inventory module remains authoritative for actual stock.

---

# 14. Laboratory Search

Users may search:

- Laboratory name.
- Medical test.
- Home collection.
- Nearby labs.
- Price.
- Availability.
- Rating.
- Verified status.

---

# 15. Laboratory Search Workflow

```text
Patient selects test
        |
        v
Find labs offering test
        |
        v
Apply location filter
        |
        v
Home collection filter
        |
        v
Availability
        |
        v
Price
        |
        v
Rating
        |
        v
Ranking
```

---

# 16. Blood Donation Discovery

Search should focus on:

```text
Blood type
Compatibility rules
Donation location / center
Availability
Approximate proximity
Urgency status
```

Important:

> Exact donor coordinates should never be exposed as public search data.

Clinical matching and eligibility remain under authorized healthcare rules/facilities.

---

# 17. Global Search vs Specialized Search

Two possible experiences:

## Global Search

```text
Search TechCare
```

Can return categorized results:

```text
Doctors
Nurses
Pharmacies
Medicines
Labs
Tests
```

## Specialized Search

```text
Find a Doctor
Find a Pharmacy
Find a Lab
Find Medicine
```

For MVP, specialized search should be the primary experience because it is easier to make precise and safe.

---

# 18. Global Search Result Structure

If a global search is implemented:

```text
Query: "orthopedic"

Doctors (12)
   ...

Services (4)
   ...

Articles / Help (optional future)
   ...
```

Do not mix unrelated record types in one undifferentiated list.

---

# 19. Query Lifecycle

```text
User enters query
        |
        v
Trim
        |
        v
Normalize
        |
        v
Validate
        |
        v
Parse filters
        |
        v
Build search request
        |
        v
Retrieve candidates
        |
        v
Filter
        |
        v
Rank
        |
        v
Paginate
        |
        v
Transform DTO
        |
        v
Return
```

---

# 20. Query Normalization

Normalize:

- Leading/trailing spaces.
- Repeated spaces.
- Arabic/English forms where supported.
- Common punctuation.
- Case.
- Common transliteration differences.

Example:

```text
"  Orthopedic doctor  "
```

becomes conceptually:

```text
"orthopedic doctor"
```

---

# 21. Arabic Search

TechCare users may type:

```text
دكتور عظام
دكتور عضم
عظام
orthopedic
ortho
```

The MVP can support:

- Unicode normalization.
- Arabic/English aliases.
- curated synonyms.
- transliteration mapping for common terms.

Do not promise perfect Arabic linguistic understanding in a basic SQL search implementation.

---

# 22. Search Synonyms

Example dictionary:

```text
orthopedic -> orthopedics -> عظام
cardiology -> cardiac -> قلب
pediatric -> pediatrics -> أطفال
pharmacy -> صيدلية
laboratory -> lab -> معمل -> مختبر
```

This can be stored in configuration/database rather than hard-coded in controllers.

---

# 23. Autocomplete

Autocomplete should return quick suggestions while the user types.

Example:

```text
User types: "card"

Suggestions:
Cardiology
Cardiologist
Cardiac checkup
```

---

# 24. Autocomplete Workflow

```text
User types
   |
   v
Debounce
   |
   v
Normalize query
   |
   v
Minimum characters
   |
   v
Suggestion lookup
   |
   v
Cache
   |
   v
Return top suggestions
```

Recommended minimum:

```text
2–3 characters
```

---

# 25. Autocomplete Debounce

Do not send an HTTP request for every keystroke.

Recommended frontend debounce:

```text
250–400 ms
```

Exact timing should be tuned through testing.

---

# 26. Search Minimum Query

Examples:

```text
q = ""
```

Possible behavior:

```text
Show popular categories / recent searches
```

For generic search:

```text
q length < 2
```

may return no server-side suggestions.

---

# 27. Filters

Search filters are domain-specific.

Generic filters:

```text
Location
Distance
Rating
Price
Availability
Verified
Service type
```

Doctor filters:

```text
Specialty
Home visit
Gender
Experience
```

Pharmacy filters:

```text
Delivery
Pickup
Open now
Medicine stock
```

Lab filters:

```text
Test
Home collection
Same-day result
```

---

# 28. Filter Ownership

The Search module orchestrates filters, but source modules own the actual business meaning.

Example:

```text
Rating > 4.5
```

Rating module owns rating calculation.

```text
Home visit = true
```

Doctor/Nurse module owns service capability.

```text
Distance < 10km
```

Location module owns distance.

---

# 29. Range Filters

Price:

```text
minPrice
maxPrice
```

Rating:

```text
minRating
```

Distance:

```text
radiusKm
```

Experience:

```text
minYears
```

Every numeric filter must have server-side min/max boundaries.

---

# 30. Filter Validation

Reject unreasonable values:

```text
radiusKm = -1
rating = 20
maxPrice < minPrice
pageSize = 100000
```

Return consistent validation errors.

---

# 31. Sorting

Possible sorts:

```text
Relevance
Distance
Rating
Price Low to High
Price High to Low
Availability
Experience
```

Default should normally be:

```text
Relevance
```

not simply distance.

---

# 32. Relevance vs Distance

The nearest result is not always the best result.

Example:

```text
Doctor A
1.2 km
Rating 3.9
No availability today

Doctor B
3.4 km
Rating 4.8
Available today
```

Doctor B may deserve higher ranking.

---

# 33. Ranking Model — MVP

A practical MVP score can combine:

```text
Distance
Rating
Availability
Verification
Price fit
Text relevance
```

Conceptually:

```text
Score =
    TextRelevance × W1
  + Availability × W2
  + Rating × W3
  + DistanceScore × W4
  + Verification × W5
  + PriceFit × W6
```

Weights are configurable.

---

# 34. Ranking Ownership

Search owns:

```text
How results are combined and ordered.
```

Search does not own the underlying truth of:

```text
rating
availability
price
location
verification
```

---

# 35. Distance Score

A simple normalized distance score may be used.

Example concept:

```text
closer -> higher score
farther -> lower score
```

The exact function must be tested.

Avoid simplistic rules that make tiny distance differences dominate every other factor.

---

# 36. Rating Score

Search should consume the aggregate rating provided by the Rating module.

Consider:

```text
average rating
review count
```

A provider with:

```text
5.0 from 2 reviews
```

should not automatically outrank:

```text
4.8 from 300 reviews
```

Use the Rating module's trusted/adjusted score where available.

---

# 37. Bayesian / Confidence Adjustment

A future ranking enhancement can use a confidence-adjusted rating rather than raw average.

Conceptually:

```text
AdjustedRating = f(mean, count, platform baseline)
```

Do not over-engineer this in the first MVP.

---

# 38. Availability Score

Potential signals:

```text
Available now
Available today
Available within 24h
Available later
No slots
```

Availability may be highly important for urgent care discovery.

---

# 39. Verification Score

Verified providers can receive a ranking signal.

Possible values:

```text
Verified = 1
Unverified = 0
```

But verification should usually also be a hard visibility requirement for regulated provider discovery.

---

# 40. Price Ranking

Price can be:

```text
Lowest first
Highest first
Best value
```

Do not equate low price with quality.

If the UI says:

```text
From 500 EGP
```

Search should not interpret the displayed price as the final transactional amount if travel or other charges apply.

---

# 41. Quote vs Search Price

Search can show:

```text
Starting price
```

Actual booking price must be recalculated at transaction time using:

```text
current rules
selected service
selected location
current fees
```

Search is not the final financial authority.

---

# 42. Nearby Search

Nearby search depends on Location & Geo.

Workflow:

```text
Search query
   |
   v
Location selected
   |
   v
Search candidates
   |
   v
Spatial filter
   |
   v
Distance result
   |
   v
Ranking
```

Search never reimplements coordinate math if the Location module already owns it.

---

# 43. Location Source in Search

The user can search near:

```text
Current Location
Home
Work
Other Saved Address
Map Point
```

The selected source should be explicit.

---

# 44. Location Selector UI

Recommended:

```text
Searching near:
📍 Home — Mansoura

[Change]
```

Click Change:

```text
Use current location
Home
Work
Choose on map
Enter address
```

---

# 45. Search Request Example

```http
GET /api/v1/search/doctors?
q=orthopedic&
specialtyId=5&
latitude=31.0001&
longitude=31.4002&
radiusKm=15&
minRating=4&
maxPrice=800&
homeVisit=true&
sort=relevance&
page=1&
pageSize=20
```

The API naming can be aligned with the final TechCare conventions.

---

# 46. Search Response Example

```json
{
  "items": [
    {
      "id": "...",
      "name": "Dr. Ahmed Hassan",
      "specialty": "Orthopedics",
      "rating": 4.8,
      "reviewCount": 231,
      "distanceKm": 3.2,
      "startingPrice": 500,
      "available": true,
      "nextAvailableAt": "2026-10-07T19:00:00Z",
      "verified": true,
      "homeVisit": true
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 42
}
```

Avoid returning fields not needed by the result card.

---

# 47. Search DTO Boundary

Search should return a **search projection**, not the full Doctor/Pharmacy/Lab entity.

Bad:

```text
Return entire Doctor entity
```

Good:

```text
DoctorSearchResult
```

This protects privacy and improves performance.

---

# 48. DoctorSearchResult

Suggested fields:

```text
DoctorId
DisplayName
Specialty
SubSpecialty
Rating
ReviewCount
DistanceKm
StartingPrice
AvailabilitySummary
Verified
HomeVisit
ProfileImageUrl
```

Do not include:

```text
Medical records
Private phone
Private address
Internal notes
Financial ledger
```

---

# 49. NurseSearchResult

Suggested:

```text
NurseId
DisplayName
Services
Rating
ReviewCount
DistanceKm
StartingPrice
Availability
Verified
ProfileImageUrl
```

---

# 50. PharmacySearchResult

Suggested:

```text
BranchId
PharmacyName
BranchName
DistanceKm
Rating
OpenNow
DeliveryAvailable
PickupAvailable
```

When searching for a medicine:

```text
MedicineName
Availability
Price
```

may also be returned from inventory/search projection.

---

# 51. LaboratorySearchResult

Suggested:

```text
LaboratoryId
BranchId
LaboratoryName
DistanceKm
Rating
HomeCollection
PriceFrom
Availability
Verified
```

---

# 52. MedicineSearchResult

Suggested:

```text
MedicineId
BrandName
GenericName
Strength
DosageForm
AvailabilityCount
PriceFrom
```

Actual pharmacy stock comes from the inventory source.

---

# 53. Search Service Interface

Suggested:

```csharp
public interface ISearchService
{
    Task<PagedResult<DoctorSearchResult>> SearchDoctorsAsync(
        DoctorSearchRequest request,
        CancellationToken cancellationToken);

    Task<PagedResult<NurseSearchResult>> SearchNursesAsync(
        NurseSearchRequest request,
        CancellationToken cancellationToken);

    Task<PagedResult<PharmacySearchResult>> SearchPharmaciesAsync(
        PharmacySearchRequest request,
        CancellationToken cancellationToken);

    Task<PagedResult<LaboratorySearchResult>> SearchLaboratoriesAsync(
        LaboratorySearchRequest request,
        CancellationToken cancellationToken);
}
```

Separate services are acceptable if the team prefers feature-based organization.

---

# 54. Search Request DTO

Example:

```csharp
public sealed record DoctorSearchRequest(
    string? Query,
    Guid? SpecialtyId,
    double? Latitude,
    double? Longitude,
    double? RadiusKm,
    decimal? MinPrice,
    decimal? MaxPrice,
    double? MinRating,
    bool? HomeVisit,
    bool? AvailableOnly,
    DoctorSortOption Sort,
    int Page,
    int PageSize
);
```

---

# 55. Search Validation

Validate:

```text
Page >= 1
PageSize within configured maximum
MinPrice >= 0
MaxPrice >= 0
MaxPrice >= MinPrice
MinRating between 0 and 5
Radius > 0
Radius <= configured maximum
Coordinates provided together
```

---

# 56. Coordinates Must Be Paired

Reject:

```text
latitude only
```

or:

```text
longitude only
```

unless the endpoint explicitly supports another mode.

---

# 57. Search Modes

A specialized endpoint may support:

```text
Text only
Location only
Text + Location
Filter only
```

Example:

```text
Nearby pharmacies
```

may have an empty text query but a valid location.

---

# 58. Empty Query Behavior

For pages such as nearby providers:

```text
q = null
```

is valid.

Search then becomes:

```text
filter + rank by location / relevance defaults
```

---

# 59. Empty Results

Never return a blank screen with no explanation.

Example:

```text
No doctors matched your current filters.

Try:
- increasing the distance
- removing a price filter
- choosing another specialty
- changing your location
```

The frontend should provide relevant recovery actions.

---

# 60. Zero-Result Query Logging

Track zero-result searches to improve the platform.

Example analytics event:

```text
search_zero_results
query = "cardiologee"
category = doctors
```

Do not store sensitive personal text indefinitely.

---

# 61. Search Analytics

Track:

```text
Search count
Successful searches
Zero-result searches
Click-through rate
Booking conversion
Filter usage
Autocomplete selection
Search latency
```

Potential event names:

```text
search_started
search_completed
search_zero_results
search_result_clicked
search_filter_applied
search_suggestion_selected
search_booking_started
```

---

# 62. Search Analytics Privacy

Search text can contain health information.

Example:

```text
"HIV clinic near me"
```

Therefore:

- Minimize raw query retention.
- Restrict analytics access.
- Avoid unnecessary user-identifying linkage.
- Define retention.
- Aggregate where possible.

---

# 63. Search History

Future optional feature:

```text
Recent searches
```

Example:

```text
Orthopedic doctor
Pharmacy near me
CBC test
```

This must be private to the user and deletable.

---

# 64. Recent Search API

Possible:

```http
GET    /api/v1/search/history
DELETE /api/v1/search/history/{id}
DELETE /api/v1/search/history
```

Search history is a user feature, not a global analytics feed.

---

# 65. Search Suggestions Sources

Suggestions can come from:

```text
Specialties
Provider names
Medicines
Tests
Categories
Popular searches
Recent searches
```

Suggestions should be ranked by relevance and safety.

---

# 66. Popular Searches

Future feature:

```text
Trending searches
```

Must be based on aggregated/thresholded data.

Avoid surfacing highly sensitive rare searches in a way that could identify a user or neighborhood.

---

# 67. Search Security

The Search module must apply authorization **before** returning results.

Do not:

```text
fetch unauthorized records
then hide them in React
```

The API should only return records the current user is allowed to discover.

---

# 68. IDOR Protection in Search

Search endpoints should never allow a filter such as:

```text
patientId=other-person-id
```

to expose private patient information unless the calling workflow explicitly authorizes it.

---

# 69. Public Provider Visibility

A provider is discoverable only if business conditions are satisfied.

Example:

```text
Account active
AND
Provider profile approved
AND
Required documents verified
AND
Provider not suspended
AND
Profile not private
```

---

# 70. Search Eligibility Pipeline

```text
Raw database records
        |
        v
Visibility filter
        |
        v
Verification filter
        |
        v
Business eligibility
        |
        v
Spatial filter
        |
        v
Text/filter match
        |
        v
Ranking
        |
        v
Pagination
```

Apply the cheapest/highest-elimination filters where practical without compromising correctness.

---

# 71. Suspended Providers

Suspended providers should not appear in normal public discovery.

Existing historical records can retain them as transaction history.

Search must not silently delete historical data.

---

# 72. Unavailable Providers

A provider can be:

```text
Visible but currently unavailable
```

or:

```text
Not accepting bookings
```

Product rules should determine whether they remain in discovery.

Possible approach:

```text
Available now -> higher rank
Unavailable -> lower rank
```

rather than always hiding them.

---

# 73. Search + Availability

Search should integrate with the Availability workflow.

Example:

```text
Available today
```

may require querying:

```text
next available slot
```

Do not duplicate slot-generation logic in Search.

---

# 74. Search + Rating

Search should consume:

```text
Aggregate rating
Review count
Visibility state
```

Do not calculate the review average separately in every search query.

---

# 75. Search + Location

Search gets:

```text
DistanceKm
WithinRadius
```

from Location.

Search should not contain:

```text
Haversine formula
```

if the shared Location service already provides it.

---

# 76. Search + Financial

Search may show:

```text
starting price
```

Financial owns:

```text
quote
payment
refund
ledger
provider payout
```

Search must not mutate financial state.

---

# 77. Search + Notification

Search can emit analytics or recommendation events.

Notification is responsible for:

```text
delivery
channels
retry
preferences
```

Do not send notifications directly from query logic.

---

# 78. Search + Complaint

Complaints may refer to:

```text
wrong provider listing
wrong price displayed
provider shown as available but unavailable
```

The Complaint module owns resolution.

Search can provide the relevant search snapshot if needed for investigation.

---

# 79. Search Snapshot

When a user starts a booking from search, consider capturing the relevant search-displayed values:

```text
ProviderId
DisplayedStartingPrice
DisplayedRating
DistanceAtSearch
SearchTimestamp
```

This is useful for dispute/debugging without making search the transaction source of truth.

---

# 80. Search Result Consistency

Search results can become stale immediately.

Example:

```text
Search shows doctor available at 18:00
Patient clicks
Doctor's slot is booked by another user
```

The Booking workflow must revalidate availability.

Search is a discovery layer, not a reservation guarantee.

---

# 81. Availability Race Condition

Never trust:

```text
available=true
```

from search as final confirmation.

Booking must recheck under transaction/locking rules.

---

# 82. Price Race Condition

Never trust:

```text
startingPrice=500
```

as final payment amount.

Financial/Pricing recalculates using current inputs.

---

# 83. Distance Race Condition

The selected address may change between search and booking.

Booking should validate the actual selected booking location.

---

# 84. Search Result Ranking Stability

When two results have similar scores, use deterministic tie-breakers.

Example:

```text
1. relevance score
2. availability score
3. distance
4. rating
5. provider ID
```

This prevents results from randomly changing between identical requests.

---

# 85. Pagination

For MVP:

```text
page + pageSize
```

For larger datasets:

```text
cursor-based pagination
```

Cursor pagination is especially useful with rapidly changing ranked datasets.

---

# 86. Page Size Limits

Example:

```text
Default: 20
Maximum: 50
```

Values are configurable.

Reject oversized requests.

---

# 87. Cursor Pagination

Conceptual cursor:

```text
score + tieBreaker + entityId
```

Avoid exposing internal database details unnecessarily.

---

# 88. SQL Search — MVP Option

A simple MVP can use:

```text
SQL Server
EF Core
Indexes
LIKE / normalized columns
```

This is enough for a controlled student project if the data volume is moderate.

---

# 89. Search Projection

Create optimized read projections.

Example:

```text
DoctorSearchProjection
```

containing only fields required for discovery.

This reduces joins and payload size.

---

# 90. Search Index Table

A future optimization may use a denormalized table:

```text
DoctorSearchIndex
```

Fields:

```text
DoctorId
NormalizedName
SpecialtyIds
VerificationState
Rating
ReviewCount
StartingPrice
SearchText
GeoPoint
AvailabilityHint
UpdatedAtUtc
```

This can make search much faster.

But source modules remain authoritative.

---

# 91. Search Index Synchronization

When a source entity changes:

```text
Doctor updated
    |
    v
Domain event
    |
    v
Outbox
    |
    v
Search index updater
    |
    v
DoctorSearchIndex updated
```

This is preferable to direct multi-module table writes.

---

# 92. Outbox Pattern for Search

Important events:

```text
DoctorProfilePublished
DoctorProfileUpdated
ProviderVerified
ProviderSuspended
ProviderLocationChanged
RatingAggregateUpdated
AvailabilityChanged
PharmacyBranchUpdated
MedicineInventoryChanged
LaboratoryTestCatalogUpdated
```

Search consumes these events to refresh its read model.

---

# 93. Search Index Delay

Read-model indexing may be eventually consistent.

Example:

```text
Doctor updates specialty
      |
      v
Source DB updated immediately
      |
      v
Search index updates after event processing
```

The product should tolerate small delays.

Critical transactional checks must use the source of truth.

---

# 94. Search Read Model vs Source Model

```text
Source Model
    = truth

Search Read Model
    = optimized discovery projection
```

Do not update domain source data through search index writes.

---

# 95. Search Cache

Useful for:

- Popular autocomplete suggestions.
- Stable public discovery queries.
- Specialty metadata.
- Category metadata.

Be careful with:

- Availability.
- Inventory stock.
- Financial quotes.
- Private data.

Short TTLs are preferred where data changes quickly.

---

# 96. Cache Key Example

Conceptually:

```text
search:doctors:
q=orthopedic:
specialty=5:
latBucket=...:
lngBucket=...:
radius=10:
minRating=4:
sort=relevance
```

Do not put private patient data into shared cache keys/values.

---

# 97. Cache Invalidation

Invalidate/expire search results when:

```text
provider suspended
provider profile changed
provider location changed
service area changed
critical availability signal changes
```

Not every change requires hard invalidation if TTL is sufficiently short.

---

# 98. Search Performance

Target the common path:

```text
validate
  |
  v
indexed filtering
  |
  v
small candidate set
  |
  v
ranking
  |
  v
pagination
```

Avoid:

```text
load all records
calculate everything in C#
then filter
```

---

# 99. Database Indexes

Potential indexes:

```text
NormalizedName
SpecialtyId
VerificationState
IsActive
StartingPrice
Rating
ReviewCount
Availability flags
```

Use actual query plans before adding excessive indexes.

---

# 100. Spatial Index

Location-based searches should use the spatial index provided by the Location design.

Search should call the spatial layer rather than creating its own geographic indexing approach.

---

# 101. Full-Text Search — Future

For larger search requirements, consider:

```text
Elasticsearch
OpenSearch
Azure AI Search
```

Benefits:

- Full text.
- Fuzzy matching.
- Synonyms.
- Ranking.
- Autocomplete.
- Facets.

Do not add one only because it sounds advanced.

---

# 102. Fuzzy Search

Future example:

```text
"ortopedic"
```

can suggest:

```text
Orthopedic
```

Use carefully for medical terms to avoid dangerous false matches.

---

# 103. Medical Search Safety

Search must not convert an ambiguous query into a medical diagnosis.

Example:

```text
"chest pain"
```

Search may recommend:

```text
Cardiology
Emergency services
```

only if product requirements and safety design support it.

It must not say:

```text
You have condition X
```

The Search module is not a diagnosis engine.

---

# 104. Search vs AI Assistant

Search:

```text
Find existing providers/services/products.
```

AI assistant:

```text
Natural-language assistance and future symptom guidance.
```

Keep these capabilities separated.

---

# 105. Query Intent

Future AI/search hybrid could classify:

```text
Doctor search
Medicine search
Lab test search
Pharmacy search
General information
```

MVP can use separate pages/tabs instead of AI intent detection.

---

# 106. Search Facets

Future faceted search may show counts:

```text
Specialty
  Orthopedics (18)
  Cardiology (32)

Home Visit
  Available (27)

Rating
  4+ (41)
```

Counts should come from the same visibility rules as results.

---

# 107. Facet Consistency

A common bug:

```text
Result says 10 items
Facet says 25
```

Use consistent filters and visibility logic.

---

# 108. Filter Combination

Filters are normally ANDed:

```text
Orthopedic
AND
Home Visit
AND
Rating >= 4
AND
Within 10 km
```

Some multi-select values use OR:

```text
Specialty in:
Orthopedics OR Sports Medicine
```

Define semantics per field.

---

# 109. Filter URL State

For React, encode shareable search state when useful.

Example:

```text
/doctors?specialty=orthopedics&radius=10&minRating=4
```

Do not put sensitive patient location coordinates into public/shareable URLs unnecessarily.

---

# 110. Search State Restoration

When returning from a provider profile:

```text
restore previous filters
restore page
restore selected location
```

This improves UX.

---

# 111. Search URL Security

Do not expose:

```text
private patient IDs
exact home coordinates
private medical keywords
```

in URLs unless explicitly necessary and protected.

---

# 112. Search Loading State

Use:

```text
Searching...
```

For skeleton cards:

```text
Doctor card skeleton
```

Avoid flashing "No results" before the request finishes.

---

# 113. Search Error State

Example:

```text
We couldn't load search results.
Please try again.

[Retry]
```

Do not show raw SQL/external provider errors.

---

# 114. Search Timeout

If the search takes too long:

```text
abort request
show retry
```

The backend should have controlled timeouts for expensive downstream calls.

---

# 115. Parallel Data Retrieval

Where safe, independent data can be retrieved in parallel.

Example:

```text
Candidate providers
       |
       +---- Rating summary
       +---- Availability summary
       +---- Location distance
```

Use bounded concurrency and avoid creating N+1 requests.

---

# 116. N+1 Problem

Bad:

```text
20 doctors
  -> 20 rating requests
  -> 20 availability requests
  -> 20 location requests
```

Better:

```text
batch / projection / joined read model
```

or a dedicated search read model.

---

# 117. Search Query Architecture

Potential implementation:

```text
Search API
   |
   v
Application Search Service
   |
   +---- Query Parser
   +---- Eligibility Filter
   +---- Location Adapter
   +---- Ranking Engine
   +---- Projection Builder
   |
   v
Search Repository / Read Model
```

---

# 118. Ranking Engine

Create a reusable component:

```csharp
IRankingEngine
```

Possible method:

```csharp
IReadOnlyList<T> Rank<T>(
    IReadOnlyList<T> candidates,
    RankingContext context);
```

For the MVP, implementation can be deterministic and rules-based.

---

# 119. Ranking Configuration

Example:

```json
{
  "SearchRanking": {
    "TextWeight": 0.30,
    "AvailabilityWeight": 0.20,
    "RatingWeight": 0.20,
    "DistanceWeight": 0.20,
    "VerificationWeight": 0.10
  }
}
```

Weights are examples, not final product values.

---

# 120. Ranking Feature Flags

Potential future feature:

```text
EnableNewRankingV2 = false
```

Allows safe experimentation.

---

# 121. Ranking Fairness

Avoid designs that permanently bury new providers simply because they have fewer reviews.

Potential mitigation:

```text
controlled exposure
minimum data thresholds
balanced exploration
```

Keep ranking understandable and auditable.

---

# 122. Provider Promotion

If TechCare later offers sponsored placements:

```text
Sponsored
```

must be visibly labeled and should not silently distort safety-critical ordering.

Promotion rules should be separate from ordinary relevance ranking.

---

# 123. Search Relevance Logs

For debugging, record structured non-sensitive information:

```text
SearchType
QueryHash / normalized query where appropriate
Filters
Sort
ResultCount
Latency
```

Avoid logging raw sensitive query text by default.

---

# 124. Search Correlation ID

Every search request should have a correlation ID where the platform uses one.

Useful for:

```text
frontend -> API -> Search -> downstream modules
```

---

# 125. Search API Rate Limiting

Protect:

```text
Global search
Autocomplete
Nearby search
```

Especially autocomplete because it creates high request volume.

---

# 126. Abuse Scenarios

Potential abuse:

```text
bot-generated queries
autocomplete flooding
provider enumeration
inventory enumeration
map scraping
price harvesting
```

Mitigations:

```text
rate limiting
pagination limits
auth where needed
bot detection
response minimization
```

---

# 127. Provider Enumeration

Do not let a user iterate IDs to retrieve private provider metadata.

Search results should expose only public discovery fields.

---

# 128. Pharmacy Inventory Privacy

Search may show:

```text
Medicine available
```

but not necessarily:

```text
exact internal stock quantity = 137
```

unless the product needs it.

Inventory details should remain controlled by Pharmacy/Inventory business rules.

---

# 129. Search Medicine Availability

Use an availability state such as:

```text
Available
Limited
Out of stock
Unknown
```

rather than exposing unnecessary inventory details.

---

# 130. Search by Medicine Name

Support:

```text
Brand
Generic
Arabic alias
English alias
Active ingredient
```

Return distinct medicine identities where possible instead of duplicate branch rows for the same product.

---

# 131. Medicine Branch Expansion

User first sees:

```text
Paracetamol 500mg
Available at 12 pharmacies
```

Then:

```text
View nearby pharmacies
```

This separates product discovery from branch discovery.

---

# 132. Laboratory Test Search

Search by:

```text
Test name
Alias
Category
Specimen
```

Example:

```text
CBC
Complete Blood Count
صورة دم كاملة
```

---

# 133. Test Availability

A lab can:

```text
offer test
```

without necessarily having:

```text
home collection
same-day result
current appointment slot
```

Search should distinguish these capabilities.

---

# 134. Doctor Specialty Search

Use a canonical specialty catalog.

Avoid allowing free text to become the official specialty truth.

Example:

```text
SpecialtyId = 5
Name = Orthopedics
```

Search aliases can point to the same canonical record.

---

# 135. Specialty Catalog

Fields:

```text
Id
NameEn
NameAr
Slug
ParentId
IsActive
SortOrder
```

Possible hierarchy:

```text
Medicine
  -> Cardiology
  -> Orthopedics
  -> Pediatrics
```

---

# 136. Search Slugs

SEO-friendly future routes:

```text
/doctors/orthopedics
/pharmacies/mansoura
/labs/cbc
```

Slugs should map to canonical IDs.

---

# 137. Search Engine Independence

Business code should depend on:

```text
ISearchProvider
```

rather than directly on Elasticsearch/SQL implementation.

---

# 138. Search Provider Interface

Conceptually:

```csharp
public interface ISearchProvider<TRequest, TResult>
{
    Task<PagedResult<TResult>> SearchAsync(
        TRequest request,
        CancellationToken cancellationToken);
}
```

Do not force all search use cases into an overly generic abstraction if it harms clarity.

---

# 139. SQL Implementation

MVP implementation can be:

```text
EfCoreSearchProvider
```

Future:

```text
ElasticSearchProvider
```

The application contract stays stable where practical.

---

# 140. Search Controller Endpoints

Suggested:

```http
GET /api/v1/search/doctors
GET /api/v1/search/nurses
GET /api/v1/search/pharmacies
GET /api/v1/search/medicines
GET /api/v1/search/laboratories
GET /api/v1/search/tests
GET /api/v1/search/suggestions
```

Optional:

```http
GET /api/v1/search/global
```

---

# 141. Search Suggestions Endpoint

Example:

```http
GET /api/v1/search/suggestions?q=card&category=doctors
```

Response:

```json
{
  "items": [
    {
      "text": "Cardiology",
      "type": "Specialty",
      "id": "..."
    }
  ]
}
```

---

# 142. Search Result Types

Use explicit types:

```text
Doctor
Nurse
Pharmacy
Medicine
Laboratory
Test
Specialty
Service
```

This improves frontend rendering and analytics.

---

# 143. Search Service Layer

Suggested responsibilities:

```text
DoctorDiscoveryService
NurseDiscoveryService
PharmacyDiscoveryService
MedicineDiscoveryService
LaboratoryDiscoveryService
TestDiscoveryService
SuggestionService
GlobalSearchService
```

A single Search facade can orchestrate these if desired.

---

# 144. Global Search Facade

Conceptual:

```csharp
public interface IGlobalSearchService
{
    Task<GlobalSearchResult> SearchAsync(
        GlobalSearchRequest request,
        CancellationToken cancellationToken);
}
```

It should call specialized search capabilities rather than contain every rule itself.

---

# 145. Result Group Limits

Global search should return limited grouped results:

```text
Doctors: 5
Pharmacies: 5
Medicines: 5
Labs: 5
```

Then:

```text
View all
```

This keeps the first screen fast.

---

# 146. Search Query Length

Set sensible bounds:

```text
Minimum: 1–2 chars for suggestions
Maximum: e.g. 100 chars
```

Exact limits are configurable.

---

# 147. Search Input Sanitization

User text must be treated as data, not SQL/code.

Use parameterized queries/EF Core expressions.

Do not concatenate SQL.

---

# 148. Wildcard Abuse

Special characters can create expensive patterns.

Normalize/escape where appropriate.

Do not allow arbitrary SQL wildcard expressions through query parameters.

---

# 149. Search SQL Injection

Use:

```text
EF Core parameterization
```

or a safe search client for external search engines.

Never concatenate untrusted query text into raw SQL.

---

# 150. Search Index Security

If an external search engine is used:

```text
Private index != public internet
```

Require authentication and network restrictions appropriate to deployment.

---

# 151. Search Data Synchronization

When source data changes:

```text
source update
   |
   v
commit
   |
   v
outbox event
   |
   v
search index worker
```

The index worker must be idempotent.

---

# 152. Idempotency — Indexing

If the same event is delivered twice:

```text
event A
 event A
```

result should still be one correct index state.

Use:

```text
EventId
Version
UpdatedAt
```

where appropriate.

---

# 153. Out-of-Order Events

Example:

```text
Doctor update v4
Doctor update v5
```

If v5 is processed first and v4 later, search should not regress.

Use an entity version or timestamp strategy.

---

# 154. Search Index Rebuild

The system should support rebuilding the search projection from source truth.

Example:

```text
RebuildDoctorSearchIndex
```

Needed after:

- schema change.
- ranking changes.
- corruption.
- migration.

---

# 155. Backfill

New search fields can be backfilled from source data.

Avoid manual inconsistent edits to search indexes.

---

# 156. Search Health Check

Possible checks:

```text
Search database available
Search index available
Queue/outbox healthy
Suggestion cache healthy
```

---

# 157. Search Metrics

Monitor:

```text
search_requests_total
search_success_total
search_error_total
search_zero_results_total
search_latency_ms
autocomplete_latency_ms
index_lag_seconds
index_update_failures
cache_hit_ratio
```

---

# 158. Search Latency Breakdown

Measure:

```text
Parsing
DB/search engine
Location
Ranking
Projection
Serialization
```

Do not only monitor total latency.

---

# 159. Slow Query Detection

Alert on:

```text
95th percentile latency
99th percentile latency
```

Tune actual query plans instead of guessing.

---

# 160. Search Load Test

Examples:

```text
100 concurrent doctor searches
500 concurrent autocomplete requests
100 concurrent pharmacy searches
```

Measure:

```text
latency
CPU
memory
SQL
external provider calls
```

---

# 161. Search Test Strategy

Test layers:

```text
Unit
Integration
Contract
Security
Performance
End-to-End
```

---

# 162. Unit Test — Query Normalization

Test:

```text
" ORTHOPEDIC " -> "orthopedic"
multiple spaces
Arabic normalization
empty query
very long query
```

---

# 163. Unit Test — Filters

Examples:

```text
minRating = 4
maxPrice = 800
radius = 10
homeVisit = true
```

Expected filtering rules must be deterministic.

---

# 164. Unit Test — Ranking

Create controlled candidates:

```text
A: 1km, rating 3.8, available
B: 4km, rating 4.9, available
C: 2km, rating 4.5, unavailable
```

Assert exact ranking according to the configured scoring model.

---

# 165. Unit Test — Tie Breaker

Two candidates with identical score.

Expected deterministic order:

```text
ProviderId ascending
```

or another documented tie-break rule.

---

# 166. Integration Test — Doctor Search

```text
Seed doctors
Seed specialties
Seed ratings
Seed locations
Seed availability

Search orthopedics within 10km

Expected:
only valid providers
correct distance
correct filters
correct order
```

---

# 167. Integration Test — Pharmacy Medicine Search

```text
Medicine A
Pharmacy 1 has stock
Pharmacy 2 has no stock
Pharmacy 3 has stock
```

Search medicine.

Expected:

```text
only eligible branches
correct location ordering
inventory state respected
```

---

# 168. Integration Test — Laboratory Search

```text
Test = CBC
Lab A offers CBC + home collection
Lab B offers CBC only
```

Filter:

```text
home collection = true
```

Expected:

```text
Lab A only
```

---

# 169. Security Test — Provider Visibility

```text
Suspended provider
```

Expected:

```text
not returned in public search
```

---

# 170. Security Test — Patient Privacy

Search APIs must not return private patient records.

Test adversarial filters and IDs.

---

# 171. Security Test — Role Restrictions

Examples:

```text
Patient cannot use admin global search
Admin features require admin role
Provider cannot enumerate arbitrary patient records
```

---

# 172. Security Test — Injection

Queries:

```text
' OR 1=1 --
<script>
...
```

Expected:

```text
safe handling
no SQL injection
no HTML injection through response rendering
```

---

# 173. E2E — Doctor Search

```text
Login as patient
Open Doctors
Select Orthopedics
Select Home
Set distance 10km
Set rating 4+
Search
Open result
Book
```

At booking:

```text
availability + price + location are revalidated
```

---

# 174. E2E — GPS Denied

```text
User opens nearby doctors
Denies GPS
```

Expected:

```text
address selector appears
manual location works
```

---

# 175. E2E — Empty Results

Use restrictive filters.

Expected:

```text
friendly empty state
suggested recovery actions
```

---

# 176. E2E — Arabic Search

Search:

```text
دكتور عظام
```

Expected relevant orthopedic results.

Exact matching quality depends on configured aliases/indexing strategy.

---

# 177. Search Analytics Test

Verify events:

```text
search_started
search_completed
search_result_clicked
```

Do not emit duplicates because of React rerenders.

---

# 178. Debounce Test

Typing:

```text
c
ca
car
card
```

should not produce four unbounded search requests when debounce is configured correctly.

---

# 179. Search Cache Test

Same public query twice:

```text
first = cache miss
second = cache hit
```

After invalidation/TTL:

```text
new data returned
```

---

# 180. Indexing Test

```text
Doctor specialty changes
   |
   v
Event emitted
   |
   v
Search index updated
   |
   v
New search result reflects change
```

---

# 181. Availability Staleness Test

```text
Search says slot available
Slot is booked by another user
```

Expected:

```text
Booking rejects/requires re-selection
```

Search must not create a false reservation.

---

# 182. Price Staleness Test

```text
Search says From 500
Pricing rules changed
User starts booking
```

Expected:

```text
Current quote calculated by financial/pricing workflow
```

---

# 183. Search Snapshot Test

When booking starts, capture:

```text
search context
```

for audit/debug purposes where product requirements require it.

---

# 184. Empty Search Handling

Possible default behavior:

```text
Show popular specialties
or
show nearby providers
```

Do not issue an expensive unrestricted database scan.

---

# 185. Recommended Search Page — Doctors

```text
------------------------------------------------
Find a Doctor

[ 🔎 Search specialty or doctor ]

📍 Searching near: Home   [Change]

Filters:
[ Specialty ] [ Rating ] [ Price ] [ Availability ]
[ Home Visit ]

Sort:
[ Relevance ▼ ]
------------------------------------------------

Doctor Card
Doctor Card
Doctor Card
...
```

---

# 186. Recommended Search Page — Pharmacy

```text
------------------------------------------------
Find Medicine / Pharmacy

[ 🔎 Search medicine or pharmacy ]

📍 Near: Home

Filters:
[ In Stock ] [ Delivery ] [ Pickup ] [ Distance ]
------------------------------------------------
```

---

# 187. Recommended Search Page — Laboratory

```text
------------------------------------------------
Find a Laboratory

[ 🔎 Search test or lab ]

📍 Near: Home

Filters:
[ Home Collection ] [ Price ] [ Rating ] [ Distance ]
------------------------------------------------
```

---

# 188. Mobile/Responsive UX

Search should remain usable on small screens.

Recommended:

```text
Search bar
Location selector
Filter button
Sort button
Cards
```

Use a filter drawer/bottom sheet on mobile layouts.

---

# 189. Accessibility

Support:

- Keyboard navigation.
- Screen readers.
- Clear filter labels.
- Accessible sort control.
- Accessible result cards.
- Visible loading/empty/error states.
- Non-color-only status indicators.

---

# 190. Search Result Card Accessibility

A card should expose:

```text
Name
Role/specialty
Rating
Distance
Availability
Primary action
```

through semantic HTML and accessible labels.

---

# 191. Search Deep Links

A result can link to:

```text
/provider/{id}
/pharmacy/{branchId}
/lab/{id}
/medicine/{id}
```

The profile endpoint should still enforce authorization/visibility.

---

# 192. Search-to-Booking Flow

```text
Search
  |
  v
Result
  |
  v
Provider Profile
  |
  v
Select service
  |
  v
Select location
  |
  v
Availability
  |
  v
Price quote
  |
  v
Booking
```

Search never creates the booking itself.

---

# 193. Search-to-Order Flow — Pharmacy

```text
Search medicine
  |
  v
Choose branch
  |
  v
View availability
  |
  v
Select delivery/pickup
  |
  v
Order
```

---

# 194. Search-to-Booking Flow — Laboratory

```text
Search test
  |
  v
Choose lab
  |
  v
Choose appointment / collection
  |
  v
Location validation
  |
  v
Quote
  |
  v
Booking
```

---

# 195. Search-to-Donation Flow

```text
Blood request
  |
  v
Compatibility screening
  |
  v
Nearby eligible opportunities
  |
  v
Authorized donation coordination
```

Search does not own clinical eligibility.

---

# 196. Search Data Retention

Consider retaining only what is operationally necessary:

```text
Search analytics events
Search history if user enabled it
Search index projections
```

Avoid indefinite storage of sensitive raw search text.

---

# 197. Search History Privacy

User controls may include:

```text
Delete one search
Delete all searches
Disable history
```

---

# 198. Search and Location Privacy

A search near a patient's home can itself reveal sensitive information.

Therefore analytics should avoid combining:

```text
exact location + sensitive medical query + identity
```

unless there is a justified and protected requirement.

---

# 199. Search Data Classification

Suggested classes:

```text
Public:
  provider name, specialty, public branch location

Sensitive:
  exact patient location
  personal search history

Highly sensitive:
  patient medical records
  private medical search history where linked to identity
```

Apply appropriate access controls.

---

# 200. Search API Authorization Matrix

| Search | Patient | Doctor | Nurse | Pharmacist | Lab | Donor | Admin |
|---|---:|---:|---:|---:|---:|---:|---:|
| Public doctors | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Public nurses | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Public pharmacies | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Public labs | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Medicine discovery | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Patient private search | Own workflows | Authorized | Authorized | Authorized | Authorized | ❌ | Restricted |
| Admin operational search | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

This table is a baseline; actual permissions should follow each domain workflow.

---

# 201. Admin Search

Admin may need:

```text
Users
Providers
Pharmacies
Labs
Orders
Bookings
Complaints
```

But admin search should be separated from public discovery.

Example:

```text
PublicDoctorSearchService
AdminProviderSearchService
```

This prevents accidental privilege leakage.

---

# 202. Admin Filters

Examples:

```text
Status
Verification
Created date
Suspension state
Complaint count
```

These are operational filters, not public search filters.

---

# 203. Search Architecture Boundary

Keep these boundaries strict:

```text
Search
  -> discover

Location
  -> distance / geo

Rating
  -> reputation

Availability
  -> slots

Finance
  -> money

Booking
  -> reservation

Notification
  -> delivery
```

---

# 204. Search Event Ownership

Search can emit:

```text
SearchExecuted
SearchResultClicked
SearchZeroResults
```

Other modules can consume them for analytics/recommendations.

---

# 205. Search Event Payload

Example:

```json
{
  "eventType": "SearchExecuted",
  "searchType": "Doctor",
  "resultCount": 21,
  "filtersApplied": ["Specialty", "Distance", "Rating"],
  "timestampUtc": "2026-10-07T15:00:00Z"
}
```

Avoid including exact patient coordinates by default.

---

# 206. Outbox for Analytics Events

For durable analytics:

```text
Search request
   |
   v
Business event / analytics event
   |
   v
Outbox
   |
   v
Analytics consumer
```

Do not slow down the search request by synchronously calling multiple analytics systems.

---

# 207. Search Error Codes

Suggested:

```text
INVALID_SEARCH_QUERY
INVALID_FILTER
INVALID_LOCATION
SEARCH_LIMIT_EXCEEDED
SEARCH_TIMEOUT
SEARCH_UNAVAILABLE
NO_RESULTS
UNAUTHORIZED_SEARCH
```

---

# 208. No Results Is Not a Server Error

`NO_RESULTS` should normally be represented as a successful search with:

```text
items = []
```

plus metadata if needed.

HTTP 500 should not be used for empty search results.

---

# 209. Search Provider Outage

If an external search provider is down:

Possible fallback:

```text
basic SQL search
```

when supported.

Or return:

```text
Search temporarily unavailable
```

without exposing infrastructure details.

---

# 210. Fallback Strategy

Recommended:

```text
Primary search provider
        |
        +---- healthy -> use
        |
        +---- unhealthy -> controlled fallback
```

Do not attempt uncontrolled multi-provider retries on every request.

---

# 211. Search Consistency Model

Public discovery can be:

```text
eventually consistent
```

Transactional checks must be:

```text
source-of-truth consistent
```

Examples:

```text
Search availability = eventually consistent
Booking availability check = authoritative
```

---

# 212. Search Snapshot Time

Include:

```text
searchedAtUtc
```

when useful for debugging and dispute handling.

---

# 213. Search Result Freshness

Potential response metadata:

```json
{
  "generatedAtUtc": "2026-10-07T15:00:00Z"
}
```

This can help frontend/backend diagnostics.

---

# 214. Search Facet Counts and Staleness

Facet counts may lag source data if backed by an index.

UI can treat them as informational, not transactional truth.

---

# 215. Search Relevance Tuning

Improve ranking using measured signals:

```text
CTR
booking conversion
zero-result rate
filter abandonment
```

But avoid optimizing only for clicks if that hurts healthcare relevance or safety.

---

# 216. Search Quality Metrics

Key product metrics:

```text
Zero-result rate
Search-to-profile CTR
Search-to-booking conversion
Average search latency
Repeat search rate
Filter usage
Autocomplete success rate
```

---

# 217. Search Safety Metrics

Track:

```text
Unauthorized result exposure incidents
Provider visibility errors
Incorrect availability reports
Price mismatch reports
Location mismatch reports
```

These are higher priority than raw CTR for healthcare workflows.

---

# 218. Price Mismatch Monitoring

If users frequently report:

```text
Search says 500
Checkout says 800
```

investigate whether the UI is misleading or whether the product needs clearer price labels.

---

# 219. Availability Mismatch Monitoring

If:

```text
Search says available
Booking says unavailable
```

frequently occurs:

- improve availability freshness.
- improve UI wording.
- reduce index lag.
- keep final revalidation.

Never remove final revalidation just to improve conversion.

---

# 220. Location Mismatch Monitoring

If distance appears wrong:

Check:

```text
coordinate order
SRID
provider data
selected search location
cache
```

Coordinate-order bugs are common and severe.

---

# 221. Search Ranking Auditability

For important support/debugging flows, the backend should be able to explain the main ranking signals.

Example internal diagnostic:

```text
Text score: 0.92
Availability: 1.00
Rating: 0.96
Distance: 0.78
Verification: 1.00
Final: 0.91
```

Do not expose internal scoring details to ordinary users unless product requires it.

---

# 222. Ranking Explainability for Admin

Admin/support may benefit from:

```text
Why was this provider ranked here?
```

This helps debugging and moderation.

---

# 223. Search Contract Versioning

When changing search response shape:

```text
v1
v2
```

or use backward-compatible additions.

Do not unexpectedly break the React frontend.

---

# 224. Search API Version

Recommended:

```text
/api/v1/search/...
```

Keep consistent with the rest of TechCare.

---

# 225. DTO Versioning

Potential future:

```text
DoctorSearchResultV2
```

Use only when meaningful breaking changes occur.

---

# 226. Repository Structure

Suggested:

```text
TechCare
│
├── Domain
│   └── Search
│       ├── Models
│       ├── Events
│       └── Enums
│
├── Application
│   └── Search
│       ├── Doctors
│       ├── Nurses
│       ├── Pharmacies
│       ├── Medicines
│       ├── Laboratories
│       ├── Tests
│       ├── Suggestions
│       └── Ranking
│
├── Infrastructure
│   └── Search
│       ├── SQL
│       ├── Projections
│       ├── Indexes
│       ├── Cache
│       └── Providers
│
└── API
    └── Search
        ├── Controllers
        ├── Requests
        └── Responses
```

---

# 227. Query Objects

Possible classes:

```text
DoctorSearchQuery
NurseSearchQuery
PharmacySearchQuery
MedicineSearchQuery
LaboratorySearchQuery
TestSearchQuery
SuggestionQuery
```

---

# 228. Search Query Builder

A dedicated builder can help:

```text
Search filters
    -> validated query specification
    -> database expression / search-engine request
```

Avoid dynamically building unsafe strings.

---

# 229. Specification Pattern

A practical approach:

```text
DoctorSpecifications
    IsPublic()
    IsVerified()
    HasSpecialty()
    SupportsHomeVisit()
    WithinRadius()
    HasMinimumRating()
```

Compose specifications as appropriate.

---

# 230. Search Specification Example

Conceptually:

```csharp
query = query
    .Where(IsPublic)
    .Where(HasSpecialty)
    .Where(IsWithinRadius);
```

The final implementation depends on the team's chosen architecture.

---

# 231. Search Read Model Example

```text
DoctorSearchIndex
-----------------
DoctorId
NameNormalized
NameSearchText
SpecialtyId
SpecialtyNameAr
SpecialtyNameEn
Rating
ReviewCount
StartingPrice
Verified
Active
HomeVisit
GeoPoint
AvailabilityScore
UpdatedAtUtc
```

---

# 232. Why a Search Read Model Helps

It avoids repeatedly joining:

```text
Doctor
Profile
Specialty
Rating
Availability
Location
```

for every public search.

The read model trades some freshness for speed and simpler query execution.

---

# 233. Read Model Limitations

Never use the read model as authoritative for:

```text
booking confirmation
payment
medical records
exact stock transaction
```

Use the source modules.

---

# 234. Search Index Rebuild Strategy

```text
Acquire lock
   |
   v
Create/clear target index
   |
   v
Read source entities in batches
   |
   v
Generate projection
   |
   v
Bulk index
   |
   v
Validate count/checksum
   |
   v
Switch active index
```

This is a future advanced approach.

---

# 235. Batch Size

Indexing should be batched.

Example:

```text
500–2000 records per batch
```

Exact tuning depends on the storage/search provider.

---

# 236. Background Worker

Search index updates should be asynchronous.

```text
Outbox
  |
  v
Background worker
  |
  v
Search index
```

Never block the main user request on a large index rebuild.

---

# 237. Dead-Letter Queue

If index update repeatedly fails:

```text
retry
retry
retry
```

then:

```text
dead-letter
```

with monitoring/alerting.

---

# 238. Search Event Retry

Retries must be idempotent.

Avoid creating duplicate search documents.

---

# 239. Search and Notification Boundaries

Search result ranking must not trigger direct notifications.

Example:

```text
Search detects provider -> no notification
```

Notification occurs only from explicit business events.

---

# 240. Search and Payment Boundaries

Search does not:

```text
charge patient
refund patient
payout provider
```

Search only displays discoverable pricing information.

---

# 241. Search and Booking Boundaries

Search does not reserve a slot.

Booking performs:

```text
availability recheck
price recheck
location recheck
authorization
reservation transaction
```

---

# 242. Search and Patient Privacy

Public search should never expose:

```text
patient name
medical record
medication list
private address
private search history
```

unless a separate authorized workflow explicitly permits it.

---

# 243. Search and Provider Privacy

Do not expose private residential addresses by default.

A provider's public location should be modeled intentionally.

---

# 244. Search and Donor Privacy

Public donor discovery should use:

```text
approximate area
availability state
compatibility state where appropriate
```

and avoid exact coordinates.

---

# 245. Medical Search Safety Guardrails

Search results may help users find a service but should not make unsupported medical claims.

Examples:

```text
"best treatment for X"
```

is not a normal provider search query.

The Search module should not become an uncontrolled medical advice engine.

---

# 246. Search Query Categorization

For specialized pages, category is explicit.

For global search:

```text
q = "CBC"
```

could produce:

```text
Tests
Labs
```

with separate result groups.

---

# 247. Search Result Deduplication

Avoid duplicate results when multiple records represent the same discoverable entity.

Examples:

```text
same pharmacy branch joined with multiple medicines
```

The query should define the correct grouping.

---

# 248. Grouped Search

Example medicine search:

```text
Paracetamol 500mg
-----------------
12 pharmacies nearby
```

not:

```text
Paracetamol + Pharmacy 1
Paracetamol + Pharmacy 2
Paracetamol + Pharmacy 3
```

unless the user asked for branch-level availability.

---

# 249. Search Sorting on Grouped Results

Sort parent groups first, then children.

Example:

```text
Medicine relevance
   -> nearest available branches
```

---

# 250. Search Result Count Semantics

Clearly define whether `total` means:

```text
all matched entities
```

or:

```text
all grouped results
```

Do not mix meanings across endpoints.

---

# 251. Count Performance

`COUNT(*)` across huge filtered datasets can be expensive.

For the MVP, regular counts may be fine.

At scale, consider:

- approximate counts.
- deferred counts.
- cursor pagination without total.

---

# 252. Sorting Null Values

Define how missing values behave.

Example:

```text
No rating -> after rated providers
```

or:

```text
No rating -> neutral score
```

This should be explicit.

---

# 253. Search Result Freshness Indicators

Possible UI:

```text
Available now
```

only if freshness is strong enough.

Otherwise:

```text
Availability may change
```

For urgent healthcare workflows, do not give false certainty.

---

# 254. Search Query Logging

Recommended log fields:

```text
CorrelationId
SearchType
LatencyMs
ResultCount
Sort
FilterCount
Success
```

Do not log raw sensitive values by default.

---

# 255. Search Result Click Tracking

When user clicks a result:

```text
SearchId
ResultId
Position
Timestamp
```

can be recorded for ranking analysis.

Avoid linking click data to sensitive identities unnecessarily.

---

# 256. Position Bias

The first result naturally receives more clicks.

Do not assume:

```text
most clicks = best provider
```

when evaluating ranking.

Future analytics should account for ranking position.

---

# 257. Search A/B Testing — Future

Potential:

```text
Ranking V1 vs Ranking V2
```

Use controlled experiments with guardrails.

Healthcare quality/safety signals should not be sacrificed for CTR.

---

# 258. Search Feature Flags

Potential flags:

```text
EnableArabicSynonyms
EnableFuzzySearch
EnableRankingV2
EnableGlobalSearch
EnableTrendingSearches
```

---

# 259. Search Configuration

Example:

```json
{
  "Search": {
    "DefaultPageSize": 20,
    "MaxPageSize": 50,
    "MaxQueryLength": 100,
    "MinSuggestionCharacters": 2,
    "SuggestionCacheSeconds": 300,
    "DefaultRadiusKm": 10,
    "MaxRadiusKm": 50
  }
}
```

Values are examples.

---

# 260. Search Admin Configuration

Future admin controls:

```text
Synonyms
Search weights
Hidden terms
Provider visibility rules
Category sort order
```

Changes must be audited.

---

# 261. Search Blacklist / Hidden Terms

Use carefully.

Possible uses:

- spam terms.
- malformed internal IDs.
- abuse patterns.

Do not hide legitimate medical search terms simply because they are uncommon.

---

# 262. Search Synonym Administration

Admin UI could allow:

```text
Canonical term:
Orthopedics

Aliases:
عظام
Orthopedic
Ortho
```

Changes should be versioned/audited.

---

# 263. Search Category Administration

Admin may configure:

```text
active specialties
active lab test categories
medicine categories
service types
```

Domain modules remain the source of truth; Search consumes the catalog.

---

# 264. Search Onboarding

New user with no saved location:

```text
Choose a specialty
```

Then:

```text
Where should we search?
Current location / address / map
```

Do not force full profile completion before basic public discovery unless product requires it.

---

# 265. Search Guest Mode

Public provider discovery may optionally be available without login.

However:

```text
Booking
private records
saved addresses
```

still require authentication.

---

# 266. Guest Search Privacy

Guests may provide a location temporarily.

Do not automatically persist it as a user address.

---

# 267. Authenticated Search

Logged-in patients can select saved addresses.

The selected saved address must belong to the current user.

---

# 268. Search by Map

Future feature:

```text
Pan map
   |
   v
Update bounding box
   |
   v
Search visible providers
```

Use throttling/debouncing to prevent excessive calls.

---

# 269. Map Search Bounds

API can accept:

```text
north
south
east
west
```

but must enforce:

```text
maximum viewport size
```

otherwise a user could request an enormous area.

---

# 270. Search Marker Privacy

Map markers should show public business locations only.

Private patient or donor markers should not appear on public maps.

---

# 271. Search + Service Areas

Search can filter providers who actually support the selected location.

```text
Patient location
   |
   v
Provider service area
   |
   +---- supported -> candidate
   |
   +---- unsupported -> exclude / lower priority according to product rule
```

For home visits, service-area filtering should usually be a hard eligibility filter.

---

# 272. Search + Delivery Areas

For pharmacies:

```text
Delivery requested
   |
   v
Check pharmacy delivery area
   |
   +---- yes -> candidate
   |
   +---- no -> exclude from delivery-specific results
```

---

# 273. Search + Home Collection Areas

For labs:

```text
Home collection requested
   |
   v
Lab collection area check
```

Same pattern.

---

# 274. Search + Provider Schedule

Search can use:

```text
next available slot
```

but Booking validates the exact slot at confirmation.

---

# 275. Search + Cancellations

A canceled provider appointment can change availability.

Availability events should eventually refresh the search projection.

---

# 276. Search + Notifications

Optional future recommendation notification:

```text
New pharmacy near your selected area has the medicine you searched for.
```

This should be opt-in and privacy-aware.

---

# 277. Search + Recommendations

Recommendation engine is a future capability.

Search provides:

```text
query
clicks
successful bookings
```

Recommendation consumes aggregated behavioral signals.

Do not make recommendation logic a hidden part of basic search.

---

# 278. Recommendation Privacy

Behavioral recommendations should not leak another user's activity.

Bad:

```text
"People near you searched for cancer treatment"
```

Avoid such content.

---

# 279. Search Personalization

Future signals:

```text
preferred language
previous categories
preferred radius
past successful bookings
```

Personalization should not override critical safety/visibility rules.

---

# 280. Personalization Opt-Out

Users may need:

```text
personalized results on/off
```

Search should still work without personalization.

---

# 281. Search Ranking Guardrails

Never allow personalization to override:

```text
provider suspension
required verification
service area eligibility
authorization
clinical safety constraints
```

---

# 282. Search and Provider Verification

Verification must be represented explicitly.

Do not infer verified status from:

```text
has profile photo
high rating
many bookings
```

Verification comes from its source workflow.

---

# 283. Search and Rating Abuse

A provider with suspicious reviews should follow the Rating/Moderation workflow.

Search consumes the public eligible aggregate.

Do not independently decide that a provider's rating is fraudulent.

---

# 284. Search and Complaints

A provider can have complaints while remaining discoverable.

Do not automatically hide providers because complaint count is high unless explicit policy defines such a rule.

Use formal moderation/suspension decisions.

---

# 285. Search and Provider Status

Suggested states:

```text
Draft
Pending Verification
Active
Temporarily Unavailable
Suspended
Rejected
```

Public search usually includes:

```text
Active
```

and possibly:

```text
Temporarily Unavailable
```

with lower ranking.

---

# 286. Search Contract Example — Doctor

```json
{
  "query": "orthopedic",
  "specialtyId": 5,
  "location": {
    "latitude": 31.0001,
    "longitude": 31.4002
  },
  "radiusKm": 10,
  "filters": {
    "minRating": 4,
    "maxPrice": 800,
    "homeVisit": true
  },
  "sort": "relevance",
  "page": 1,
  "pageSize": 20
}
```

---

# 287. Search Contract Example — Pharmacy Medicine

```json
{
  "query": "paracetamol",
  "location": {
    "latitude": 31.0001,
    "longitude": 31.4002
  },
  "radiusKm": 10,
  "filters": {
    "inStock": true,
    "delivery": true
  },
  "sort": "distance"
}
```

---

# 288. Search Contract Example — Lab Test

```json
{
  "query": "CBC",
  "location": {
    "latitude": 31.0001,
    "longitude": 31.4002
  },
  "filters": {
    "homeCollection": true,
    "maxPrice": 500
  },
  "sort": "relevance"
}
```

---

# 289. Search Request Correlation

Every request should be traceable across:

```text
React
API
Search
Location
Availability
```

Use correlation IDs.

---

# 290. Search Timeout Budget

The complete request should have a bounded latency budget.

Example conceptual allocation:

```text
API total: 800ms
Search query: 300ms
Location: 150ms
Projection: 100ms
```

These are engineering examples, not hard requirements.

---

# 291. Downstream Failure Isolation

If Rating service data is temporarily unavailable, the search should degrade gracefully according to product policy rather than crash all discovery.

Example:

```text
Rating unavailable
-> omit rating sort/filter if required
or
-> use cached rating projection
```

Critical eligibility rules should remain correct.

---

# 292. Search Partial Results

Potential response metadata:

```json
{
  "partial": true,
  "warnings": [
    "Some availability data may be delayed."
  ]
}
```

Use this only when the product UX can explain it appropriately.

---

# 293. Search Provider Versioning

Keep external search-engine implementation behind adapters.

Example:

```text
ISearchProvider
   |
   +---- SqlSearchProvider
   +---- OpenSearchProvider (future)
```

---

# 294. Query Result Caching Rules

Good candidates:

```text
specialty list
popular suggestions
public provider searches
```

Bad candidates:

```text
private patient search history
exact patient address queries
real-time inventory transactions
```

---

# 295. Search Performance Optimization Order

Optimize in this order:

```text
1. Correct indexes
2. Correct query shape
3. Spatial filtering
4. Projection
5. Pagination
6. Caching
7. Denormalized search index
8. External search engine
```

Do not jump directly to Elasticsearch before understanding the query bottleneck.

---

# 296. Search API Documentation

Document:

```text
Request params
Response schema
Filters
Sort values
Pagination
Error codes
Authorization
Rate limits
Examples
```

Use OpenAPI/Swagger.

---

# 297. Swagger Example

Endpoint:

```text
GET /api/v1/search/doctors
```

Parameters:

```text
q
specialtyId
latitude
longitude
radiusKm
minRating
maxPrice
homeVisit
availableOnly
sort
page
pageSize
```

---

# 298. Search Documentation for Frontend

Frontend team should have:

```text
Which fields are stable?
Which fields can be null?
What does distance mean?
What does price mean?
When is availability authoritative?
What errors are retryable?
```

---

# 299. Frontend Search State

Suggested React state:

```text
query
selectedLocation
filters
sort
results
page
loading
error
hasNextPage
suggestions
```

---

# 300. Frontend Query Synchronization

When filters change:

```text
reset page to 1
```

When only pagination changes:

```text
preserve filters
```

When location changes:

```text
reset page and refetch
```

---

# 301. Filter Drawer Workflow

```text
Open filters
    |
    v
Change values locally
    |
    v
Apply
    |
    v
Update URL/state
    |
    v
Reset page
    |
    v
Search
```

This prevents a request for every checkbox click when not desired.

---

# 302. Clear Filters

Provide:

```text
Clear all
```

and optionally individual chips:

```text
Rating 4+
Home visit
Within 10 km
```

---

# 303. Filter Chips

Example:

```text
[Orthopedics ×]
[4+ rating ×]
[Home visit ×]
[10 km ×]
```

Helpful on mobile and desktop.

---

# 304. Search Result Skeleton

Use skeleton loaders instead of blank cards.

This improves perceived performance.

---

# 305. Empty State Recommendations

For zero results:

```text
Try a larger radius
Remove price filter
Remove rating filter
Change specialty
Change location
```

These actions can be generated from the active filters.

---

# 306. Search Suggestions Ranking

Suggestion score can combine:

```text
text prefix match
popularity
category relevance
language match
```

Do not let one extremely popular irrelevant term dominate every context.

---

# 307. Search Dictionary

Potential tables:

```text
SearchTerms
SearchAliases
SpecialtyAliases
MedicineAliases
TestAliases
```

Keep canonical IDs.

---

# 308. SearchTerms Example

```text
Id
CanonicalType
CanonicalId
Language
Text
NormalizedText
PopularityScore
IsActive
```

---

# 309. Search Index Update Rules

When an alias changes:

```text
update alias source
   |
   v
refresh search projection
```

---

# 310. Search Security — Mass Assignment

Do not bind arbitrary query fields to internal database columns.

Create explicit request DTOs.

---

# 311. Search Security — Sort Injection

Reject:

```text
sort=DROP TABLE...
```

Map user-friendly values to known enum values.

Example:

```csharp
DoctorSortOption.Relevance
DoctorSortOption.Distance
DoctorSortOption.Rating
```

---

# 312. Search Security — Field Injection

Never let the client submit:

```text
orderBy=SomeInternalColumn
```

unless it maps to an allowed field list.

---

# 313. Search Security — Response Filtering

Search result projection should explicitly select fields.

Do not serialize full entities.

---

# 314. Search + Audit

Sensitive admin searches should be audited.

Public patient search generally does not require an audit record for every normal query, but analytics and security telemetry may still exist.

---

# 315. Admin Search Audit

Record:

```text
AdminId
SearchType
Time
Reason / ticket if required
```

Do not store excessive sensitive query content.

---

# 316. Search Feature Ownership for 5 Developers

Suggested team ownership:

### Developer 1

```text
Search contracts
Doctor/Nurse discovery
```

### Developer 2

```text
Pharmacy/Medicine search
Laboratory/Test search
```

### Developer 3

```text
Location integration
Nearby filtering
```

### Developer 4

```text
Ranking
Autocomplete
Search analytics
```

### Developer 5

```text
React search UI
Filters
Pagination
E2E/QA
```

These are suggested boundaries; team members can rotate ownership.

---

# 317. Sprint Breakdown

## Sprint 1

```text
Search contracts
Doctor search
Nurse search
Basic filters
Pagination
```

## Sprint 2

```text
Pharmacy search
Medicine search
Laboratory search
Test search
```

## Sprint 3

```text
Location integration
Distance
Service-area filtering
```

## Sprint 4

```text
Ranking
Autocomplete
Arabic/English normalization
```

## Sprint 5

```text
Caching
Analytics
Security hardening
```

## Sprint 6

```text
Performance
E2E
Bug fixing
```

---

# 318. Jira Epic

Suggested:

```text
EPIC: Search & Discovery
```

Stories:

```text
SEARCH-001 Doctor Search
SEARCH-002 Nurse Search
SEARCH-003 Pharmacy Search
SEARCH-004 Medicine Search
SEARCH-005 Laboratory Search
SEARCH-006 Test Search
SEARCH-007 Filters
SEARCH-008 Sorting
SEARCH-009 Nearby Search
SEARCH-010 Autocomplete
SEARCH-011 Arabic Normalization
SEARCH-012 Ranking Engine
SEARCH-013 Search Analytics
SEARCH-014 Search Cache
SEARCH-015 Search Index
SEARCH-016 Security
SEARCH-017 Performance
SEARCH-018 E2E Tests
```

---

# 319. Definition of Ready

A search story is ready when:

```text
Source entity defined
Search fields defined
Filters defined
Sort options defined
Visibility rules defined
Location rules defined
Privacy rules defined
API contract defined
Ranking behavior defined
Empty-state behavior defined
Test scenarios defined
```

---

# 320. Definition of Done

A Search feature is done when:

```text
Backend implemented
Validation implemented
Authorization implemented
Query optimized
DTOs complete
Frontend implemented
Loading state implemented
Empty state implemented
Error state implemented
Analytics implemented where required
Unit tests complete
Integration tests complete
Security tests complete
E2E test complete
Swagger updated
Documentation updated
```

---

# 321. Acceptance Criteria — Doctor Search

```text
Given an active verified doctor
When the patient searches by specialty
Then the doctor appears if matching.

Given a suspended doctor
Then the doctor does not appear in normal public search.

Given a selected patient location
Then returned doctors include valid distance when permitted.

Given an unavailable slot
Then search does not guarantee a booking.
```

---

# 322. Acceptance Criteria — Pharmacy Search

```text
Given a medicine
When a patient searches nearby pharmacies
Then only branches meeting the selected availability rules are returned.

Distance is based on the selected search location.

Branch inventory remains authoritative.
```

---

# 323. Acceptance Criteria — Laboratory Search

```text
Given CBC
When home collection is selected
Then only labs supporting CBC + home collection in the selected area are returned.
```

---

# 324. Acceptance Criteria — Search Privacy

```text
Public search never returns patient private records.

Public donor discovery does not expose exact donor coordinates.

Provider private residential data is not exposed unless explicitly public and authorized.
```

---

# 325. Acceptance Criteria — Booking Handoff

```text
Given a search result
When the patient starts booking
Then booking revalidates:
- provider visibility
- availability
- price
- selected location/service area
```

---

# 326. Critical Business Invariants

```text
1. Search never bypasses authorization.
2. Search never trusts frontend price as final money.
3. Search never trusts stale availability as a reservation guarantee.
4. Search never exposes private patient information.
5. Search uses Location for geographic calculations.
6. Search uses Rating for reputation.
7. Search uses Availability for slots.
8. Search uses Financial for final pricing.
9. Search results are paginated.
10. Search inputs are validated.
11. Search sorting is deterministic.
12. Search index is not the transactional source of truth.
13. Public provider visibility respects verification/suspension rules.
14. Historical transactions are not changed by search updates.
15. Sensitive analytics are minimized and protected.
```

---

# 327. Complete Doctor Discovery Flow

```text
START
  |
  v
Patient opens Doctors
  |
  v
Select specialty/query
  |
  v
Select location
  |
  v
Apply filters
  |
  v
Validate request
  |
  v
Find public eligible doctors
  |
  v
Spatial filter
  |
  v
Text / specialty match
  |
  v
Availability signal
  |
  v
Rating signal
  |
  v
Ranking
  |
  v
Pagination
  |
  v
Search result DTO
  |
  v
Frontend cards
  |
  v
Profile
  |
  v
Booking revalidation
  |
  v
END
```

---

# 328. Complete Pharmacy Discovery Flow

```text
START
  |
  v
Patient searches medicine
  |
  v
Normalize medicine query
  |
  v
Find canonical medicine
  |
  v
Find eligible pharmacy branches with matching inventory
  |
  v
Select location
  |
  v
Spatial filter
  |
  v
Delivery/pickup filter
  |
  v
Rank
  |
  v
Display branch results
  |
  v
Open pharmacy
  |
  v
Create order
  |
  v
Revalidate stock + price
  |
  v
END
```

---

# 329. Complete Laboratory Discovery Flow

```text
START
  |
  v
Patient searches CBC
  |
  v
Normalize test query
  |
  v
Find test
  |
  v
Find labs offering test
  |
  v
Select location
  |
  v
Home collection check
  |
  v
Service area
  |
  v
Distance
  |
  v
Price / availability
  |
  v
Rank
  |
  v
Display
  |
  v
Book
  |
  v
Revalidate
  |
  v
END
```

---

# 330. Global Search Flow

```text
User enters query
       |
       v
Normalize
       |
       v
Classify / search categories
       |
       v
Run category searches
       |
       v
Apply public visibility
       |
       v
Rank inside each category
       |
       v
Return grouped results
```

Global search is optional for MVP.

---

# 331. Recommended MVP Implementation Sequence

Implement in this order:

```text
1. Doctor Search
2. Nurse Search
3. Pharmacy Search
4. Medicine Search
5. Laboratory Search
6. Test Search
7. Location integration
8. Filters
9. Sorting
10. Pagination
11. Autocomplete
12. Arabic aliases
13. Ranking
14. Analytics
15. Caching
16. Advanced indexing
```

This order gives the team usable value early.

---

# 332. Recommended Database Objects

Potential tables/read models:

```text
SearchAliases
SearchTerms
DoctorSearchIndex
NurseSearchIndex
PharmacySearchIndex
MedicineSearchIndex
LaboratorySearchIndex
TestSearchIndex
SearchHistory
SearchAnalyticsEvents
```

Only create what the selected implementation actually needs.

---

# 333. Final Integration Architecture

```text
                         ┌──────────────────────┐
                         │      React UI        │
                         └──────────┬───────────┘
                                    |
                                    v
                         ┌──────────────────────┐
                         │ Search API / BFF     │
                         └──────────┬───────────┘
                                    |
                                    v
                         ┌──────────────────────┐
                         │ Search Application   │
                         │ Service              │
                         └──────────┬───────────┘
                                    |
             ┌──────────────────────┼───────────────────────┐
             |                      |                       |
             v                      v                       v
        Search Read Model      Location Module        Source Modules
             |                      |              ┌──────┼──────┐
             |                      |              |      |      |
             v                      v            Doctor Nurse Pharmacy
         Ranking                    Geo             |      |      |
             |                      |            Lab  Rating Availability
             └──────────────────────┼───────────────────────┘
                                    |
                                    v
                              Search Result
                                    |
                                    v
                                Frontend
```

---

# 334. Final Boundary Rules

The team should remember:

```text
SEARCH
finds, filters, ranks, paginates

LOCATION
coordinates, distance, service areas

RATING
reviews, aggregate rating, moderation state

AVAILABILITY
slots, availability state

FINANCE
pricing, quote, payment, refund

BOOKING
reservation and confirmation

PHARMACY
inventory and medicine stock truth

LABORATORY
test catalog and collection capability truth

AUTHORIZATION
who may access what

NOTIFICATION
how events are delivered
```

---

# 335. Final Search Team Checklist

Before merging a search feature, confirm:

```text
[ ] Query defined
[ ] Entity source defined
[ ] Visibility rules defined
[ ] Filters defined
[ ] Sort options defined
[ ] Ranking defined
[ ] Location integration defined
[ ] Distance semantics defined
[ ] Price semantics defined
[ ] Availability semantics defined
[ ] Privacy reviewed
[ ] Authorization reviewed
[ ] DTO fields minimized
[ ] Pagination implemented
[ ] Empty state implemented
[ ] Error state implemented
[ ] Analytics reviewed
[ ] Rate limiting reviewed
[ ] Indexes/query plan reviewed
[ ] Unit tests
[ ] Integration tests
[ ] Security tests
[ ] E2E tests
[ ] Swagger updated
[ ] Documentation updated
```

---

# 336. Final Architecture Principle

The most important rule is:

```text
Search discovers.

It does NOT own the truth of:

Money      -> Financial
Slots      -> Availability
Distance   -> Location
Ratings    -> Rating
Stock      -> Pharmacy / Inventory
Tests      -> Laboratory
Booking    -> Booking
Access     -> Authorization
```

Search simply combines trusted signals into a useful discovery experience.

---

# 337. Completion Status

**Search & Discovery Workflow: COMPLETE**

Covered:

- Doctor search.
- Nurse search.
- Pharmacy search.
- Medicine search.
- Laboratory search.
- Test search.
- Blood donation discovery boundaries.
- Global search architecture.
- Query normalization.
- Arabic/English aliases.
- Autocomplete.
- Filters.
- Sorting.
- Ranking.
- Nearby search.
- Service-area integration.
- Availability integration.
- Rating integration.
- Pricing boundaries.
- Search result DTOs.
- API design.
- Search projections.
- Search indexes.
- Outbox synchronization.
- Caching.
- Pagination.
- Privacy.
- Authorization.
- IDOR protection.
- Rate limiting.
- Search analytics.
- Security testing.
- Performance testing.
- E2E testing.
- React UX.
- Accessibility.
- Admin boundaries.
- Team ownership.
- Sprint planning.
- Definition of Ready/Done.
- Acceptance criteria.
- End-to-end architecture.

---

# 338. Cross-System Workflow Status

The major TechCare cross-system workflows are now:

```text
✅ Authentication & Authorization
✅ Payment & Financial
✅ Notification
✅ Rating & Complaint
✅ Location & Geo Services
✅ Search & Discovery
```

This completes the main **platform-wide workflow layer** required before moving into a final system-wide integration/architecture document.
