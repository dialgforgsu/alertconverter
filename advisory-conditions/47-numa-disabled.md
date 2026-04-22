# NUMA Disabled

## Overview
Evaluates to true when NUMA is disabled â fewer than 2 distinct parent NUMA nodes are visible to SQL Server. On multi-socket systems, NUMA-aware memory allocation significantly improves performance.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(DISTINCT parent_node_id)
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE parent_node_id <> 32;
```

---

## Redgate Monitor Custom Metric

### Metric Name
NUMA Node Count

### Description
Returns the number of distinct NUMA nodes visible to SQL Server (excluding the DAC node 32). A value of 1 on multi-socket hardware indicates NUMA is disabled or not being used effectively.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(DISTINCT parent_node_id) AS NumaNodeCount
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE parent_node_id <> 32
  AND [status] = N'VISIBLE ONLINE';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Alert based on expected hardware topology â if you have a 2-socket server, alert when count < 2.

| Severity | Threshold |
|---|---|
| Medium | < 2 (on multi-socket hardware) |

### Notes
- NUMA can be disabled in BIOS or via Windows settings. On modern multi-socket servers it should always be enabled.
- Soft-NUMA can be configured in SQL Server to create additional NUMA nodes for I/O performance: `ALTER SERVER CONFIGURATION SET SOFTNUMA ON;`
- Node ID 32 (0x20) is the DAC scheduler and is excluded from the count.
