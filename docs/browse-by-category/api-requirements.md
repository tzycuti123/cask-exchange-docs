# BACKEND LOGIC & FLOW SPECS

Browse by Category – Home Page Curated List

## Document Version Control

| Version | Summary of changes | Updated by | Date |
| --- | --- | --- | --- |
| 0.1 | Creation and filled all API docs information | Alice | 08/06/2026 |
| 0.2 | Closed Trending signal decision (volume-only); added `CategoryMetadata` admin entity reference; cross-linked category-metadata-admin.md | Alice | 08/06/2026 |
| 0.3 | Updated display label structure: the card now uses `classificationLabel` from the new hierarchical schema instead of the flat `categoryLabel` (SCR-003). | Alice | 12/06/2026 |

---

## I. Introduction

### 1. Purpose

Provide a complete list of all active cask **classifications** available on the platform, ranked by a tiered sort strategy:

* Prioritizing **Lifetime Trading Volume** for Tier 1
* **Estimated Market Value** used as the primary ranking signal for Tier 2 (cold-start) entities.

This endpoint returns all qualified classifications with no hard limit, sorted by lifetime volume and estimated market value.

*Notation Note: Throughout this document, data field references are written in a conceptual (e.g., transaction.transactionPrice, checkoutSession.originalAmount), singular form to denote the properties of a single logical entity. Developers should interpret these references conceptually.*

### 2. Definitions

| Term | Definition |
|------|------------|
| Classification | A grouping of Master Casks. |
| Master Cask (Parent) | A parent entity grouping multiple variant casks. |
| Variant Cask (Child) | A specific vintage of a Master Cask. Transactions occur at the variant level. |
| Active Ask | An ask where `ask.status = 'active'`. |
| Successful Transaction | A completed trade where its `checkoutSession.status = 'completed'`. |
| Checkout Session Original Amount | Total agreed trade value for the checkout session. Source: `checkoutSession.originalAmount`. |
| Lifetime Volume | `SUM(checkoutSession.originalAmount)` across all completed checkout sessions linked to variant casks of a classification. |
| Transaction Price | The agreed per-unit price of a single cask in a completed transaction. Source: `transaction.transactionPrice`. |
| Median Price | The median of `transaction.transactionPrice` values across all completed transactions for a classification, all time. |
| 30D Median Price Delta | Percentage change in median transaction price: comparing last 30 days vs. prior 30 days. |
| Master Cask Count | Total number of **active** Master Casks linked to the classification, used as a tie-breaker. |
| Traded Classifications | Classifications with `lifetimeVolume > 0`. |
| Cold-start Classifications | Classifications with `lifetimeVolume = 0`. |
| Tie-break | Secondary (and tertiary) sort keys used to ensure a stable, predictable ranking when primary metrics are equal. |
| Trending | A classification is considered Trending when its recent trading activity shows a significant positive momentum. |

---

## II. Functional Requirements

### 1. Overview

Returns all qualified classifications on the platform ranked by **Lifetime Volume** or **Estimated Market Value** using a Tiered Sort Strategy:

- Tier 1: Traded classifications are ranked by their `lifetimeVolume DESC` (**BC-BR-2**)
- Tier 2: Cold-start classifications are ranked below traded classifications by `estMarketValue DESC, NULLs last`
- Any ties are resolved via a **Tie-break (**BC-BR-4**)**

This requires a new dedicated endpoint. The existing `GET /api/cask-metadata/classifications` returns classification labels but does not include market metrics.

#### 1.1. Query Parameters

None. The endpoint returns all qualifying classifications without pagination or limiting, as the total number of classifications on the platform is naturally small (expected < 20).

#### 1.2. Data Description & Metric Implementation

The API calculates and returns the following payload for each classification. Internal metrics used for calculation are also listed.

