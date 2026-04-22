# CPU Schedulers Visible Offline Status

## Overview
Evaluates to true when CPU cores are visible to SQL Server but offline. SQL Server licensing may prevent use of all cores. Uneven offline core distribution across NUMA nodes can harm performance.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(*) AS 'Offline Schedulers'
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE OFFLINE'
AND scheduler_id < 255;
```

---

## Redgate Monitor Custom Metric

### Metric Name
CPU Schedulers Visible Offline

### Description
Returns the count of CPU schedulers that are visible to SQL Server but in an offline state. Typically caused by core-based licensing restrictions or CPU affinity mask configuration.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS VisibleOfflineSchedulers
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE OFFLINE'
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
- If cores are offline due to licensing, ensure they are distributed evenly across NUMA nodes.
- Scheduler ID < 255 filters out the dedicated admin connection (DAC) scheduler.
- Review `sys.configurations` for `affinity mask`, `affinity64 mask` to understand the source.
