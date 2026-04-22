# High Pending Disk IO Count

## Overview
Evaluates to true when `sys.dm_os_schedulers.pending_disk_io_count` has an average value greater than zero. A consistently elevated pending I/O count indicates an I/O bottleneck.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT AVG(pending_disk_io_count)
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Average Pending Disk I/O Count

### Description
Returns the average number of pending disk I/O operations across all online CPU schedulers. A value consistently above zero indicates I/O requests are queuing, signalling an I/O bottleneck.

### Target Level
Instance

### T-SQL Query
```sql
SELECT AVG(pending_disk_io_count) AS AvgPendingDiskIOCount
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
| High | > 5 |
| Medium | > 2 |
| Low | > 0 |

### Notes
- A value of 0 is normal. Any consistent non-zero value warrants I/O investigation.
- Pair with the disk latency metric (condition 30) and Redgate Monitor's built-in disk activity view.
- Reflects I/O submitted to the OS by SQL Server's lazy writer or checkpoint that has not yet completed.
