# CPU Schedulers Failed to Create Worker

## Overview
Evaluates to true when there are schedulers that could not create a new worker thread, most likely due to memory constraints. Any non-zero value indicates the SQL Server thread pool is under severe pressure.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(*) AS 'Failed Workers'
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE failed_to_create_worker = 1;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Schedulers Failed to Create Worker

### Description
Returns the count of CPU schedulers that have recorded a worker creation failure. Any non-zero value indicates SQL Server could not allocate a new worker thread â typically due to memory pressure or hitting the `max worker threads` limit.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS SchedulersFailedToCreateWorker
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE failed_to_create_worker = 1;
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
| High | > 2 |
| Medium | > 0 |

### Notes
- `failed_to_create_worker` is cumulative since last SQL Server start. Use rate-of-change mode to detect new failures.
- Investigate: available memory (`sys.dm_os_sys_memory`), `max worker threads` configuration, and non-SQL processes consuming memory.
- Any non-zero value on a healthy server warrants immediate investigation.
