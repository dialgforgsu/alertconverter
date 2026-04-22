# High Database Replica Send or Recovery Queue

## Overview
Large AG log send or recovery queues indicate a system or network bottleneck. Triggered when either queue exceeds 3 MB. Use the Always On pipeline view in Performance Analysis to troubleshoot.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry monitors `sys.dm_hadr_database_replica_states.log_send_queue_size` and `redo_queue_size` via its native AG integration.

---

## Redgate Monitor Custom Metric

### Metric Name
Max AG Replica Queue Size (KB)

### Description
Returns the maximum of log send queue or redo queue size in KB across all AG secondary replicas. Values above 3,072 KB (3 MB) indicate the secondary is falling behind.

### Target Level
Instance

### T-SQL Query
```sql
SELECT ISNULL(
    (SELECT MAX(CASE
        WHEN drs.log_send_queue_size > drs.redo_queue_size
        THEN drs.log_send_queue_size
        ELSE drs.redo_queue_size END)
     FROM sys.dm_hadr_database_replica_states AS drs
     INNER JOIN sys.dm_hadr_availability_replica_states AS ars
         ON drs.replica_id = ars.replica_id
     WHERE ars.role_desc = N'SECONDARY'
       AND drs.is_local = 0), 0) AS MaxReplicaQueueSizeKB;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 10240 KB (10 MB) |
| Medium | > 3072 KB (3 MB) |
| Low | > 1024 KB (1 MB) |

### Notes
- Growing send queue = network bandwidth insufficient for log generation rate.
- Growing redo queue = secondary CPU or I/O cannot apply logs fast enough.
- Redgate Monitor's built-in AG pipeline view tracks queue sizes visually â use this metric for automated alerting.
