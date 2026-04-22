# CPU Unused Schedulers

## Overview
Evaluates to true when CPU schedulers are fully disabled (`is_online = 0`). SQL Server cannot use disabled cores for any processing. Root causes include affinity masking or core-based licensing limits.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(*) AS 'Unused Schedulers'
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [is_online] = 0
AND scheduler_id < 255;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Disabled (Unused) CPU Schedulers

### Description
Returns the count of CPU schedulers completely disabled by SQL Server (`is_online = 0`). These cores are unavailable for query processing.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS UnusedSchedulers
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE is_online = 0
  AND scheduler_id < 255;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| Medium | > 0 |

### Notes
- Unlike VISIBLE OFFLINE, `is_online = 0` means completely disabled â not just paused.
- Review affinity mask settings in `sys.configurations` alongside CPU layout in `sys.dm_os_schedulers`.
