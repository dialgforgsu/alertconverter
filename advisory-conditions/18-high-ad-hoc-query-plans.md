# High Ad Hoc Query Plans

## Overview
Checks for plan cache bloat from single-use plans when "optimize for ad hoc workloads" is disabled and PLE is low. High single-use plan cache pushes useful data pages out of buffer pool.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT SUM(CAST((CASE WHEN usecounts = 1 THEN size_in_bytes ELSE 0 END) AS DECIMAL(19,3)))/1024/1024
FROM sys.dm_exec_cached_plans
```

---

## Redgate Monitor Custom Metric

### Metric Name
Single-Use Plan Cache Size (MB)

### Description
Returns the total size in MB of cached query plans used only once. High values indicate ad hoc plan cache bloat. Enable `optimize for ad hoc workloads` to resolve.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(
    SUM(CASE WHEN usecounts = 1 THEN CAST(size_in_bytes AS DECIMAL(19,3)) ELSE 0 END)
    / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS SingleUsePlanCacheMB
FROM sys.dm_exec_cached_plans WITH (NOLOCK);
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 5 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 500 MB |
| Medium | > 200 MB |
| Low | > 100 MB |

### Notes
- Fix: `EXEC sp_configure 'optimize for ad hoc workloads', 1; RECONFIGURE;` â no restart required.
- Threshold: > 10% of plan cache concerning for servers with <= 64 GB RAM; > 5% for > 64 GB RAM.
