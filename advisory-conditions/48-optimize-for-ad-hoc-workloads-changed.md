# Optimize for Ad Hoc Workloads Changed

## Overview
Detects whenever the "optimize for ad hoc workloads" server configuration setting changes by comparing the current value to the previously recorded value. This setting controls whether SQL Server caches full plans or only stubs on first execution of ad hoc queries.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT value_in_use
FROM sys.configurations
WHERE name = N'optimize for ad hoc workloads';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Optimize for Ad Hoc Workloads Enabled

### Description
Returns 1 if "optimize for ad hoc workloads" is currently active, 0 if disabled. Use rate-of-change alerting to detect unexpected changes. This setting should be enabled on most production servers to reduce single-use plan cache bloat.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(value_in_use AS INT) AS OptimizeForAdHocWorkloads
FROM sys.configurations
WHERE name = N'optimize for ad hoc workloads';
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
| Low | != 0 (any change) |

### Notes
- On most production servers this setting should be 1 (enabled) â see condition 18 (High Ad Hoc Query Plans) for the impact when disabled.
- Change takes effect immediately: `EXEC sp_configure 'optimize for ad hoc workloads', 1; RECONFIGURE;`
- An unexpected change to 0 may indicate an unauthorised configuration modification.