| # | Metric | Included in response? | Data Type | Calculation Formula / Description | Update Triggers | Notes / Dependencies |
|---|--------|-----------------------|-----------|-----------------------------------|-----------------|----------------------|
| 1 | `classification` | ✅ | `string` | - | Classification created/updated | - |
| 2 | `classificationLabel` | ✅ | `string` | - | Classification label created/updated | Pending **SCR-003** |
| 3 | `classificationImageUrl` | ✅ | `string \| null` | - | Classification image created/updated | Pending **SCR-003** |
| 4 | `masterCaskCount` | ✅ | `number` | Count of Master Casks `masterCask.status = 'active'` AND with at least 1 variant where `caskStatus = 'active'` and `isListed = true` | Master Cask created/status change/deleted ; Variant Cask status change/deleted | - |
| 5 | `lifetimeVolume` | ✅ | `number` | `SUM(checkoutSession.originalAmount)` Default 0 if no transactions | Checkout session completed | Filter: **completed** checkout sessions |
| 6 | `medianPrice` | ✅ | `number \| null` | `MEDIAN(transaction.transactionPrice)` `null` if no completed checkout sessions exist | Checkout session completed | Filter: **completed** checkout sessions. **Unit-weighted per BC-BR-5** |
| 7 | `estMarketValue` | ✅ | `number \| null` | `SUM((referencePriceMin + referencePriceMax) / 2)` across all variant casks. Null if no active casks or `lifetimeVolume > 0` | Variant price/status change, Variant created/deleted, Checkout session completed | **Cold Start fallback only.** `(lifetimeVolume = 0)`. Filter: `caskStatus = 'active'` |
| 8 | `estMedianPrice` | ✅ | `number \| null` | `MEDIAN((referencePriceMin + referencePriceMax) / 2)` across all variant casks. Null if no active casks or `lifetimeVolume > 0` | Variant price/status change, Variant created/deleted, Checkout session completed | **Cold Start fallback only.** `(lifetimeVolume = 0)`. Filter: `caskStatus = 'active'` |
| 9 | `volumeLast30D` | ❌ (internal) | `number` | `SUM(checkoutSession.originalAmount)` | Checkout session completed | Window: `checkoutSession.completedAt >= NOW - 30 days` |
| 10 | `volumePrev30D` | ❌ (internal) | `number` | `SUM(checkoutSession.originalAmount)` | Checkout session completed | Window: `checkoutSession.completedAt >= (NOW - 60 days)` AND `checkoutSession.completedAt < (NOW - 30 days)` |
| 11 | `volumeDelta30D` | ✅ | `number \| null` | `(volumeLast30D - volumePrev30D) / volumePrev30D × 100`. null if `volumePrev30D = 0` | Checkout session completed | - |
| 12 | `medianLast30D` | ❌ (internal) | `number \| null` | `MEDIAN(transaction.transactionPrice)`. null if no transactions in window | Checkout session completed | Window: `checkoutSession.completedAt >= NOW - 30 days` |
| 13 | `medianPrev30D` | ❌ (internal) | `number \| null` | `MEDIAN(transaction.transactionPrice)`. null if no transactions in window | Checkout session completed | Window: `checkoutSession.completedAt >= (NOW - 60 days)` AND `checkoutSession.completedAt < (NOW - 30 days)` |
| 14 | `medianPriceDelta30D` | ✅ | `number \| null` | `(medianLast30D - medianPrev30D) / medianPrev30D × 100`. null if `medianPrev30D = 0` or null, or if `medianLast30D` is null | Checkout session completed | - |
| 15 | `trendingThreshold` | ❌ (internal) | `number` | Configurable percentage threshold for a category to be considered trending. Default `+20` | Admin config update | - |
| 16 | `isTrending` | ✅ | `boolean` | 1. `true` IF: `volumeLast30D` > 0 AND `volumePrev30D` = 0<br>2. `false` IF: `volumeDelta30D` is null<br>3. `true` IF: `volumeDelta30D` >= `trendingThreshold`<br>4. `false` ELSE | 1-day cronjob | Computed using volume momentum only. |
| 17 | `lastRefreshedAt` | ✅ | `datetime` | Timestamp of when this classification's stats were last recomputed. Allows FE to display data freshness if needed. | Aggregation job runs | - |

> [!NOTE]
> **Fields #11 (`volumeDelta30D`) is included in the payload even though FE currently only displays `medianPriceDelta30D` and `isTrending`.** Volume delta is included for future use and for the Trending metric computation. FE may choose to ignore `volumeDelta30D` in the initial UI iteration.

#### 1.3. Formula Reference & Examples

> [!IMPORTANT]
> **Note on Median calculation:** Median is computed at the **unit level** — if a transaction covers 2 units at £10,000 each, both £10,000 values are included in the median dataset. This prevents bulk-buy transactions from being under-represented versus single-unit trades.

> **Example of Unit-Weighted Median:**
> - **Tx 1:** 5 units @ £100
> - **Tx 2:** 1 unit @ £500
> - **Dataset for Median:** `[100, 100, 100, 100, 100, 500]`
> - **Result:** **£100** (Correct unit-level) vs £300 (Incorrect transaction-level)

