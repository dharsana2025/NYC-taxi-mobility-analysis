# Data Quality Report: NYC FHV Trip Data (Q1 2026)

**Report Date:** May 2026
**Data Source:** NYC TLC High Volume For-Hire Vehicle Trip Records
**Time Period:** January 1, 2026 - March 31, 2026
**Analyst:** [Your Name]

---

## Executive Summary

This report documents all data quality validations performed on 62.9 million NYC FHV trip records. After systematic validation, **99.79% of records (62.7M rows)** are confirmed as valid for analysis. Key findings include identification of WAV match flag reporting issues and CBD congestion fee geographic discrepancies.

| Metric | Value |
|--------|-------|
| Raw rows | 62,874,417 |
| Rows removed | 132,815 (0.21%) |
| Final clean rows | 62,741,602 |
| Data quality confidence | High |

---

## 1. Data Source Information

### Source
NYC Taxi & Limousine Commission (TLC)
- **Dataset:** High Volume For-Hire Vehicle (HVFHV) Trip Records
- **Format:** Parquet (Snappy compressed)
- **Storage:** Databricks Volume

### TLC Disclaimer
> *"The TLC publishes base trip record data as submitted by the bases, and we cannot guarantee or confirm their accuracy or completeness."*

**Implication:** Data quality varies by base. This report identifies specific base-level issues.

---

## 2. Validation Methodology

### Tools Used
- **Databricks** (PySpark, Spark SQL)
- **Delta Lake** (versioning, ACID transactions)

### Validation Categories

| Category | Validation Rule | Threshold |
|----------|-----------------|-----------|
| Completeness | Null/zero checks | Remove placeholders |
| Accuracy | Speed calculation | Remove >100 mph for >5 miles |
| Uniqueness | Duplicate detection | Remove exact duplicates |
| Consistency | Cross-field logic | Flag anomalies |
| Reasonableness | Domain knowledge | Flag outliers |

---

## 3. Issues Identified & Resolved

### 3.1 Zero-Trip Placeholders (Removed)

| Count | % of Total | Decision |
|-------|------------|----------|
| 58 | 0.00009% | ❌ Remove |

**Criteria:** All key fields = 0
- `trip_miles = 0`
- `trip_time = 0`  
- `base_passenger_fare = 0`
- `driver_pay = 0`

**Rationale:** System placeholders with no analytical value.

---

### 3.2 Speed Outliers (Removed)

| Count | % of Total | Decision |
|-------|------------|----------|
| 22 | 0.00003% | ❌ Remove |

**Criteria:** 
- Speed > 100 mph
- Distance > 5 miles

**Example:** Trip from LGA (138) to Buffalo (265): 1,644 miles in 2,195 seconds = 2,697 mph (impossible)

**Rationale:** Data entry errors where `trip_time` is incorrectly recorded.

---

### 3.3 Data Errors: High Miles + $0 Pay (Removed)

| Count | % of Total | Decision |
|-------|------------|----------|
| 3 | 0.000005% | ❌ Remove |

**Criteria:** 
- `trip_miles > 200`
- `driver_pay = 0`

**Rationale:** Impossible for a driver to complete a 200+ mile trip with zero compensation.

---

### 3.4 Exact Duplicates (Removed)

| Count | % of Total | Decision |
|-------|------------|----------|
| 132,712 | 0.21% | ❌ Remove |

**Criteria:** Same values for all key fields:
- `PULocationID`, `DOLocationID`
- `request_datetime`, `pickup_datetime`, `dropoff_datetime`
- `trip_miles`, `base_passenger_fare`, `driver_pay`

**Example:** Route 4→162 at 07:06:59 appears multiple times across different dates with identical values.

**Rationale:** Systematic duplication error from base reporting system.

---

## 4. Issues Identified & Documented (Not Removed)

### 4.1 WAV Match Flag Unreliable (Flagged)

| Base | WAV Requests | WAV Matches | Match Rate | Issue |
|------|--------------|-------------|------------|-------|
| B03404 | 104,334 | 4,850,073 | 4,649% | Flooded (every trip marked as match) |
| B03406 | 57,998 | 1,007,262 | 1,737% | Flooded (every trip marked as match) |

**Finding:** Both bases in the dataset populate `wav_match_flag = 'Y'` for 100% of trips, regardless of WAV request status.

**Action:**
- ❌ Do NOT use `wav_match_flag` for match rate calculations
- ✅ Use `wav_request_flag` only (demand signal)
- ✅ Document in dashboard: "Match rate unavailable due to base reporting issue"

