# High CPU for Single Core

## Overview
Sustained CPU > 90% on a single core may indicate a serial workload bottleneck. The threshold is calculated as 100 / number of online cores â if total CPU exceeds this value, at least one core is fully saturated.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT 100.0/COUNT(*)
FROM SYS.DM_OS_SCHEDULERS
WHERE STATUS = 'VISIBLE ONLINE' AND IS_ONLINE = 1;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Single Core CPU Saturation Threshold (%)

### Description
Returns the CPU % equivalent to 100% utilisation on a single logical core (100 / online scheduler count). Use alongside the SQL Server CPU metric (condition 25) to assess whether total CPU % indicates single-core saturation.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(100.0 / COUNT(*) AS DECIMAL(10,2)) AS SingleCoreSaturationThresholdPct
FROM sys.dm_os_schedulers
WHERE [status] = N'VISIBLE ONLINE'
  AND is_online = 1;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
This metric returns a reference value rather than a monitored signal. No direct alert threshold is required. Use it to contextualise the CPU % metric from condition 25.

### Notes
- A 16-core server has a single-core threshold of 6.25% â any total CPU above 6.25% means at least one core is in use.
- Single-core saturation matters most for serial workloads: scalar UDFs, triggers, non-parallel queries, and maintenance operations.
- Use Redgate Monitor's Top Queries sorted by CPU, filtering for DOP = 1 queries, to find serial bottleneck candidates.
