# Low Page Life Expectancy

## Overview
Page Life Expectancy (PLE) is the average lifespan in seconds of a data page in buffer pool and is one of the best indicators of memory pressure. The formula used accounts for buffer pool size: PLE < (DataCacheSizeInGB / 4GB Ã 300). Each NUMA node has its own PLE value.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry monitors `SQLServer:Buffer Manager - Page life expectancy` via PerfMon and applies the per-NUMA formula across all nodes.

---

## Redgate Monitor Custom Metric

### Metric Name
Minimum Page Life Expectancy (Seconds)

### Description
Returns the minimum PLE value across all buffer pool NUMA nodes. A low PLE relative to buffer pool size indicates memory pressure â data pages are being evicted from cache too quickly.

### Target Level
Instance

### T-SQL Query
```sql
SELECT MIN(cntr_value) AS MinPageLifeExpectancy
FROM sys.dm_os_performance_counters
WHERE counter_name = N'Page life expectancy'
  AND [object_name] LIKE N'%Buffer Manager%';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
The recommended threshold scales with buffer pool size: PLE < (BufferPoolGB / 4) Ã 300. For common server sizes:

| Buffer Pool Size | Suggested Low PLE Threshold |
|---|---|
| 4 GB | < 300 seconds |
| 16 GB | < 1200 seconds |
| 64 GB | < 4800 seconds |
| 256 GB | < 19200 seconds |

### Notes
- The historic rule of "PLE < 300 = bad" was derived for servers with 4 GB of RAM and does not apply to modern servers.
- On NUMA systems, check each node's PLE individually â one NUMA node under pressure while others are healthy still indicates a problem.
- Redgate Monitor's built-in memory dashboard tracks PLE over time alongside buffer pool size.
