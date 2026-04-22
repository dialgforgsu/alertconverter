# Log File Growth

## Overview
Evaluates to true if any transaction log file >= 1,024 MB has grown since the last evaluation. Frequent log growth events create excessive VLFs and indicate the log is not pre-sized appropriately.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
Evaluated from SQL Sentry's internal `PerformanceAnalysisSqlFile` size history â detects an increase in log file size between collections for files meeting the 1,024 MB minimum size threshold.

---

## Redgate Monitor Custom Metric

### Metric Name
Total Transaction Log File Size (GB)

### Description
Returns the total allocated size of all transaction log files across all databases in GB. Enable rate-of-change alerting to detect growth events.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(SUM(CAST(size AS BIGINT)) * 8.0 / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS TotalLogFileSizeGB
FROM sys.master_files
WHERE type_desc = N'LOG';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 15 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes |

### Alert Thresholds (Suggested â GB growth per collection)
| Severity | Threshold |
|---|---|
| High | > 5 GB |
| Medium | > 1 GB |
| Low | > 0.25 GB |

### Notes
- Frequent log growth creates excessive VLFs â see condition 38 (High VLF Count).
- Pre-size logs appropriately and configure auto-growth as a safety net only (use MB increments, not %).
- Redgate Monitor's built-in disk usage projections help anticipate log growth trends.
