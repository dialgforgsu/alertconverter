# Tintri High vDisk Throttle Latency

## Overview
Checks for Tintri Virtual Disk throttle latency > 10 ms on any vDisk. vDisk-level throttling means a specific virtual disk is being rate-limited, which will surface as elevated I/O latency on database files stored on that vDisk.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads `Tintri Virtual Disk: Throttle Latency (ms)` via its Tintri integration.

---

## Redgate Monitor Custom Metric

### Metric Name
Max Data File Write Latency (ms) â Tintri vDisk Proxy

### Description
Tintri vDisk-specific counters are not accessible via T-SQL. This metric returns maximum write latency across all database files as a proxy for Tintri vDisk throttling impact.

> **Caveat:** For genuine Tintri vDisk throttle monitoring, use Tintri's management API or native monitoring. This metric surfaces the SQL Server impact.

### Target Level
Instance

### T-SQL Query
```sql
SELECT MAX(
    CASE WHEN fs.num_of_writes = 0 THEN 0
    ELSE CAST(fs.io_stall_write_ms AS DECIMAL(18,2)) / fs.num_of_writes
    END) AS MaxAvgWriteLatencyMs
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS fs
INNER JOIN sys.master_files AS mf
    ON fs.database_id = mf.database_id AND fs.[file_id] = mf.[file_id];
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
| High | > 50 ms |
| Medium | > 20 ms |
| Low | > 10 ms |

### Notes
- Transaction log write latency target: < 2 ms (best), < 5 ms (acceptable), > 10 ms (investigate).
- For write-heavy workloads, separate the log file metric from data file metrics by filtering `mf.type_desc = N'LOG'` for log-specific thresholds.
