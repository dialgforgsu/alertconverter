# Service Broker - Activation Error Occurred

## Overview
Checks for Service Broker activation stored procedures that have exited due to errors. Activation errors mean queued messages are not being processed and the queue may grow unboundedly.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL exposed â SQL Sentry monitors Service Broker activation errors via its internal event capture.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Activation Errors (All Databases)

### Description
Returns the total count of Service Broker queues across all databases where activation has been aborted due to errors (`is_activation_enabled = 0` AND an activation procedure exists, combined with queue error state). A non-zero count means messages may be accumulating unprocessed.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
UNION ALL SELECT COUNT(*) FROM ' + QUOTENAME(name) + N'.sys.service_queues
WHERE activation_procedure IS NOT NULL
  AND is_activation_enabled = 0
  AND is_receive_enabled = 1;'
FROM sys.databases
WHERE state_desc = N'ONLINE' AND database_id > 4;

SET @sql = STUFF(@sql, 1, 12, N'');
CREATE TABLE #sb (cnt INT);
INSERT INTO #sb EXEC sys.sp_executesql @sql;
SELECT SUM(cnt) AS ActivationDisabledWithProcedure FROM #sb;
DROP TABLE #sb;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 5 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 |

### Notes
- Service Broker activation is automatically disabled when the activation procedure raises an unhandled error 5 times in a row.
- Re-enable with: `ALTER QUEUE [queue_name] WITH ACTIVATION (STATUS = ON);`
- Investigate and fix the activation procedure error before re-enabling to prevent repeated failure.
- Check `sys.transmission_queue` for messages waiting to be delivered.
