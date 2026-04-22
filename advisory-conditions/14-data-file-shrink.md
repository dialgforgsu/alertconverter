# Data File Shrink

## Overview
Evaluates to true if any data file is smaller than it was during the last evaluation. Shrinking data files creates fragmentation and forces future re-growth, incurring double the I/O cost.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
Evaluated from SQL Sentry's internal `PerformanceAnalysisSqlFile` size history â detects a decrease in file size between collections.

---

## Redgate Monitor Custom Metric

### Metric Name
Total Data File Size (GB) â Shrink Detection

### Description
Returns total data file size in GB. Use with rate-of-change alerting and a **below threshold** (negative delta) alert direction to detect shrink events.

### Target Level
Instance

### T-SQL Query
```sql
-- Same query as Data File Growth (condition 13).
-- Create a second alert on this metric with direction set to "below zero" (negative rate of change).
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
| Alert direction | Below threshold (negative delta) |

### Alert Thresholds (Suggested â rate of change)
| Severity | Threshold |
|---|---|
| High | < -1 GB |
| Low | < 0 (any decrease) |

### Notes
- Reuse the same metric as condition 13 â add a second alert with the direction reversed.
- Data file shrinking causes index fragmentation and forces SQL Server to re-grow files later.
- Disable auto-shrink: `ALTER DATABASE [dbname] SET AUTO_SHRINK OFF;`
