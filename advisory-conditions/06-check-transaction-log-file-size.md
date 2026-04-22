# Check Transaction Log File Size

## Overview
Utility/helper condition that verifies a transaction log file is at least 1,024 MB. Used as a prerequisite filter inside composite conditions (e.g. Log File Growth) to avoid alerting on small log files where growth is routine.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL exposed. SQL Sentry evaluates log file size from its internal `PerformanceAnalysisSqlFile` table.

---

## Redgate Monitor Custom Metric

### Metric Name
Large Transaction Log Files Count (>= 1 GB)

### Description
Returns the count of transaction log files across all databases that are 1,024 MB or larger. Use alongside the Log File Growth metric (condition 42).

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS LargeLogFileCount
FROM sys.master_files
WHERE type_desc = N'LOG'
  AND size * 8.0 / 1024 >= 1024;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Primarily informational. Use in rate-of-change mode to detect new log files crossing the 1 GB threshold.

### Notes
- Large log files paired with frequent growth events indicate the log is not pre-sized appropriately.
- See condition 38 (High VLF Count) for the impact of many small auto-growth events on VLF count.
