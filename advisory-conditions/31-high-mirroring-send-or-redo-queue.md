# High Mirroring Send or Redo Queue

## Overview
Large database mirroring log send or redo queues indicate a system or network bottleneck. Triggered when either queue exceeds 3 MB. Note: Database Mirroring is deprecated â plan migration to Always On Availability Groups.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry reads `sys.database_mirroring.mirroring_send_queue_size` and `mirroring_redo_queue_size`.

---

## Redgate Monitor Custom Metric

### Metric Name
Max Database Mirroring Queue Size (KB)

### Description
Returns the maximum log send or redo queue size in KB across all Principal-role mirrored databases. Values above 3,072 KB (3 MB) indicate the mirror is falling behind.

### Target Level
Instance

### T-SQL Query
```sql
SELECT ISNULL(
    (SELECT MAX(CASE
        WHEN ISNULL(mirroring_send_queue_size, 0) > ISNULL(mirroring_redo_queue_size, 0)
        THEN mirroring_send_queue_size
        ELSE mirroring_redo_queue_size END)
     FROM sys.database_mirroring
     WHERE mirroring_state IS NOT NULL
       AND mirroring_role = 1), 0) AS MaxMirroringQueueSizeKB;
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
- Database Mirroring was deprecated in SQL Server 2012 and removed in SQL Server 2022+. Applicable to environments on SQL Server 2008â2019 only.
- `mirroring_send_queue_size` is in KB and only populated on the Principal endpoint.
- Growing queues indicate network or I/O bottleneck on the mirror server.
