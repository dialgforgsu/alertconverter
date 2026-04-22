# High Active User Sessions

## Overview
Triggered if user sessions with recent activity exceed a threshold. High active sessions can precede an overload state or indicate denial-of-service activity. Threshold must be calibrated per server.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT count(*)
FROM sys.dm_exec_sessions
WHERE is_user_process = 1
AND last_request_start_time > DATEADD(minute, -1, GETDATE())
```

---

## Redgate Monitor Custom Metric

### Metric Name
Active User Sessions (Last 1 Minute)

### Description
Returns the count of user sessions (excluding system processes) that have started a request within the last minute. Baseline over several business days before setting alert thresholds.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS ActiveUserSessions
FROM sys.dm_exec_sessions
WHERE is_user_process = 1
  AND last_request_start_time > DATEADD(MINUTE, -1, GETDATE());
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested â adjust per server)
| Severity | Threshold |
|---|---|
| High | > 500 |
| Medium | > 200 |
| Low | > 100 |

### Notes
- Thresholds vary enormously between environments â baseline before setting values.
- Pair with Redgate Monitor's built-in blocking chain view when high session counts correlate with performance problems.
