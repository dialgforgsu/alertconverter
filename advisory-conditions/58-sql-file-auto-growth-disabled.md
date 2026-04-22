# SQL File Auto-growth Disabled

## Overview
Detects when auto-growth has been disabled for any data or transaction log file. Although auto-growth should be a contingency rather than the primary growth strategy, disabling it entirely can cause space exhaustion that halts all DML activity.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT DBFileName = EC.ObjectName + '.' + SF.Name, SF.Growth
FROM EventSourceConnection EC
JOIN PerformanceAnalysisSqlFile SF ON EC.ID = SF.EventSourceConnectionID
WHERE EC.IsPerformanceAnalysisEnabled = 1
```

> **Note:** Uses SQL Sentry's internal monitoring database tables.

---

## Redgate Monitor Custom Metric

### Metric Name
Database Files With Auto-growth Disabled

### Description
Returns the count of data and log files across all databases where auto-growth is disabled (`growth = 0`). Files with auto-growth disabled will halt activity when they run out of space with no automatic recovery.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS FilesWithAutogrowthDisabled
FROM sys.master_files
WHERE growth = 0
  AND state_desc = N'ONLINE';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 |

### Notes
- While auto-growth should not be relied upon as the primary sizing strategy, having it completely disabled is risky in production.
- Enable auto-growth with a sensible fixed MB increment: `ALTER DATABASE [db] MODIFY FILE (NAME = N'file', FILEGROWTH = 512MB);`
- After enabling, ensure the max file size is set appropriately to prevent unbounded growth.