---

### 4.2 CBD Congestion Fee Geographic Discrepancy (Flagged)

| Metric | Value | Status |
|--------|-------|--------|
| Trips with CBD fee | 19,887,432 | ✅ Plausible |
| Avg CBD fee | $1.50 | ✅ Correct (TLC rate for FHV) |
| Pickup zones affected | 262 of 263 | ✅ Almost all zones |
| Non-CBD zones with fee | 5,265,120 | ⚠️ Issue |

**Finding:** 5.2M trips recorded with CBD congestion fee outside Manhattan CBD zone (below 60th St).

**Action:**
- ⚠️ Flag as potential data issue in documentation
- ✅ Use `cbd_congestion_fee` as reported (cannot verify geographic accuracy)

---

### 4.3 Shared Match Flag (Validated - Keep)

| Shared Requests | Shared Matches | Match Rate | Status |
|-----------------|----------------|------------|--------|
| 1,131,234 | 640,576 | 56.6% | ✅ Good |

**Finding:** No flooding or anomalies detected. Match rate is reasonable.

**Action:** ✅ Safe to use for analysis.

---

### 4.4 Valid Outliers (Keep)

| Category | Count | Rationale |
|----------|-------|-----------|
| Cancellations (negative fare) | 39,779 | Legitimate business events (0.06% rate) |
| Negative driver pay | 20 | Rare edge cases (<0.0001%) |
| Long-distance trips (>200 miles) | ~10 | Airport-to-out-of-state travel (e.g., LGA→Buffalo) |

**Action:** ✅ Keep, add quality flags for filtering if needed.

---

## 5. Data Quality Metrics Summary

| Metric | Value |
|--------|-------|
| **Completeness** | |
| Records with null keys | 0 (validated) |
| Records with zero values removed | 58 |
| **Accuracy** | |
| Speed outliers removed | 22 |
| Data errors removed | 3 |
| **Uniqueness** | |
| Exact duplicates removed | 132,712 |
| Duplicate rate | 0.21% |
| **Consistency** | |
| WAV match flag reliable? | No (flagged) |
| Shared match flag reliable? | Yes |
| CBD fee geographically accurate? | Partial (flagged) |
| **Validity** | |
| Cancellations kept | 39,779 |
| Negative driver pay kept | 20 |
| Long trips kept | ~10 |

---

## 6. Final Clean Dataset Specification

| Property | Value |
|----------|-------|
| **Table name** | `agg_metrics.cleaned_fhv_data` |
| **Format** | Delta Lake |
| **Total rows** | 62,741,602 |
| **Total columns** | 25 |
| **Storage size** | ~1.4 GB (Parquet + Delta log) |
| **Quality flags added** | `is_cancellation`, `is_negative_driver_pay`, `is_long_trip` |

### Quality Flag Definitions

| Flag | Definition | Use Case |
|------|------------|----------|
| `is_cancellation` | `base_passenger_fare < 0` | Filter or analyze separately |
| `is_negative_driver_pay` | `driver_pay < 0` | Rare edge case flag |
| `is_long_trip` | `trip_miles > 200` | Airport/out-of-state trips |

---

## 7. Recommendations for Analysis

### Do Use
- ✅ `shared_request_flag`, `shared_match_flag` (validated)
- ✅ `wav_request_flag` (demand signal only)
- ✅ All financial columns (after flagging cancellations)
- ✅ Trip distance and time (after removing outliers)

### Do Not Use
- ❌ `wav_match_flag` for match rate calculations (data unreliable)

### Use with Caution
- ⚠️ `cbd_congestion_fee` - geographic accuracy not verified
- ⚠️ Any aggregation by base B03404 or B03406 for WAV metrics

---

## 8. Validation Scripts

All validation code is available in the `notebooks/` directory:

| Script | Purpose |
|--------|---------|
| `01_data_cleaning.py` | Remove placeholders, outliers, duplicates |
| `02_aggregation.py` | Create gold layer for Power BI |
| `03_validation.py` | Quality checks and documentation |

---

## 9. References

- [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/trip-record-data.page)
- [TLC Data Dictionary](https://www.nyc.gov/assets/tlc/downloads/pdf/trip_record_data_dictionary.pdf)
- [CBD Congestion Pricing](https://nyc.gov/cbd)

---

## 10. Change Log

| Date | Version | Changes |
|------|---------|---------|
| May 2026 | 1.0 | Initial data quality report |

---

**Report prepared by:** [Your Name]
**Contact:** [Your Email / LinkedIn]
