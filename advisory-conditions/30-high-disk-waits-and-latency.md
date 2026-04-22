# High Disk Waits and Latency

## Overview
Detects high disk latency potentially causing high disk wait time in SQL Server. Latency guidelines: < 10 ms (fast), 10â20 ms (acceptable), 20â50 ms (slow), > 50 ms (critical). Transaction log writes should be < 2 ms.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No single T-SQL query â SQL Sentry combines disk latency from Windows PerfMon (`LogicalDisk: Avg. Disk sec/Read|Write`) with SQL Server disk wait stats. A compound condition requiring both to be elevated.

---

## Redgate Monitor Custom Metric

### Metric Name
Max Average Data File Read Latency (ms)

### Description
Returns the maximum average read latency in milliseconds across all SQL Server data files, calculated from cumulative I/O statistics. Values above 20 ms indicate slow I/O; above 50 ms is critical.

### Target Level
Instance

### T-SQL Query
```sql
SELECT MAX(
    CASE WHEN fs.num_of_reads = 0 THEN 0
    ELSE CAST(fs.io_stall_read_ms AS DECIMAL(18,2)) / fs.num_of_reads
    END) AS MaxAvgReadLatencyMs
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS fs
INNER JOIN sys.master_files AS mf
    ON fs.database_id = mf.database_id
    AND fs.[file_id] = mf.[file_id]
WHERE mf.type_desc = N'ROWS';
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
- `sys.dm_io_virtual_file_stats` is cumulative since last restart â for point-in-time latency, snapshot at two time points and calculate the delta.
- Create a companion metric for log write latency: change `type_desc = N'LOG'` and threshold target < 2 ms.
- Redgate Monitor's built-in disk activity analysis shows per-file latency â use this for diagnosis when the alert fires.
