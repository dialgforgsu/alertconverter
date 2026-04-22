# High Runnable Tasks Count

## Overview
Evaluates to true when the average `sys.dm_os_schedulers.runnable_tasks_count` exceeds 10. A high runnable tasks count is one of the most reliable indicators of CPU pressure.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT AVG(runnable_tasks_count)
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count

### Description
Returns the average count of tasks ready to run but waiting for CPU scheduler time across all online schedulers. Persistent values above 1 indicate CPU pressure; values above 10 indicate significant saturation.

### Target Level
Instance

### T-SQL Query
```sql
SELECT AVG(runnable_tasks_count) AS AvgRunnableTasksCount
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
- More reliable than CPU % alone because it shows work genuinely waiting for CPU cycles.
- Combined with high CPU signal waits (condition 28) this strongly confirms CPU saturation.
- Remediation: reduce MAXDOP, tune top CPU-consuming queries, add CPU capacity, or reduce concurrent workload.