> [!NOTE]
> **Active Variant Definition:** A variant is active if its parent `MasterCask.status = 'active'` AND its own `caskStatus = 'active'`.

---

## III. Business Rules Summary

| Rule Code | Description | Condition |
|-----------|-------------|-----------|
| **BC-BR-1** | **Exclusion: Empty Classifications** | Classifications with `masterCaskCount = 0` MUST be excluded. Ensures only classifications with active inventory are surfaced. |
| **BC-BR-2** | **Tier 1 (Traded) Primary ranking** | Traded classifications are ranked by their `lifetimeVolume DESC`. Traded classifications are classifications with `lifetimeVolume > 0`. |
| **BC-BR-3** | **Tier 2 (Cold Start)** | Cold-start classifications (Tier 2) are ranked below traded classifications (Tier 1). Tier 2 entries are ranked among themselves by `estMarketValue DESC`, NULL last. Cold-start classifications are classifications with `lifetimeVolume = 0`. |
| **BC-BR-4** | **Tie-breaks** | If primary metrics within a tier are equal (e.g., volume tied in Tier 1, or market value tied in Tier 2), resolve it using these fallback rules:<br>First tie-break: `masterCaskCount DESC`<br>Final tie-break: `classification ASC` (alphabetical). |
| **BC-BR-5** | **Unit-Weighted All-time Median** | Median price calculations are unit-weighted: if a transaction covers 2 units at £10,000 each, both values enter the median dataset. |
| **BC-BR-6** | **Historical Integrity** | All-time metrics (`lifetimeVolume`, `medianPrice`) include ALL completed transactions, including those from inactive master casks. |
| **BC-BR-7** | **No Limit/ No Pagination** | All qualifying classifications MUST be returned in a single response (no limit or pagination). |

---

## IV. Use Cases

### 1. Fetch All Qualifying Classifications

| Item | Description |
|------|-------------|
| **Objective** | Return all qualifying classifications with market signals, ranked by Tier 1 and Tier 2 (**BC-BR-2**, **BC-BR-3**) and Tie-breaks (**BC-BR-4**) |
| **Actor** | System |
| **Trigger** | FE requests Browse by Category data (e.g., on Home page load) |
| **Pre-condition(s)** | Classifications exist in the system with at least one active master cask |
| **Post-condition(s)** | Returns all qualifying classifications (excluding empty ones per **BC-BR-1**) with computed stats |



**Happy Path:**

| # | Scenario | Expected Result | Notes |
|---|----------|----------------|-------|
| 1 | Platform has multiple classifications with `lifetimeVolume > 0` | Return all classifications sorted by `lifetimeVolume DESC`, then `estMarketValue DESC` | Fields resolved per Section 1.2 definitions |

**Negative Path:**

| # | Scenario | Expected Result | Error Code |
|---|----------|----------------|------------|
| 1 | No qualifying classifications exist | Return empty array `[]` | 200 OK |
| 2 | Server error | Return error | 500 Internal Server Error |
| 3 | Unauthenticated request | Reject request | 401 Unauthorized |

**Edge Cases:**

| # | Scenario | Expected Behavior | Side Effects |
|---|----------|-------------------|-|
| 1 | All classifications have `lifetimeVolume = 0` | Return all non-empty classifications sorted by `estMarketValue DESC` (NULLS LAST), then tie-break | FE renders cards with `null` medianPrice (shows `estMedianPrice` fallback or "—") |
| 2 | A classification has `masterCaskCount = 0` | Exclude per **BC-BR-1** | — |
| 3 | Two classifications have equal `lifetimeVolume` AND equal `masterCaskCount` | Apply final tie-break: `classification ASC`. Must not rely on database default. | — |
| 4 | `volumePrev30D = 0` but `volumeLast30D > 0` | `volumeDelta30D = null` (division by zero); `isTrending = true` (new activity from zero base per **BC-BR-6**) | — |
| 5 | `medianPrev30D = null` or `0` | `medianPriceDelta30D = null` | FE hides delta badge |
| 6 | `referencePriceMax = 0` for a variant (data quality issue) | Exclude that variant from `estMarketValue` / `estMedianPrice` calculations (treat as invalid reference data) | Log warning for data team |

---

## V. Technical Dependencies

### 1. Schema Change Request: SCR-003

The classification display labels and images depend on **SCR-003**.

