# SQL Server Memory Exhaustion

## Overview
Memory Grants Pending is the number of processes waiting for a workspace memory grant. This value should be zero. When above zero, RESOURCE_SEMAPHORE waits will be present, indicating queries are waiting for memory to execute.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry combines `SQLServer:Memory Manager - Memory Grants Pending` (PerfMon) with RESOURCE_SEMAPHORE wait detection from its wait stats collection.

---

## Redgate Monitor Custom Metric

### Metric Name
Memory Grants Pending

### Description
Returns the current count of processes waiting for a query workspace memory grant. This value should always be 0. Any non-zero value indicates severe memory pressure â queries are queuing for execution memory.

### Target Level
Instance

### T-SQL Query
```sql
SELECT cntr_value AS MemoryGrantsPending
FROM sys.dm_os_performance_counters
WHERE [object_name] LIKE N'%Memory Manager%'
  AND counter_name = N'Memory Grants Pending';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 5 |
| Medium | > 0 |

### Notes
- Memory Grants Pending > 0 is always a problem requiring immediate investigation.
- Check: Is `max server memory` set too low? Is another process consuming memory? Does SQL Server need more RAM?
- Pair with RESOURCE_SEMAPHORE in Redgate Monitor's wait stats (Memory category) to confirm duration of waits.
- `SELECT * FROM sys.dm_exec_query_memory_grants WHERE grant_time IS NULL;` shows sessions waiting for grants.
