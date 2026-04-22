# Data File Growth

## Overview
Evaluates to true if any data file greater than 5 GB has grown since the last evaluation. Frequent auto-growth events cause file fragmentation and degrade performance.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
Evaluated from SQL Sentry's internal `PerformanceAnalysisSqlFile` size history â no standalone T-SQL exposed. Compares current size against the previously stored value for files >= 5,120 MB.

---

## Redgate Monitor Custom Metric

### Metric Name
Total Data File Size (GB)

### Description
Returns the total allocated size of all data files across all databases in GB. Enable rate-of-change alerting to detect growth events. Adjust thresholds to suit your expected growth patterns.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(SUM(CAST(size AS BIGINT)) * 8.0 / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS TotalDataFileSizeGB
FROM sys.master_files
WHERE type_desc = N'ROWS';
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
| High | > 10 GB |
| Medium | > 2 GB |
| Low | > 0.5 GB |

### Notes
- Pre-size data files proactively â use Redgate Monitor's built-in disk usage projections to anticipate growth.
- Auto-growth events cause file fragmentation; avoid relying on auto-growth as the primary sizing strategy.
