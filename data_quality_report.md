# NYC For-Hire Vehicle Analytics: Q1 2026

**A data quality-focused analysis of 62.9M NYC taxi trips using Databricks and Power BI**

---

## Project Overview

This project analyzes High Volume For-Hire Vehicle (FHV) trip data from New York City for January-March 2026. The focus is on data quality validation, identifying platform economics, operational efficiency, and accessibility metrics.

**Key Technologies:** Databricks (PySpark), Delta Lake, Power BI, SQL

**Dataset Size:** 62,874,417 raw rows → 62,741,602 cleaned rows (99.79% retention)

---

## Business Questions Answered

| Question | Dashboard Page |
|----------|----------------|
| What is the platform take rate across providers? | Platform Economics |
| Which zones have the longest wait times? | Operations & SLA |
| Where are WAV (wheelchair-accessible) vehicles most needed? | Accessibility |
| How do congestion fees impact driver earnings? | Platform Economics |
| What is the shared ride success rate? | Accessibility |

---

## Data Source

**NYC TLC High Volume FHV Trip Records** (Jan-Mar 2026)

- **Source:** NYC Taxi & Limousine Commission
- **Access:** Databricks Volume (Parquet format)
- **TLC Disclaimer:** *"The TLC publishes base trip record data as submitted by the bases, and we cannot guarantee or confirm their accuracy or completeness."*

---

## Data Quality Validation

### Summary

| Category | Count | % of Total | Decision |
|----------|-------|------------|----------|
| Zero-trip placeholders | 58 | 0.00009% | ❌ Removed |
| Speed outliers (>100 mph for >5 miles) | 22 | 0.00003% | ❌ Removed |
| Data errors (high miles + $0 pay) | 3 | 0.000005% | ❌ Removed |
| Exact duplicates | 132,712 | 0.21% | ❌ Removed |
| Cancellations (negative fare) | 39,779 | 0.06% | ✅ Kept (flagged) |
| Negative driver pay | 20 | 0.00003% | ✅ Kept (flagged) |
| Long-distance trips (>200 miles) | 13 | 0.00002% | ✅ Kept |
| **Valid trips** | **62,741,602** | **99.79%** | ✅ Kept |

### Unique Findings

#### 1. WAV Match Flag Issue
The `wav_match_flag` column was found to be populated as 'Y' for 100% of trips by bases B03404 and B03406 (the only bases in the dataset). This makes match rate calculation impossible.

**Action:** Removed from match rate calculations. Display WAV request volume only (demand signal).

#### 2. CBD Congestion Fee Geographic Discrepancy
NYC CBD congestion fee ($1.50) applies to trips entering Manhattan below 60th St. Data shows 5,265,120 trips with CBD fee recorded outside designated CBD zones.

**Action:** Flagged as potential data issue in documentation.

#### 3. Shared Match Flag Validated
Shared match flag shows 56.6% match rate with no flooding - confirmed trustworthy.

---

## Pipeline Architecture

