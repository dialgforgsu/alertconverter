# Large Windows File Cache

## Overview
The Windows file cache typically stays small on a dedicated SQL Server since SQL Server manages its own memory. If another process causes it to grow, it creates memory pressure for SQL Server â particularly relevant where Analysis Services is co-located.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry monitors `Memory: System Cache Resident Bytes` via Windows WMI/PerfMon.

---

## Redgate Monitor Custom Metric

### Metric Name
System Cache Memory (GB)

### Description
Returns the Windows system cache size in GB as reported to SQL Server via `sys.dm_os_sys_memory`. An unexpectedly large value may indicate memory contention between SQL Server and the Windows file cache.

> **Caveat:** Redgate Monitor's WMI collection surfaces `Memory: System Cache Resident Bytes` in the Activity Graph. Use the Activity Graph for precise system cache monitoring; this metric provides a T-SQL approximation.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(system_cache_kb AS DECIMAL(18,2)) / 1024.0 / 1024.0 AS SystemCacheGB
FROM sys.dm_os_sys_memory;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 5 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Establish a baseline on your server. Alert when system cache exceeds 20% of total physical memory or a fixed GB threshold appropriate for your RAM size.

| Severity | Threshold (example for 64 GB server) |
|---|---|
| High | > 12 GB |
| Medium | > 6 GB |

### Notes
- Common causes of large Windows file cache on a SQL Server: SSAS loading cube data, file server workloads, excessive FILESTREAM usage, or third-party backup products streaming large files.
- On dedicated SQL Server machines, system cache should remain small (< 1â2 GB). Larger values warrant investigation.
- Use `EXEC sp_configure 'max server memory', <value>;` to ensure SQL Server does not compete excessively with the file cache.
