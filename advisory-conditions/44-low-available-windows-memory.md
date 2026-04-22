# Low Available Windows Memory

## Overview
Most Windows servers need at least 100 MB of available memory to function properly and avoid expensive disk paging operations. This condition alerts when available memory drops below this threshold.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry monitors `Memory: Available MBytes` via Windows WMI/PerfMon. Alerts when < 100 MB.

---

## Redgate Monitor Custom Metric

### Metric Name
Available Physical Memory (MB)

### Description
Returns available physical memory in MB as reported by SQL Server's OS memory DMV. Values below 100 MB indicate the system is under severe memory pressure and Windows may begin paging.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(available_physical_memory_kb AS DECIMAL(18,2)) / 1024.0 AS AvailableMemoryMB
FROM sys.dm_os_sys_memory;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | < 100 MB |
| Medium | < 500 MB |
| Low | < 1024 MB |

### Notes
- Alert direction is **below** threshold (low value = problem).
- Low available memory on a SQL Server indicates either SQL Server's max server memory is too high, another process is consuming memory, or the server is undersized.
- Pair with `sys.dm_os_process_memory.process_physical_memory_low` (condition 63) and Page Life Expectancy (condition 45) for a complete memory pressure picture.