| Item | Detail |
|------|--------|
| **Scope** | Enriched classification schema (classificationLabel, imageUrl). |
| **Status** | 🟡 Pending |
| **Fallback Behavior** | None. Await delivery. |
| **Reference** | See [schema-change-requests.md](../data-models/schema-change-requests.md). |

---

## VI. Data Model

### Entity: Classification — existing (reference)

Managed via `cask-metadata/classifications`. **Pending SCR-003**, this will become an enriched entity containing the classification label and image URL natively.

- `classification`: string (value, e.g., `single_malt`)
- `classificationLabel`: string (display name for UI grouping, e.g., `Single Malt`)
- `imageUrl`: string | null (Representative image)

### Entity: ClassificationMetadata — new (admin-managed)

A new admin-managed entity to store per-classification display configuration (image, label overrides, visibility). Full spec in [classification-metadata-admin.md](./classification-metadata-admin.md).

- `classification`: string (value — primary key, FK to classification enum)
- `classificationImageUrl`: string | null — S3-hosted image URL for this classification card
- `displayName`: string | null — optional admin override for `classificationLabel` display (falls back to `classificationLabel` from classification data if null)
- `isVisible`: boolean — if `false`, classification is suppressed from the Browse by Category section even if it has data
- `createdAt` / `updatedAt`: datetime

### Entity: Master Cask — existing (reference)

- `id`: string (UUID)
- `name`: string
- `classification`: string (FK reference to classification value)
- `classificationLabel`: string
- `status`: string (`active` / `inactive`)
- `peatLevels`: string — **New requested field (SCR-004)**

### Entity: CheckoutSession — existing (reference)

- `id`: string (UUID)
- `originalAmount`: number — total agreed trade value for the session.
- `status`: string (`completed`, `pending`, etc.)
- `completedAt`: datetime | null — Primary boundary for 30D window calculations.

### Entity: Transaction — existing (reference)

- `transactionPrice`: number — agreed per-unit price.
- `quantity`: number — units in the transaction. Used for unit-weighted median (**BC-BR-4**).
- `status`: string (`completed`, `refunded`, etc.)

### Derived / Pre-computed: ClassificationVolumeStats — new (recommended)

| Field | Type | Description |
|-------|------|-------------|
| `classification` | string | Classification value — primary key |
| `classificationLabel` | string | Short display label for the card (e.g., "Single Malt"). Sourced from `Classification.classificationLabel` (**SCR-003**). |
| `classificationImageUrl` | string \| null | Sourced from `Classification.imageUrl` (**SCR-003**). `null` if not configured. |
| `lifetimeVolume` | number | SUM of all completed `cs.originalAmount` |
| `masterCaskCount` | number | Count of active Master Casks in this classification |
| `medianPrice` | number \| null | Unit-weighted median transaction price (all time) |
| `estMarketValue` | number \| null | SUM of mid-points of active variant reference prices |
| `estMedianPrice` | number \| null | Median of mid-points of active variant reference prices |
| `volumeDelta30D` | number \| null | 30-day volume percentage change |
| `medianPriceDelta30D` | number \| null | 30-day median price percentage change |
| `isTrending` | boolean | Whether classification meets the Trending threshold (**volume-only** per formula in 1.3.2) |
| `lastRefreshedAt` | datetime (ISO) | Timestamp of last stats refresh |

---

## VII. Permissions & Access Control

| Use Case / Action | Anonymous | Buyer | Seller | Admin |
|---|---|---|---|---|
| Fetch Browse by Category list | ❌ | ✅ | ✅ | ✅ |

> **Rationale for no anonymous access:** Classification market data (median prices, volumes, deltas) is proprietary market intelligence. Access is restricted to authenticated users only, consistent with the platform's authentication-first policy.

---

## VIII. Performance Considerations

1. **Pre-computation:** Data is served from a `ClassificationVolumeStats` table, refreshed by a scheduled aggregation job every **15–30 minutes**.
2. **Event-Driven Refresh:** An asynchronous refresh of a specific classification's stats MUST be triggered upon:
   - Any checkout session transitioning to `completed` (setting or updating `completedAt`).
   - Any admin update to `masterCask.status`.
   - Any admin update to `ClassificationMetadata` (image, visibility, display name) — see [classification-metadata-admin.md](./classification-metadata-admin.md).
3. **Median Optimization:** Use unit-weighted pre-calculation (**BC-BR-5**) during the stats aggregation job, not at query time.
4. **No Pagination Needed:** Since the number of classifications is small (expected < 20 on the platform), no pagination is required. All classifications are returned in a single response.

---

_End of Document 2_
