# High CPU Signal Waits

## Overview
Signal waits occur when a task has its resource wait satisfied but must still wait to be scheduled on a CPU. High signal wait % (> 20%) is often indicative of CPU pressure.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT CAST(100.0 * SUM(signal_wait_time_ms)/ SUM(wait_time_ms) AS NUMERIC(20,2))
FROM sys.dm_os_wait_stats WITH (NOLOCK);
```

---

## Redgate Monitor Custom Metric

### Metric Name
CPU Signal Wait Percentage

### Description
Returns the percentage of total SQL Server wait time that is signal wait time. Values persistently above 20% indicate CPU pressure â work is ready to run but cannot get CPU scheduling time.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(
    100.0 * SUM(signal_wait_time_ms)
    / NULLIF(SUM(wait_time_ms), 0)
AS NUMERIC(10,2)) AS SignalWaitPercent
FROM sys.dm_os_wait_stats WITH (NOLOCK)
WHERE wait_type NOT IN (
    N'SLEEP_TASK', N'WAITFOR', N'BROKER_TO_FLUSH', N'BROKER_TASK_STOP',
    N'CLR_AUTO_EVENT', N'DISPATCHER_QUEUE_SEMAPHORE', N'FT_IFTS_SCHEDULER_IDLE_WAIT',
    N'HADR_WORK_QUEUE', N'LAZYWRITER_SLEEP', N'LOGMGR_QUEUE',
    N'ONDEMAND_TASK_QUEUE', N'REQUEST_FOR_DEADLOCK_SEARCH', N'RESOURCE_QUEUE',
    N'SERVER_IDLE_CHECK', N'SLEEP_DBSTARTUP', N'SLEEP_DCOMSTARTUP',
    N'SLEEP_MASTERDBREADY', N'SLEEP_MASTERMDREADY', N'SLEEP_MASTERUPGRADED',
    N'SLEEP_MSDBSTARTUP', N'SLEEP_SYSTEMTASK', N'SLEEP_TEMPDBSTARTUP',
    N'SNI_HTTP_ACCEPT', N'SP_SERVER_DIAGNOSTICS_SLEEP', N'SQLTRACE_BUFFER_FLUSH',
    N'XE_DISPATCHER_WAIT', N'XE_TIMER_EVENT'
);
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
| High | > 30% |
| Medium | > 20% |
| Low | > 15% |

### Notes
- `sys.dm_os_wait_stats` is cumulative since last SQL Server restart â sudden changes are more meaningful than the absolute value.
- Combine with Redgate Monitor's built-in wait stats (CPU wait category) to confirm whether CPU pressure is current.
- High signal waits + high runnable tasks count (condition 37) together strongly confirm CPU saturation.
