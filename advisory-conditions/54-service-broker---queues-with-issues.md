# Service Broker - Queues with Issues

## Overview
Checks `sys.service_queues` for entries that have an activation procedure but where `is_activation_enabled`, `is_receive_enabled`, or `is_enqueue_enabled` is disabled. Any such queue has a configuration inconsistency that prevents normal operation.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry queries `sys.service_queues` for queues with an activation procedure where one or more of the enabled flags is false.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Queues With Configuration Issues

### Description
Returns the count of Service Broker queues across all databases that have an activation procedure defined but have one or more critical flags disabled (activation, receive, or enqueue). These queues are partially functional and may silently fail to process messages.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
UNION ALL SELECT COUNT(*) FROM ' + QUOTENAME(name) + N'.sys.service_queues
WHERE activation_procedure IS NOT NULL
  AND (is_activation_enabled = 0 OR is_receive_enabled = 0 OR is_enqueue_enabled = 0);'
FROM sys.databases WHERE state_desc = N'ONLINE' AND database_id > 4;

SET @sql = STUFF(@sql, 1, 12, N'');
CREATE TABLE #sq2 (cnt INT);
INSERT INTO #sq2 EXEC sys.sp_executesql @sql;
SELECT SUM(cnt) AS QueuesWithIssues FROM #sq2;
DROP TABLE #sq2;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 10 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 |

### Notes
- `is_receive_enabled = 0` prevents the activation procedure from dequeuing messages.
- `is_enqueue_enabled = 0` prevents new messages from being added to the queue.
- `is_activation_enabled = 0` prevents the activation procedure from being invoked automatically.
- All three should typically be 1 for a healthy production Service Broker queue.
