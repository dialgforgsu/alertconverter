# High Redo Completion Time

## Overview
Calculates estimated redo completion time by dividing `log_send_rate` by `redo_rate` from `sys.dm_hadr_database_replica_states`. Alerts when the average estimated catch-up time exceeds 5 minutes. Run on the primary replica only.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT [AG Name, Replica, & Database] = AG.name + N' [' + AR.replica_server_name + N']: (' + DB.database_name + N')',
    [Average Redo Completion Time (Sec)] = COALESCE(RS.log_send_rate / NULLIF(RS.redo_rate, 0), 0)
FROM sys.dm_hadr_database_replica_states AS RS
INNER JOIN sys.availability_databases_cluster AS DB ON RS.group_id = DB.group_id AND RS.group_database_id = DB.group_database_id
INNER JOIN sys.availability_groups AS AG ON AG.group_id = RS.group_id
INNER JOIN sys.availability_replicas AS AR ON RS.group_id = AR.group_id AND RS.replica_id = AR.replica_id
INNER JOIN sys.dm_hadr_availability_group_states AS AGS ON AGS.group_id = AG.group_id
INNER JOIN sys.dm_hadr_availability_replica_states AS ars ON ar.replica_id = ars.replica_id
WHERE ars.role_desc = 'SECONDARY' AND AGS.primary_replica = @@SERVERNAME;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Max AG Redo Completion Time (Seconds)

### Description
Returns the estimated maximum redo completion time in seconds across all AG secondary replicas. A high value means the secondary is receiving logs faster than it can apply them.

### Target Level
Instance

### T-SQL Query
```sql
SELECT ISNULL(
    (SELECT MAX(COALESCE(RS.log_send_rate / NULLIF(RS.redo_rate, 0), 0))
     FROM sys.dm_hadr_database_replica_states AS RS
     INNER JOIN sys.availability_groups AS AG ON AG.group_id = RS.group_id
     INNER JOIN sys.availability_replicas AS AR ON RS.group_id = AR.group_id AND RS.replica_id = AR.replica_id
     INNER JOIN sys.dm_hadr_availability_group_states AS AGS ON AGS.group_id = AG.group_id
     INNER JOIN sys.dm_hadr_availability_replica_states AS ARS ON AR.replica_id = ARS.replica_id
     WHERE ARS.role_desc = N'SECONDARY'
       AND AGS.primary_replica = @@SERVERNAME
       AND RS.log_send_rate > 0 AND RS.redo_rate > 0), 0) AS MaxRedoCompletionTimeSec;
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
| High | > 600 seconds (10 min) |
| Medium | > 300 seconds (5 min) |
| Low | > 120 seconds (2 min) |

### Notes
- High redo completion time means the secondary's redo thread is the bottleneck â typically high I/O latency or slower disk subsystem on the secondary.
- Returns 0 when no secondaries exist or rate data is unavailable (e.g. replica just reconnected).
- Use Redgate Monitor's AG pipeline view to track send/redo queue trends visually.
