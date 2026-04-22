# Cost Threshold of Parallelism Changed

## Overview
Queries `sys.configurations` to detect whenever the server-wide Cost Threshold of Parallelism (CTP) setting changes, by comparing the current value against the last collected value.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT value_in_use
FROM sys.configurations
WHERE name = N'cost threshold for parallelism'
```

---

## Redgate Monitor Custom Metric

### Metric Name
Cost Threshold for Parallelism

### Description
Returns the current active Cost Threshold for Parallelism value. The default of 5 is widely considered too low for modern workloads (25â75 is typical). Use rate-of-change alerting to detect unauthorised configuration drift.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(value_in_use AS INT) AS CostThresholdForParallelism
FROM sys.configurations
WHERE name = N'cost threshold for parallelism';
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
| High | Change > 20 |
| Low | Change != 0 |

### Notes
- Changes take effect immediately (no restart) via `EXEC sp_configure 'cost threshold for parallelism', <value>; RECONFIGURE;`
- Unexpected changes may indicate an unauthorised modification â pair with Redgate Monitor's built-in configuration change detection.
- Most environments benefit from a CTP between 25 and 75; test thoroughly before changing in production.
