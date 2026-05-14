## Data Quality Summary

| Validation | Finding | Action |
|------------|---------|--------|
| Zero-trip placeholders | 58 rows | ❌ Removed |
| Speed outliers (>100 mph) | 22 rows | ❌ Removed |
| Data errors (200+ miles, $0 pay) | 3 rows | ❌ Removed |
| Exact duplicates | 132,712 rows (0.21%) | ❌ Removed |
| Cancellations (negative fare) | 39,779 rows | ✅ Kept (flagged) |
| WAV match flag | B03404/B03406 flood (4,649% rate) | ⚠️ Flagged - do not use |
| Shared match flag | 56.6% match rate | ✅ Validated |
| CBD congestion fee | 5.2M trips outside CBD zone | ⚠️ Flagged as potential issue |

**Final clean rows:** 62,741,602 (99.79% of raw data)

📄 [Full Data Quality Report](DATA_QUALITY_REPORT.md)
