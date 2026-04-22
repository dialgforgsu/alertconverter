# Service Broker - Task Limit Reached/Sec

## Overview
Checks for the `SQLServer:Broker Statistics - Task Limit Reached/sec Total` counter >= 1. Task limit reached events indicate Service Broker could not create a new task because the maximum concurrent tasks limit was hit.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry reads `SQLServer:Broker Statistics - Task Limit Reached/sec` via Windows PerfMon.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Task Limit Reached per Second

### Description
Returns the current rate of Service Broker task limit reached events per second. Values >= 1 indicate Service Broker is unable to keep up with its workload and messages are being queued waiting for task slots.

### Target Level
Instance

### T-SQL Query
```sql
SELECT cntr_value AS BrokerTaskLimitReachedPerSec
FROM sys.dm_os_performance_counters
WHERE [object_name] LIKE N'%Broker Statistics%'
  AND counter_name = N'Task Limit Reached/sec';
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
| Medium | > 1 |
| Low | > 0 |

### Notes
- The default Service Broker max concurrent tasks is 20. This can be adjusted but increasing it has memory implications.
- Task limit events indicate the activation procedure is too slow or the message volume is too high for the current concurrency setting.
- Investigate activation procedure performance before increasing the concurrency limit.
