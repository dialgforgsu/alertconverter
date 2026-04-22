# CPU Schedulers Hot Added

## Overview
Evaluates to true when schedulers have been added due to a hot-add CPU event. Hot-added CPUs may not be fully utilised by SQL Server until it is restarted or CPU affinity is reconfigured.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(*) AS 'Hot Added'
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'HOT_ADDED';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Hot Added CPU Schedulers

### Description
Returns the count of CPU schedulers with HOT_ADDED status. Hot-added CPUs may not be efficiently utilised by SQL Server without a restart or affinity reconfiguration.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS HotAddedSchedulers
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'HOT_ADDED';
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
- Supported on SQL Server Enterprise Edition on hot-add capable hardware.
- After a CPU hot-add, a SQL Server restart or affinity mask change may be needed to redistribute work.
- Any detection warrants a review of NUMA topology and CPU affinity configuration.
