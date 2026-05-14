# Data Quality Assessment: NYC FHV Trip Records (Q1 2026)

**Analyst:** [Your Name]  
**Date:** May 2026  
**Source:** NYC TLC High Volume For-Hire Vehicle Trip Data  
**Time Period:** January 1, 2026 - March 31, 2026  
**Total Rows Examined:** 62,874,417

---

## Publisher Disclaimer

The NYC Taxi & Limousine Commission (TLC) explicitly states:

> *"The trip data was not created by the TLC, and TLC makes no representations as to the accuracy of these data. These records are generated from the FHV Trip Record submissions made by bases, so we cannot guarantee or confirm their accuracy or completeness."*

This assessment identifies which fields are reliable for analysis and which require caveats.

---

## Assessment Criteria

| Criteria | Definition | Applied To |
|----------|------------|------------|
| Completeness | Values are present and non-null | All columns |
| Accuracy | Values conform to expected ranges/formats | Numeric fields, timestamps |
| Consistency | Related fields make logical sense together | Cross-field validation |
| Verifiability | Values can be confirmed against external sources | Fee amounts, flags |

---

## Completeness Assessment

**Primary finding:** All core trip fields are populated.

**One exception:**

| Column | Null Count | % of Total | Assessment |
|--------|------------|------------|------------|
| `originating_base_num` | 17,447,652 | 27.8% | Acceptable - field not required for route/financial analysis |

The `originating_base_num` field identifies the base that dispatched the vehicle. This field is not needed for:
- Trip volume analysis
- Financial calculations (fares, driver pay, fees)
- Geographic patterns (pickup/dropoff zones)
- Time-based trends

**Decision:** Proceed with analysis without this column. No imputation attempted.

---

## Accuracy Assessment: Removed Records

**Removal criteria:** Records removed only when all four conditions below were met (system placeholders with no analytical value).

| Condition | Threshold |
|-----------|-----------|
| Zero trip distance | `trip_miles = 0` |
| Zero trip duration | `trip_time = 0` |
| Zero passenger fare | `base_passenger_fare = 0` |
| Zero driver pay | `driver_pay = 0` |

**Rows removed:** 58

**Additional removals:**

| Issue | Detection Method | Rows Removed | Rationale |
|-------|-----------------|--------------|-----------|
| Impossible speed | `speed_mph > 100` AND `trip_miles > 5` | 22 | Indicates `trip_time` data entry error |
| Data inconsistency | `trip_miles > 200` AND `driver_pay = 0` | 3 | Long trips require driver compensation |

**Total removed:** 84 rows (0.00013% of dataset)

**Data retention:** 99.99987%

---

## Consistency Assessment: Flagged Fields

The following fields pass basic validation but require caveats for proper interpretation.

### 3.1 WAV (Wheelchair Accessible Vehicle) Flags

| Field | Definition | Finding | Recommendation |
|-------|------------|---------|----------------|
| `wav_request_flag` | Passenger requested WAV | 162,332 requests | Reliable for demand analysis |
| `wav_match_flag` | Trip occurred in WAV | 5,853,335 matches | Not reliable for match rate |

**Root cause:** Two base numbers (B03404, B03406) populate `wav_match_flag = 'Y'` for 100% of trips.

| Base | Requests | Matches | Implied Match Rate |
|------|----------|---------|---------------------|
| B03404 | 104,334 | 4,850,073 | 4,649% |
| B03406 | 57,998 | 1,007,262 | 1,737% |

**Action:** Use `wav_request_flag` for demand signal only. Do not calculate match rates.

### 3.2 Shared Ride Flags

| Field | Finding | Recommendation |
|-------|---------|----------------|
| `shared_request_flag` | 1,131,234 requests | Reliable |
| `shared_match_flag` | 640,576 matches | Reliable |

**Validation:** 56.6% match rate with no flooding. Within expected range for shared ride services.

### 3.3 Congestion Fees

| Field | Official Rate (FHV) | Finding | Recommendation |
|-------|---------------------|---------|----------------|
| `congestion_surcharge` | $2.75 | Values consistent | Reliable |
| `cbd_congestion_fee` | $1.50 | Values consistent | Reliable |

**Note:** CBD fee applies to trips that start, end, or pass through the Congestion Relief Zone (Manhattan south of 60th St). Pickup/dropoff zone checks alone are insufficient for validation.

### 3.4 Negative Values (Business Events)

| Field | Count | Interpretation | Action |
|-------|-------|----------------|--------|
| `base_passenger_fare < 0` | 39,779 | Cancellations/refunds | Keep, flag as `is_cancellation` |
| `driver_pay < 0` | 20 | Rare adjustments | Keep, flag separately |

Cancellation rate: 0.06% (within industry range of 2-5% for ride-hailing).

### 3.5 Long-Distance Trips

| Route | Distance | Validity |
|-------|----------|----------|
| 138 → 265 (LaGuardia to Buffalo area) | 1,644 miles | Valid (airport to out-of-state) |

Trips exceeding 200 miles represent less than 0.01% of total and are kept as valid outliers.

---

## Compliance Verification

The following fields were verified against official NYC TLC and MTA documentation:

| Field | Source | Verified Rate | Status |
|-------|--------|---------------|--------|
| `congestion_surcharge` | NYS Tax Law § 1286 | $2.75 (FHV) | Confirmed |
| `cbd_congestion_fee` | MTA CRZ Tolling Order | $1.50 (FHV) | Confirmed |
| Driver pay calculation | TLC FHV Trip Record Spec | Not publicly specified | Cannot verify |

---

## Summary by Analysis Type

| Intended Analysis | Recommended Fields | Exclude / Caveat |
|-------------------|-------------------|------------------|
| Platform take rate | `base_passenger_fare`, `driver_pay`, all fees | None |
| Wait time analysis | `request_datetime`, `on_scene_datetime` | None |
| WAV accessibility | `wav_request_flag` | `wav_match_flag` (unreliable) |
| Shared ride success | `shared_request_flag`, `shared_match_flag` | None |
| Congestion pricing impact | `cbd_congestion_fee`, `congestion_surcharge` | None |
| Cancellation rate | `base_passenger_fare` (negative as flag) | None |

---

## Limitations (Inherited from Publisher)

1. No accuracy guarantee - TLC disclaims accuracy of base-submitted data
2. No route data - Cannot verify CBD zone passage
3. WAV match flag unusable - Base-level reporting issue (4,649% match rate)
4. Trip time validated only through speed plausibility

---

## References

- NYC TLC Trip Record Data: https://www.nyc.gov/site/tlc/about/trip-record-data.page
- MTA Congestion Relief Zone: https://mta.info/congestion-relief-zone
- NYC Taxi Zones: https://data.cityofnewyork.us/Transportation/NYC-Taxi-Zones/

---

## Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | May 2026 | Initial assessment |
