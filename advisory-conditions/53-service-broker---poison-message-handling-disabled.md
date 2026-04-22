# Service Broker - 'Poison Message Handling' Disabled

## Overview
Checks `sys.service_queues` to see if `is_poison_message_handling_enabled` is false. When disabled, a message that repeatedly causes the activation procedure to roll back will not be automatically moved out of the queue, potentially causing an infinite loop.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry queries `sys.service_queues` for `is_poison_message_handling_enabled = 0`.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Queues With Poison Message Handling Disabled

### Description
Returns the count of Service Broker queues across all databases where poison message handling is disabled. Without this protection, a single malformed message can permanently stall queue processing.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
UNION ALL SELECT COUNT(*) FROM ' + QUOTENAME(name) + N'.sys.service_queues
WHERE is_poison_message_handling_enabled = 0;'
FROM sys.databases WHERE state_desc = N'ONLINE' AND database_id > 4;

SET @sql = STUFF(@sql, 1, 12, N'');
CREATE TABLE #sq (cnt INT);
INSERT INTO #sq EXEC sys.sp_executesql @sql;
SELECT SUM(cnt) AS QueuesWithPoisonHandlingDisabled FROM #sq;
DROP TABLE #sq;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| Medium | > 0 |

### Notes
- Enable poison message handling: `ALTER QUEUE [queue_name] WITH POISON_MESSAGE_HANDLING (STATUS = ON);`
- With poison message handling enabled, if a message causes 5 consecutive transaction rollbacks, the queue is automatically disabled and a System Event is raised.
- Deliberately disabling this is sometimes done in development â ensure it is enabled in production.
