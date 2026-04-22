# Check Data File Size

## Overview
Utility/helper condition that verifies a data file is at least 5,120 MB. Used as a building block inside composite conditions (e.g. Data File Growth) to avoid alerting on very small files.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL exposed. SQL Sentry evaluates file size from its internal `PerformanceAnalysisSqlFile` table and uses this as a gate condition within composite rules.

---

## Redgate Monitor Custom Metric

### Metric Name
Large Data Files Count (>= 5 GB)

### Description
Returns the count of data files across all databases that are 5,120 MB or larger. Use alongside the Data File Growth metric (condition 13) to understand which files are material in size.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS LargeDataFileCount
FROM sys.master_files
WHERE type_desc = N'ROWS'
  AND size * 8.0 / 1024 >= 5120;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Primarily informational. Use in rate-of-change mode to detect when new files cross the 5 GB threshold.

### Notes
- `sys.master_files.size` is in 8 KB pages. Multiply by 8 for KB, divide by 1024 for MB.
- This metric pairs with condition 13 (Data File Growth) to filter alerts to substantively sized files.
