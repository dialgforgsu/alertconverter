# Server MAXDOP Changed

## Overview
Detects when the server-wide max degree of parallelism (MAXDOP) configuration setting changes on a server with more than one processor, by comparing the current value against the last recorded value.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT value_in_use
FROM sys.configurations
WHERE name = N'max degree of parallelism';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Server Max Degree of Parallelism

### Description
Returns the current active MAXDOP value. Use rate-of-change alerting to detect unexpected changes from the approved baseline. MAXDOP changes take effect immediately and can significantly impact query performance and CPU utilisation.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(value_in_use AS INT) AS MaxDegreeOfParallelism
FROM sys.configurations
WHERE name = N'max degree of parallelism';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes â alert on any change |

### Alert Thresholds (Suggested â Rate of Change)
| Severity | Threshold |
|---|---|
| High | Change > 4 |
| Low | Change != 0 |

### Notes
- MAXDOP = 0 means unlimited parallelism (typically undesirable on large servers). Common guidance: set to number of physical cores per NUMA node, capped at 8.
- Change takes effect immediately: `EXEC sp_configure 'max degree of parallelism', <value>; RECONFIGURE;`
- Unexpected changes may indicate an unauthorised modification â Redgate Monitor's built-in configuration change detection provides additional context.
