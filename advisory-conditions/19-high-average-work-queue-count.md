# High Average Work Queue Count

## Overview
Evaluates to true when `sys.dm_os_schedulers.work_queue_count` has an average value greater than one. A high work queue count indicates that worker threads are backing up and max worker threads may need to be increased.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT AVG(work_queue_count)
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Average Scheduler Work Queue Count

### Description
Returns the average number of tasks queued waiting for a CPU scheduler across all online schedulers. A sustained value > 1 indicates worker threads are backing up.

### Target Level
Instance

### T-SQL Query
```sql
SELECT AVG(work_queue_count) AS AvgWorkQueueCount
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
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
| High | > 10 |
| Medium | > 3 |
| Low | > 1 |

### Notes
- Before increasing `max worker threads`, investigate whether CPU saturation is the root cause.
- Persistent values > 1 indicate CPU or worker thread saturation requiring investigation.
