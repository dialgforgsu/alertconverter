# High Avg Wait Time per User Session

## Overview
Divides average wait time per second by active user session count to calculate per-session wait time. If this value exceeds 50 ms, users may be experiencing noticeable delays. SQL Sentry filters benign waits before calculating; the T-SQL equivalent includes all waits.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry uses internally collected wait stats (with benign wait filtering) combined with:
```sql
SELECT count(*)
FROM sys.dm_exec_sessions
WHERE is_user_process = 1
AND last_request_start_time > DATEADD(minute, -1, GETDATE())
```

---

## Redgate Monitor Custom Metric

### Metric Name
Total User Wait Time per Active Session (ms)

### Description
Returns an approximation of average wait time per active user session by dividing total in-flight wait time by current active session count. Values > 50 ms per session suggest noticeable delays.

> **Note:** Redgate Monitor's built-in wait stats provide richer analysis. Use this metric for threshold alerting; use the wait stats dashboard for diagnosis.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @ActiveSessions INT;
DECLARE @TotalWaitMs BIGINT;

SELECT @ActiveSessions = COUNT(*)
FROM sys.dm_exec_sessions
WHERE is_user_process = 1
  AND last_request_start_time > DATEADD(MINUTE, -1, GETDATE());

SELECT @TotalWaitMs = ISNULL(SUM(wait_time), 0)
FROM sys.dm_exec_requests
WHERE is_user_process = 1 AND wait_time > 0;

SELECT CASE WHEN @ActiveSessions > 0
    THEN CAST(@TotalWaitMs AS DECIMAL(18,2)) / @ActiveSessions
    ELSE 0 END AS AvgWaitMsPerSession;
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
| High | > 500 ms |
| Medium | > 100 ms |
| Low | > 50 ms |

### Notes
- This uses currently waiting requests (`sys.dm_exec_requests`) for a point-in-time view.
- Redgate Monitor's wait stats show breakdown by category (CPU, I/O, Lock, Memory) â use for root cause analysis.
- Benign waits (SLEEP, WAITFOR, etc.) are not filtered in this T-SQL version; raw values will be higher than SQL Sentry's filtered equivalent.
